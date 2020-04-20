---
title: "Data filtering and normalisation"
teaching: 30
exercises: 0
questions:
- How can I filter and normalize my occurrence data?  
objectives:
- Learn how to remove counts that are below a define minimum  
- Learn how to remove counts that are not shared by a define percentage of the samples  
- Learn how to rarefy the occurrence data  
keypoints:
- ""
---
  
## Table of Contents  
- [1. Data filtering  ](#1-data-filtering)
- [2. Normalization per sample  ](#2-normalisation-per-sample)
  
  
## 1. Data filtering  
  
In the remainder of the data analyses we want to focus on the differences between the samples, the bacterial community composition and the beta-diversity. For this it is necessary to prepare the data in a way that improves the comparability of the samples.  
As microbiome data sets are usually sparse, it is important to filter the data set. Indeed, data filtering aims to remove low quality or uninformative variables (OTU/ASV) to improve downstream statistical analysis.  
For example, we can filter variables (OTU/ASV) with very low number of counts in only a few samples, which are likely due to sequencing errors or low-level contaminations. In this example, we will keep OTU that have at least 2 counts in at least 11% of the samples. Indeed, we have here 18 samples and 3 replicates per treatment and we want, for example, to have at least 2 counts in at least two samples (2/18=0.111).  
  
> ## Remark
>  First, data filtering should be applied on the raw data. Then, we are using here the phyloseq object: *data_phylo* because we are using a plyloseq function to filter 
the raw data.  
{: .callout}
  
~~~
{: .language-r}{r}
data_phylo_filt = filter_taxa(data_phylo, function(x) sum(x > 2) > (0.11*length(x)), TRUE) # filter the OTU data using filter_taxa function included in phyloseq package
data_otu_filt = data.frame(otu_table(data_phylo_filt)) # create a separated file
~~~
{: .language-r}   
  
**QUESTIONS:** How many zeros are present in the filtered OTU table? What is the percentage of zeros in the filtered OTU table? Plot the number of non zero values for each OTU (in the same way as for the nonfiltered data). Interpret these results.  
**SOLUTIONS:**  
~~~
{: .language-r}{r}
sum(data_otu_filt==0)
sum(data_otu_filt==0)/(dim(data_otu_filt)[2]*dim(data_otu_filt)[1])*100
hist(as.matrix(data_otu_filt), max(data_otu_filt), right=FALSE, las=1, xlab = "Occurrence value", ylab = "Frequency", main = "Occurrence frequency")
min(colSums(data_otu_filt))
non_zero<-0*1:dim(data_otu_filt)[2]
for (i in 1:dim(data_otu_filt)[2]){
  non_zero[i]<-sum(data_otu_filt[,i] != 0)
  }
plot(non_zero, xlab = "OTU", ylab = "Frequency", main="Number of non zero values", las=1)
# You can see that we removed 3867 OTU, which were extremly rare and mostly found in only few samples. We deacresed the number and percentage of zeros in our data set by removing these OTU.
~~~
{: .language-r}
  
  
## 2. Normalization per sample  
  
In the further analyses, we will compare counts between samples. To do so, we will have first to normalise the filtered occurrence table by sample in order to obtain the same library size for every samples.  
Different methods exist to normalise microbiome data: proportions and rarefying were commonly used for long time but other methods were also developed, such as DESeq2 or edgeR‐TMM, which are commonly used in RNA-seq data analyses. While there is still no consensus on which normalisation method should be used for microbiome data, microbial ecologists prefer the proportion or the rarefying methods. Indeed, as explained McKnight and colaborators (DOI: 10.1111/2041-210X.13115) DESeq2 or edgeR‐TMM are recommended based on studies that focused on differential abundance testing and variance standardization, rather than community-level comparisons (*i.e.* beta-diversity). Moreover, standardizing the within-sample variance across samples may suppress differences in species evenness, potentially distorting community level patterns, and the log transformations can exaggerate the importance of differences among rare OTU, while suppressing the importance of differences among abondant OTU.  
For proportion, each read count is divided by the total sum of all reads of the corresponding sample. As a results, read sums for all the different samples are then equal to 1 and OTU occurrences are between 0 and 1.  
For rarefying, the library size is arbritary defined for every samples as the smallest library size observed for all the samples (here, *min(sum_seq)*). Then, simple random samples without replacement are performed for every sample using the raw data.  
  
**QUESTION:** What do you think are the pros and cons of these two methods?  
  
> ## Remark 
> When differences between library sizes is high (such as 10 fold change), it is recommended to use rarefying. As we usually observe 10 fold change in the library sizes, we will normalize here using the rarefying method.    
> The normalization should be done on the filtered data, the phyloseq object *data_phylo_filt*.  
 {: .callout}
  
Rarefy the data. 
~~~
set.seed(1782) # set seed for analysis reproducibility
OTU_filt_rar = rarefy_even_depth(otu_table(data_phylo_filt), rngseed=T, replace=FALSE) # rarefy the raw data using Phyloseq package
data_otu_filt_rar = data.frame(otu_table(OTU_filt_rar)) # create a separated file
data_phylo_filt_rar <- phyloseq(OTU_filt_rar, TAX, SAM) # create a phyloseq object
~~~
{: .language-r}
  
**QUESTION:** What is the number of counts per sample for the rarfied data set?  
**SOLUTIONS:** 7750

~~~
rowSums(data_otu_filt_rar)
~~~
{: .language-r}  