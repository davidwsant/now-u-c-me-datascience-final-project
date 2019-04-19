<!-- vim: set textwidth=80 : -->

## Now You "C" Me Group for DataScience COMP 5360 Spring 2019

I am aware that python notebooks are horrible for collaboration, so new versions will just receive a new title (probably containing the date) to avoid messing things up with git.

For the initial commit, I have cleaned all of the data for the ARPE-19 cells.
I will add the jupyter notebook as well as the input and output files to the repository for access by the team.

### Info about the updates that have been pushed as of April 17, 2019

I have now cleaned up the data for all three datasets of combined RNA-seq and
hMeDIP-seq (ARPE-19 vitamin C, Schwann Cell vitamin C, and Schwann Cell cAMP).
Additionally, I have taken the markdown comments form the ARPE-19 cleaning
notebook and put them into their own separate document
(Info_about_Data_Cleaning.md) so that it can be viewed separately. Each dataset
was explored thoroughlly in the cleaning and quality control, but further
exploration in clustering and then analysis in classification will be added in
the next steps.

### Info about the updates that have been pushed as of April 19, 2019

Classification analysis has been completed. The code for this is in the notebooks RM_Analyzing_Cleaned_Data_KNN_Apr16.ipynb and Classification_of_genes.ipynb. Tests were run to determine the parameters for Decision Trees (DTrees is notebook), K-Nearest Neighbors (KNN) and Support Vector Machines (SVM). The parameters were tested on both the ARPE dataset and the Schwann Cell cAMP dataset. Although these datasets were very different, the parameters ended up being almost the exact same. All three machine learning models were trained on any given dataset and then tested on all datasets. Given three datasets for training, three datasets for testing and three different classification methods, this meant 27 machine learning tests.

### Merging the jupyter notebooks

Our class Canvas page references "a" notebook, so I found a method that can combine all of the notebooks into one. There is a python package called nmberge. The commands to merge the notebook (from the command line) were as follows:

`pip install nbmerge`
`nbmerge ARPE_normalizing.RNA.and.hMeDIP.ipynb SC_cAMP_normalizing.RNA.and.hMeDIP.ipynb SC_VitC_normalizing.RNA.and.hMeDIP.ipynb Clustering_and_Data_Exploration.ipynb RM_Analyzing_Cleaned_Data_KNN_Apr16.ipynb Classification_of_genes.ipynb > Now-you-C-me_full_Analysis_Notebooks.ipynb`

Unfortunately, this makes for a very large notebook and it is hard for my browser (Chrome) to handle something this large. In a text editor I can see that I have 51,586 lines in the merged notebook, and we have not yet added the link to the 3-minute video.

## Conclusions

After all data cleaning, clustering and classification, we have some results. The method of data cleaning worked quite well, with the exception that finding the limit of detection using the genomic input sequencing read counts seems to predict too high of a limit of detection. I think I will need to find a list of genomic regions that should not be expressed and take read counts in those regions as my "not expressed" region baseline instead of the genomic input files.

Clustering appears to have worked quite well. I think I need to run DBSCAN with a higher number of minimum samples next time, maybe 50 genes instead. Anyways, both K-means and DBSCAN returned one predominant cluster of genes that had almost no 5hmC peaks or no 5hmC peaks. Aside from that, there are a few other large peaks. I did not have time to do it for this project, but I would like to go through this and see if the genes within groups are regulated by similar mechanisms (same transcription factor, maybe same enhancer region). This can be done by pathway analysis, but was outside of the scope of the DataScience class so was not included in the notebooks. Overall, I thought the clustering looked successful and I am interested to see if there are other similarities between genes within clusters.

Classification was less successful. Both SVM and KNN were did not predict any genes to be downregulated in the ARPE dataset. DTrees predicted a small number. I think this is because there are very few downregulated genes and they probably have peaks very similar to the nondifferential genes. I tried training on the ARPE dataset and then testing on all 3 datasets, and I was almost just as successful on the SC Vitamin C dataset as I was on the ARPE dataset. However, the cAMP dataset would have been just as well off drawing marbles out of a bag. When I tried training on SC Vitamin C, I got the same results: it works for the ARPE dataset but is useless at predicting the outcome of genes in the SC cAMP dataset. When I train the models on the SC cAMP dataset and then test it on the ARPE or SC Vitamin C datasets, the results are actually much worse than baseline (Naive Bayes would have given ~52% but the machine learning models gave between 18% and 33%). All three machine learning methods returned similar results. This shows me that the cAMP dataset is vastly different in regulation from the vitamin C datasets. This is possibly just because of the transcriprtion factors (CREB, CREM, ATF1) that are regulated by cAMP, but I think it also has to do with the fact that about 2/3 of the genes in the cAMP dataset were differential. Any way about it, I think the classification was not very successful. For future work I am more likely to look into the clustering.
