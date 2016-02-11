# Miseq tutorial for workshop Microbiome analysis using 16S rRNA gene libraries workshop
  
## Developed for University of British Columbia MICB 506 class
### Modified from protocol by Patrick Schloss by Erick Cardenas Poire


Files you need to run this tutorial:
  
- Installed Mothur program
- stability.files (List of files)
- SILVA-based bacterial reference alignment
- mothur-formatted version of the RDP training set (v.9)


**Note:** This tutorial is based on the Miseq tutorial available [in this page_](<http://www.mothur.org/wiki/MiSeq_SOP>)_. 


You can also run the whole pipeline usind the batch file which containst the script with all the needed commands. You only need to change the name of the *.files* file.
  
To run all the commands inside the batch file run:

```
./mothur stability.batch
```

## Joining pairs

This command joins overlapping paired end sequences and uses the quality information to assign the bases in the overlapping region.


```
make.contigs(file=stability.files, processors=1)
```

This command will also produce several files that you will need down the road:
  
- *stability.trim.contigs.fasta* : Sequences data
- *stability.contigs.groups* : Group identity of each sequence
- *stability.contigs.report* : Contig assembly report
  
  
To have some statistics about the sequences use the summary.seqs* command:
 
 ```
 summary.seqs()
 ```
 
## Quality Control
 
We use the *screen.seqs*  command to remove sequences according to their length and number of ambigous bases (Ns).

 
```
screen.seqs(fasta=stability.trim.contigs.fasta, group=stability.contigs.groups, maxambig=0, maxlength=275)
```

Removing any sequences with ambiguous bases (N) and anything longer than 275 bp. The maximum length depends on the region targeted with the primers.
 

## Dereplication

To decrease the computational time we remove duplicated sequences. Mothur will keep track of the sequences.

```
unique.seqs(fasta=stability.trim.contigs.good.fasta)
``` 

The protocol now uses the *count.seqs* command to create a table that keeps the track of the duplicated files

```
count.seqs(name=stability.trim.contigs.good.names, group=stability.contigs.good.groups)
```

## Align sequences
Now we will align the sequences against the * SILVA-based bacterial reference alignment *. This is a big file with over ##position##  and ## sequences. To reduce computational time we need to trim the alignments


```
pcr.seqs(fasta=silva.bacteria.fasta, start=11894, end=25319, keepdots=F)
```

And we can rename the file, using the system command (if you are using a windows machine use *rename* insted of *mv*)

```
system(mv silva.bacteria.pcr.fasta silva.v4.fasta)
```

Now we can align the sequences to the SILVA master alignment

```
align.seqs(fasta=stability.trim.contigs.good.unique.fasta, reference=silva.v4.fasta)
```

If we run summary.seqs() you will see the distribution of the sequences along the alignment. Outliers tend to have to many insertions (bad alignment), so to have comparable positions we need to remove those sequences. We can also remove here sequences with too many homopolymers (likely errors).

```
screen.seqs(fasta=stability.trim.contigs.good.unique.align, count=stability.trim.contigs.good.count_table, summary=stability.trim.contigs.good.unique.summary, start=1968, end=11550, maxhomop=8)
```

Optional: Now we can remove the positions that only have gaps, they do not contribute with data. The trump character comes from using  reference aligments (in this case SILVA).

```
filter.seqs(fasta=stability.trim.contigs.good.unique.good.align, vertical=T, trump=.)
```
### **Optional**: Dereplicate again with unique.seqs()


The next step helps to reduce the error by grouping closely similar sequences. The algorith with sort the sequences according to their abundance and find sequences that are no more than different in no more than two differences (1 difference per 100 bases)

```
pre.cluster(fasta=current, count=current, diffs=2)
```
## Removing chimeras

Chimeras are artifacts of PCR, hybrids of sequences and need to be detected and removed.   
First we detect them:
  
```
chimera.uchime(fasta=current, count=current, dereplicate=t)
```
And then then we removed them.

```
remove.seqs(fasta=current, accnos=current)
```

## Removing undesirable lineages

We can remove undesired sequences from chloroplast, mitochondria, and archea by first identifying them with *classify.sequences()* and then removing them with *remove.lineage()* 

```
classify.seqs(fasta=current, count=current, reference=trainset9_032012.pds.fasta, taxonomy=trainset9_032012.pds.tax, cutoff=80)
  
remove.lineage(fasta=current, count=current, taxonomy=current, taxon=Chloroplast-Mitochondria-unknown-Archaea-Eukaryota)
```

### Optional if working with mock communities
```
remove.groups(count=current, fasta=current, taxonomy=current, groups=Mock)
```

## Creating a OTUS

The next big step is cluster the sequences into operational taxonomic units (OTUs). The traditional way is to first create a distance matrix with *dist.seqs()* and then use the matrix to cluster the sequences with *cluster()*. However this methods is slow and computationally intensive so we protocol first calls for classifying the sequences, separating the groups in orders (taxonomic group) and then appplying the cluster with those groups. 

This command will create clustering results from different distance tresholds up to the cutoff value.

```
cluster.split(fasta=current, count=current, taxonomy=current, splitmethod=classify, taxlevel=4, cutoff=0.15)
```

Now we can create the distance matrix for an specific distances. In this case we will use the 3% distance.

```
make.shared(list=current, count=current, label=0.03)
```

The generated table is what you need to do most of the diversity analysis (alpha, beta).  

The final step in the protocol is to assign a taxonomic classification to each of the OTUs. We can get the consensus taxonomy for each OTU using *classify.otu()*.
Now 

```
classify.otu(list=current, count=current, taxonomy=current, label=0.03)
```

#####
Need to add export to BIOME

####

 
