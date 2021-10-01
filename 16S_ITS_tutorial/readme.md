# Start the analysis
In this tutorial you’ll use QIIME 2 to perform an analysis of soil samples from the Atacama Desert in northern Chile.
**You can find the starting tutorial on the official page of QIIME2** ( [link text](https://docs.qiime2.org/2020.2/tutorials/atacama-soils/))

"The Atacama Desert is one of the most arid locations on Earth, with some areas receiving less than a millimeter of rain per decade. The soil microbiomes profiled in this study follow two east-west transects, **Baquedano** and **Yungay**, across which average soil relative humidity is positively correlated with elevation (higher elevations are less arid and thus have higher average soil relative humidity). Along these transects, pits were dug at each site and soil samples were collected from three depths in each pit.""

![alt text](https://www.nationalgeographic.com/content/dam/travel/2016-digital/chile/atacama/valley-moon-atacama-desert-chile.ngsversion.1483561804450.adapt.1900.1.jpg)

**Experimental information**:
![](V4_atacama.png)
*   The v4 region of the 16S rRNA gene was amplified from all community DNA extracts using barcoded primers 515F/806R.

*   Amplicon sequencing was performed Illumina MiSeq system and MiSeq control software version 2.2.0.




You can find the paper here:
[link text](https://msystems.asm.org/content/2/3/e00195-16.abstract)

or in Reference folder of the course.








Start by creating a directory to work in.


```
mkdir qiime2-atacama-tutorial
cd qiime2-atacama-tutorial
```

## The Metadata file

Metadata play a key rule in every ecological study. For how is familiar with QIIME 1 this file corresponde to "mapping file" or for R user is the env file in *vegan* package.

QIIME 2 metadata is most commonly stored in a **TSV** (i.e. tab-separated values) file. These files typically have a .tsv or .txt file extension, though it doesn’t matter to QIIME 2 what file extension is used.

TSV files are simple text files used to store `tabular data`, and the format is supported by many types of software, such as editing, importing, and exporting from spreadsheet programs and databases. Thus, it’s usually straightforward to manipulate QIIME 2 metadata using a software like Microsoft Excel or (better) Google Sheets to edit and export your metadata files.

You can find the [file](https://docs.google.com/spreadsheets/d/1a1NFqpBjwb8Ul0c4O68IVFb9JUn5BjLdyHr422LMfE0/edit#gid=1988763045) in the Lab folder of today or download it directly from qiime to the server by running the command:

```
wget \
  -O "sample-metadata.tsv" \
  "https://data.qiime2.org/2020.8/tutorials/atacama-soils/sample_metadata.tsv"
```


**For avoid rewriting of the file please copy this file in an other folder.**

This *sample-metadata_16S.tsv* file is used throughout the rest of the tutorial.


Since there is no universal standard for TSV files, it is important understand how QIIME 2 will interpret the file’s contents to get the most out of your (meta)data!

Sample and feature metadata files stored in Google Sheets can be validated using **Keemei**:  
 1. Select *Add-ons* 
 2. Keemei  
 3. Validate *QIIME 2 metadata file* to validate metadata stored in Google Sheets.  

QIIME 2 will also automatically validate a metadata file anytime it is used by the software. However, using Keemei to validate your metadata is recommended because a report of all validation errors and warnings will be presented each time Keemei is run.



## Obtain Data

Creating a new folder in the folder qiime2-atacama-tutorial



```
mkdir emp-paired-end-sequences
```

For this tutorial we use a subsample of the real data (10%) that you can find follow the links below:


```
wget \
  -O "emp-paired-end-sequences/forward.fastq.gz" \
  "https://data.qiime2.org/2020.8/tutorials/atacama-soils/10p/forward.fastq.gz"
```


```
wget \
  -O "emp-paired-end-sequences/reverse.fastq.gz" \
  "https://data.qiime2.org/2020.8/tutorials/atacama-soils/10p/reverse.fastq.gz"
```


```
wget \
  -O "emp-paired-end-sequences/barcodes.fastq.gz" \
  "https://data.qiime2.org/2020.8/tutorials/atacama-soils/10p/barcodes.fastq.gz"
```

## Start QIIME2 session

```
source activate qiime2-2020.8
```

## Pipeline Overview
Here is an overview of the general steps of the QIIME2 pipeline:

* **Step 1**: Importing and demultiplex data, summarize the results, and examing quality of the reads.
* **Step 2**: Quality controlling sequences and building Feature Table and Feature Data
* **Step 3**: Summarizing Feature Table and Feature Data
* **Step 4**: Assigning Taxonomy
* **Step 5**: Generating a phylogenetic tree
* **Step 6**: Analyzing Alpha and Beta diversities


## STEP1: Importing and Demultiplex files

The sequences that you just downloaded are defined in QIIME2 as EMPPairedEndSequences. **What that mean?**

QIIME 2 supports various data formats for sequences files and BIOM tables, however the descriptions of these formats are still being developed. Some common data formats are described in the [Importing data tutorial](https://docs.qiime2.org/2020.2/tutorials/importing/).


---


Paired-end “**Earth Microbiome Project (EMP) protocol**” formatted reads should have **three** fastq.gz files total:

*   one fastq.gz file that contains the forward sequence reads,
*   one fastq.gz file that contains the reverse sequence reads,
*   a third fastq.gz file containinig the associated barcode reads.  

In this format, sequence data is still **multiplexed**.  

**The order of the records** in the fastq.gz files defines the association between a sequence read and its barcode read (i.e. the first barcode read corresponds to the first sequence read, the second barcode to the second read, and so on.)


---

Take a look as the sequences are (in the most cases) generated

![alt text](https://sfvideo.blob.core.windows.net/sitefinity/images/default-source/product-page-images/next-generation-sequencing/ngs_adapter_designs.png?sfvrsn=8ce20807_8)

For more details

 [Illumina support](https://support.illumina.com/content/dam/illumina-support/documents/documentation/system_documentation/miseq/indexed-sequencing-overview-guide-15057455-04.pdf)

 [earth microbiome 16S protocols](http://www.earthmicrobiome.org/protocols-and-standards/16s/)



```
qiime tools import \
   --type EMPPairedEndSequences \
   --input-path emp-paired-end-sequences \
   --output-path emp-paired-end-sequences.qza
```


```
qiime tools import --show-importable-formats --help
```

### Explore QIIME2 Objects

By importing the sequences you have generated your first object in QIIME2:  

`emp-paired-end-sequences.qza`  

Data produced by QIIME 2 exist as QIIME 2 **artifacts**. A QIIME 2 artifact contains data and metadata.

Since QIIME 2 works with artifacts instead of data files (e.g. FASTA files), you must create a QIIME 2 artifact by importing data.  
In the coming days we will see how to import and export different objects from QIIME2 and we will always use artifacts.

Artifacts enable QIIME 2 to track, in addition to the data itself, the provenance of how the data came to be. With an artifact’s provenance, you can trace back to all previous analyses that were run to produce the artifact, including the input data used at each step.

This requires the sample metadata file, and you must indicate which column in that file contains the **per-sample barcodes**. In this case, that column name is **BarcodeSequence**.
In this data set, the barcode reads are the reverse complement of those included in the sample metadata file, so we additionally include the  '--p-rev-comp-mapping-barcodes ' parameter.

```
qiime demux emp-paired \
  --m-barcodes-file sample-metadata.tsv \
  --m-barcodes-column barcode-sequence \
  --p-rev-comp-mapping-barcodes \
  --i-seqs emp-paired-end-sequences.qza \
  --o-per-sample-sequences demux-full.qza \
  --o-error-correction-details demux-details.qza
```

After demultiplexing, we can generate and view a summary of how many sequences were obtained per sample.

```
qiime demux summarize \
  --i-data demux-full.qza \
  --o-visualization demux.qzv
```

With the command  `demux summarize` you generated a new type of objet: a file with `.qzv` extencion

*Visualizations* are another type of data generated by QIIME 2.  
Visualizations contain similar types of metadata as QIIME 2 artifacts, including provenance information.

Both files (`qza` and `qzv`) can be **extracted** with a Compression/Decompression software (ex. gzip or unzip).

Let's look  together,  download on your PC and unzip it. **What's inside?**

There are also different ways to [export QIIME2 objects](https://docs.qiime2.org/2020.2/tutorials/exporting/).
But remember that all existing provenance will be lost after exporting the files.

***In most of cases you can have already demultiplex data...How to import the files? [We will discuss it in the next days](../DataImport_and_Tax_management/readme.md)***

### Visualize with QIIME2 viewer

You can use [https://view.qiime2.org](https://view.qiime2.org) to easily view QIIME 2 artifacts and visualizations files (generally .qza and .qzv files (see later)) without requiring a QIIME installation.

Open it and drag and drop your qza or qzv file
(You must download the file you want to upload,on your computer.)
  
# Step2: Quality controlling sequences and building Feature Table and Feature Data

The polymerase will read through the amplicon, the primer, the barcode, and on into the adapter sequence. This is non-biological DNA that will cause major issues downstream, e.g., with sequence classification. So we want to trim primers from either end of the sequence to eliminate read-through issues

## Quality filter of 16S

After demultiplexing reads, we’ll look at the sequence quality based randomly selected samples, and then denoise the data.

The plot on the left presents the quality scores for the **forward** reads, and the plot on the right presents the quality scores for the **reverse** reads. We’ll use these plots to determine what trimming parameters we want to use for denoising with DADA2, and then denoise the reads using dada2 denoise-paired.

In this example we have **150-base forward and reverse reads**.  
Since we need the reads to be long enough to overlap when joining paired ends, the first thirteen bases of the forward and reverse reads are being trimmed, but no trimming is being applied to the ends of the sequences to avoid reducing the read length by too much.

In this example, the same values are being provided for `--p-trim-left-f` and `--p-trim-left-r` and for `--p-trunc-len-f` and `--p-trunc-len-r`, but that is not a requirement.  
We're also using a multi-threading command to split the processing across multiple CPUs, which will be very useful when performing your own analysis with larger datasets.

```
qiime dada2 denoise-paired \
  --i-demultiplexed-seqs demux-full.qza \
  --p-trim-left-f 13 \
  --p-trim-left-r 13 \
  --p-trunc-len-f 150 \
  --p-trunc-len-r 150 \
  --o-table table.qza \
  --p-n-threads 2 \
  --o-representative-sequences rep-seqs.qza \
  --o-denoising-stats denoising-stats.qza
```

The second command produces a visualisation table of the denoising process so we can see the effect it had on the data.
```
qiime metadata tabulate --m-input-file denoising-stats.qza --o-visualization denoising-stats.qzv
```

# Step 3: Summarizing Feature Table and Feature Data

## Summaraize 16S data
You  now will have artifacts containing the feature table and corresponding feature sequences.  
You can generate summaries of those as follows.

```
qiime feature-table summarize \
  --i-table table.qza \
  --o-visualization table.qzv \
  --m-sample-metadata-file sample-metadata.tsv

qiime feature-table tabulate-seqs \
  --i-data rep-seqs.qza \
  --o-visualization rep-seqs.qzv

```
#### 454 and Ion Torrent data
To whom it may be interested at this [**link**](https://benjjneb.github.io/dada2/faq.html#can-i-use-dada2-with-my-454-or-ion-torrent-data) you may find some suggestion to apply **DADA2** on 454 and Ion Torrent data.

# Step 4 Taxonomy assignment 

## 16S taxonomy assignment 
The QIIME 2 plugin [feature-classifier](https://docs.qiime2.org/2020.2/plugins/available/feature-classifier/) supports taxonomic classification of features using a variety of methods, including:  
 1. **Naive Bayes**;  
 2. **vsearch**; 
 3. **BLAST+**.  

The `q2-feature-classifier` contains three different classification methods. **classify-consensus-blast** and **classify-consensus-vsearch** are both *alignment-based methods*, that find a consensus assignment across N top hits. These methods take reference database `FeatureData[Taxonomy]` and `FeatureData[Sequence]` files directly, and do not need to be pre-trained.
We’ll do that using a pre-trained ***Naive Bayes classifier*** and the **q2-feature-classifier plugin**.  
This classifier was trained on the *SILVA 138 NR99 collection*, where the sequences have been trimmed to only include 250 bases from the region of the 16S that was sequenced in this analysis (the V4 region, bound by the 515F/806R primer pair).  
We’ll apply this classifier to our sequences, and we can generate a visualization of the resulting mapping from sequence to taxonomy.

The first step in this process is to assign taxonomy to the sequences in our `FeatureData[Sequence]` QIIME 2 artifact. 

```
qiime feature-classifier classify-sklearn \
  --i-classifier ~/Share/SILVA_138_NR99_515F_806R_classifier.qza \
  --i-reads rep-seqs.qza \
  --o-classification taxonomy_16S_SKLEARN.qza 
```

Now create the barplot for data visualization:  
```
qiime metadata tabulate \
  --m-input-file taxonomy_16S_SKLEARN.qza \
  --o-visualization taxonomy_16S_SKLEARN.qzv
```

```
qiime taxa barplot \
  --i-table table.qza \
  --i-taxonomy taxonomy_16S_SKLEARN.qza \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization taxa-bar-plots_16S_SKLEARN.qzv
```

# Step 5: Generating a phylogenetic tree
We have to generate phylogenetic tree because QIIME2 support different phylogenetic diversity metrics, including Faith’s Phylogenetic Diversity and weighted and unweighted UniFrac. 

In addition to counts of features per sample (i.e., the data in the `FeatureTable[Frequency]` QIIME 2 artifact), these metrics require a rooted phylogenetic tree relating the features to one another. This information will be stored in a `Phylogeny[Rooted]` QIIME 2 artifact. To generate a phylogenetic tree we will use **align-to-tree-mafft-fasttree** pipeline from the q2-phylogeny plugin`.

* First, the pipeline uses the **mafft** program to perform a multiple sequence alignment of the sequences in our `FeatureData[Sequence]` to create a `FeatureData[AlignedSequence]` QIIME 2 artifact. 

* Next, the pipeline **masks** (or filters) the alignment to remove positions that are highly variable. These positions are generally considered to add noise to a resulting phylogenetic tree. 

* Following that, the pipeline applies **FastTree** to generate a phylogenetic tree from the masked alignment. The FastTree program creates an unrooted tree

* The final step in this section midpoint **rooting** is applied to place the root of the tree at the midpoint of the longest tip-to-tip distance in the unrooted tree.


```
qiime phylogeny align-to-tree-mafft-fasttree \
  --i-sequences rep-seqs.qza \
  --o-alignment aligned-rep-seqs.qza \
  --o-masked-alignment masked-aligned-rep-seq.qza \
  --o-tree unrooted-tree_16S.qza \
  --o-rooted-tree rooted-tree_16S.qza
```

# Step 6: Analyzing Alpha and Beta diversities

First, lets look at alpha diversity as a function of sequencing depth, as a test of our sequencing run:

```
qiime diversity alpha-rarefaction \
  --i-table table.qza \
  --i-phylogeny rooted-tree_16S.qza \
  --p-max-depth 8000 \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization alpha-rarefaction.qzv
```

QIIME 2’s diversity analyses are available through the [`q2-diversity plugin`](https://docs.qiime2.org/2020.2/plugins/available/diversity/), which supports computing alpha and beta diversity metrics, applying related statistical tests, and generating interactive visualizations. We’ll first apply the core-metrics-phylogenetic method, which rarefies a `FeatureTable[Frequency]` to a user-specified depth, computes several alpha and beta diversity metrics, and generates **Principle Coordinates Analysis (PCoA)** plots using Emperor for each of the beta diversity metrics. The metrics computed by default are:

**Alpha diversity**
* Shannon’s diversity index (a quantitative measure of community richness)
* Observed OTUs (a qualitative measure of community richness)
* Faith’s Phylogenetic Diversity (a qualitative measure of community richness that incorporates phylogenetic relationships between the features)
* Evenness (or Pielou’s Evenness; a measure of community evenness)

**Beta diversity**
* Jaccard distance (a qualitative measure of community dissimilarity)
* Bray-Curtis distance (a quantitative measure of community dissimilarity)
* unweighted UniFrac distance (a qualitative measure of community dissimilarity that incorporates phylogenetic relationships between the features)
* weighted UniFrac distance (a quantitative measure of community dissimilarity that incorporates phylogenetic relationships between the features)

An important parameter that needs to be provided to this script is **--p-sampling-depth**, which is the even sampling (i.e. rarefaction) depth.

```
qiime diversity core-metrics-phylogenetic \
  --i-phylogeny rooted-tree_16S.qza \
  --i-table table.qza \
  --p-sampling-depth 1000 \
  --m-metadata-file sample-metadata.tsv \
  --output-dir core-metrics-results_16S
````
From these outputs, a range of datasets and visualisations are generated that can be tested in the next steps, or explored visually. Let's look at the 'emperor' PCoA results.

## Time to test

We’ll first test for associations between **categorical** metadata columns and alpha diversity data. We’ll do that here for the Faith Phylogenetic Diversity (a measure of community richness) and evenness metrics.

```
qiime diversity alpha-group-significance \
  --i-alpha-diversity core-metrics-results_16S/faith_pd_vector.qza \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization core-metrics-results_16S/faith-pd-group-significance_16S.qzv
```

```
qiime diversity alpha-group-significance \
  --i-alpha-diversity core-metrics-results_16S/evenness_vector.qza \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization core-metrics-results_16S/evenness-group-significance.qzv
```

Using **Pearson** and **Spearman** correlation it is possible to determine whether numeric sample metadata columns are correlated with
alpha diversity.

```
qiime diversity alpha-correlation \
  --i-alpha-diversity core-metrics-results_16S/faith_pd_vector.qza \
  --m-metadata-file sample-metadata.tsv \
  --p-method spearman \
  --o-visualization core-metrics-results_16S/faith_pd_correlation.qzv
````

Next we’ll analyze sample composition in the context of categorical metadata using **PERMANOVA** (first described in [Anderson (2001)](https://onlinelibrary.wiley.com/doi/full/10.1111/j.1442-9993.2001.01070.pp.x)) using the beta-group-significance command. The following commands will test whether distances between samples within a group, are more similar to each other then they are to samples from the other groups. In this case we test only for the Transect_name and for the vegetation

```
qiime diversity beta-group-significance \
  --i-distance-matrix core-metrics-results_16S/unweighted_unifrac_distance_matrix.qza \
  --m-metadata-file sample-metadata.tsv \
  --m-metadata-column transect-name \
  --o-visualization core-metrics-results_16S/unweighted-unifrac-tran-name-significance.qzv \
  --p-pairwise
```
```
qiime diversity beta-group-significance \
  --i-distance-matrix core-metrics-results_16S/unweighted_unifrac_distance_matrix.qza \
  --m-metadata-file sample-metadata.tsv \
  --m-metadata-column vegetation \
  --o-visualization core-metrics-results_16S/unweighted-unifrac-subject-group-significance.qzv \
  --p-pairwise
  ```

[**Back to the program**](../README.md)  