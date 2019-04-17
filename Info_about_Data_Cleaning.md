<!-- vim: set textwidth=80 : -->

# Normalization and cleaning of hMeDIP-seq and RNA-seq data

## Introduction to what I am doing here

To start with, I have my raw count files. I have combined them already (using bash scripting), so it is in table format where it is tab delimited like this:

    Gene_ID | Sample 1 | Sample 2 | Sample 3 | Sample 4
    ENSRN0001 | 6 | 8 | 24| 29
    ENSRN0002 | 12 | 13 | 14 | 13
    ENSRN0003 | 18 | 17 | 1 | 0

For the RNA I also have results from edgeR and DESeq2. I have the transcript info from the GTF file in a csv file that I will use to determine transcript length.

For the hMeDIP samples I also have the edgeR statistical analysis, the bed file stating the location of the each peak (chromosome, start, stop, name of peak), the count data for genomic input (which was used for the peak calling and will be used for determining the cutoff of expression) and the region_analysis output which tells which gene the peak is close to. 

RNA-seq samples were aligned using STAR and then quantified using htseq-count. Both edgeR and DESeq2 are used for differential analysis and genes below a false discovery rate (FDR, also called adjusted P-value) of 0.05 by both DESeq2 and edgeR will be called differential. A limit of detection will be calculated using the genomic input from the hMeDIP-seq portion of the experiment. 

hMeDIP-seq peaks were called using the irreproducible discovery rate pipeline designed for the ChIP-seq portion of the ENCODE consortium. Counts were generated using htseq-count. Differential analysis was calculated using edgeR. To account for not having a second statistical program (DESeq2 can't normalize to all data), a cutoff of 1.5-fold change will be used in conjunction with an FDR of 0.05 by edgeR to determine differential peaks.

Genes that have more count data after treatment (increase in expression) or peaks that have higher count data (increase in enrichment from the antibody) are considered "Upregulated". Genes that have fewer counts or lower coverage (decreae in expression) or peaks that have lower coverage (decreased enrichment from the antibody) are considered "Downregulated". Genes that have relatively similar count data or that do not achieve statistical significant are considered "Nondifferential".

### hMeDIP-seq normalization 

In this particular jupyter notebook, I am going to be cleaning up the data. This has many steps, and I am going to save many intermediate files. Most of those files will not be used in the correlation or classification parts of this project, but the files are still useful for this project and may be used in future directions from this sampleset.

The first step to cleaning up the hMeDIP-seq data is to normalize the counts to the total number of reads. This is a Read Count Per Million (RCPM), but this first one does not exclude outliers. 

The next step is to determine which peaks are outliers. The reason for this is that outliers can bias your normalization. As an example, if you have two samples that are identical, and then you spike in DNA that will be in one region for 50% of one of the samples, your normalization will all be wrong. Instead of saying that your spiked-sample has one peak that is significantly different than what is found in the other sample, all of your peaks look like they have half as much coverage as the non-spiked sample and all regions appear differential. If you remove the one spiked region from the normalization, now the unchanged regions all appear the same between the samples (as they should), but the spiked in reads still appear only in the one sample. For this reason, this step includes a linear regression between the two sample types (in this case control and vitamin C), and then a Cook's value of 1 (can be changed), is used to determine the outliers. 

Once you have identified outliers, you can go back to the raw count data and normalize to total read counts excluding the outliers. This gives you the RCPM. This file is saved as the RCPM table.

The next step is to normalize to the RCPM by the width of each peak. This allows you to compare one peak to another. If you don't do this and you compare one peak that covers 100 base pairs (bp) to another peak that covers 100,000 bp, you would just by chance expect that the secon peak has 1,000 times as many reads. To normalize this, we take the RCPM and multiply by 1,000 and divide by the length of the peak. This is referred to as "Fragments Per Kilobase per Million mapped reads" (FPKM, see Trapnell et al. 2012, Nature Protocols). To do this, I use the peaks bed file, which contains information about the start and stop positions of each peak that can be used to calculate the length of each peak. This file is saved as the FPKM table. 

Once reads are normalized correctly, I incorporate the statistical data. For hMeDIP-seq, I use edgeR and a fold change. Peaks that receive a false-discovery rate (FDR, benjamini-hocherberg correction) below 0.05 and a fold change of 1.5x or greater are considered differential.

The next step is to determine which genes are most likely affected by each peak. To do this I used region_analysis, which is a program developed for the ENCODE project. This tells you which gene is closest to the center of the peak and how far the peak is from the transcription start site (TSS) of the gene. Using this data, I can assign each peak to a gene (or classify it as intergenic), and classify each to a region of the gene (upstream promoter, TSS, downstream promoter or gene body). 

There are a lot more peaks than genes, and most genes contain many peaks. As such, the next step is to determine the counts of peaks in each region per gene. Additionally, these peaks are divided by how they respond to vitamin C (upregulated, downregulated or nondifferential). This file is saved as a counts per gene region file. 

### Normalizing RNA-seq data

Like the hMeDIP-seq data, the RNA-seq data needs to be normalized. The first step is to normalize to the total reads within annotated features, and then to find outliers, just like is done for the hMeDIP-seq. You next go back to the raw count data and normalize to the total read counts within annotated features that are not outliers to get the RCPM table. Next, you use the genomic info file to get the lengths of each transcript and calculate the FPKM. This is essentially identical to the first few steps of hMeDIP-seq, except that we don't include reads that don't lie within annotated features for RNA. 

Next, we need to determine differential genes. For RNA-seq data, we use both edgeR and DESeq2. Genes that receive an FDR value below 0.05 by both edgeR and DESeq2 are considered differential. No fold change limit is taken into account and using two algorithms for differential analysis is thought to limit false positives. 

Again, the fold change is used to split differential genes into upregulated and downregulated genes.

Now, the counts of peaks per gene region are integrated with the RNA-seq data to generate a table that contains information that will be used for our classification analysis. Other information that is kept in the file includes gene biotype (e.g. protein coding, long noncoding RNA, antisense RNA, etc.) and the length of the gene. 

The next step is different than was used for hMeDIP-seq. For the hMeDIP-seq, I used the IDR pipeline (from ENCODE) to determine which regions were highly enriched. However, for RNA we look at all annotated regions. Most of these regions, however, are not going to have any reads. This step is determining which genes are above our limit of detection. To do this, I perform the normalization of the genomic input data. The genomic input data was generated for hMeDIP-seq, but has no enrichment. It is meant to represent what we would get if we sequenced just noise. For the RNA, I draw out density plots of the RNA-seq samples and the genomic input. If we use a limit of detection (LOD) below the highest expressed genomic input, then those are essentially false positives for the LOD. If we use a LOD above the bottom of the true genes, then those true genes below that point ar false negatives for our LOD. For this, I look at the density plot and take the point where the RNA-seq data overlaps the genomic input to find the minimum number of false positives and false negatives. Only the transcripts above the determined LOD are used for the rest of this project. 


