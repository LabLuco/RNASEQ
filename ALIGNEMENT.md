
## Quick overview

---

Here we describe the **alignment** part of the workflow.



## Set up Tools

---

First you need to check that the following tools are installed on server/computer.


Scripts available here are in Python3.  
It's not required but advised to install Conda if python3 is not set up on your computer.   
It will make things easier then for installing tools or switching to older python version if needed.

_**Conda**_ : [here](https://www.continuum.io/downloads)

ALIGNMENT : 

_**STAR**_ : Aligner [here](https://github.com/alexdobin/STAR)

_**trim-galore**_ : Aligner [here](https://anaconda.org/bioconda/trim-galore)

_**tpmcalculator**_ : Compute TPM values f [here](https://anaconda.org/bioconda/trim-galore)

_**Salmon**_ : Compute TPM values from fastq [here](https://github.com/COMBINE-lab/salmon)

_**Samtools**_ : Bam handler [here](http://www.htslib.org/download/)

_**wigToBigWig**_ : Include in KentTools suite [here](http://hgdownload.soe.ucsc.edu/downloads.html#source_downloads)

## Set up Files

Download  fasta sequence of the genome of interest. (Here we use PRIM_ASSEMBLY from GENCODE R25 based on ENSEMBL 85 for Human and GENCODE M15 based on ENSEMBL 90 for Mouse.  
Create this file with chromosome files.  

```shell
conda install pyfaidx
faidx yourgenome.fasta -i chromsizes > yourgenome.genome.sizes
```

Also you need to create index for STAR :  

```
STAR --runMode genomeGenerate --genomeDir /home/jean-philippe.villemin/mount_archive2/commun.luco/ref/indexes/GRCm38_PRIM_GENCODE_M15/ --genomeFastaFiles /home/jean-philippe.villemin/mount_archive2/commun.luco/ref/genome/GRCm38_PRIM_GENCODE_M15/GRCm38.primary_assembly.genome.fa --runThreadN 30 --sjdbGTFfile /home/jean-philippe.villemin/mount_archive2/commun.luco/ref/genes/GRCm38_PRIM_GENCODE_M15/gencode.vM15.primary_assembly.annotation.gtf
```

Generate the fasta file index :  
```shell
samtools faidx reference.fa 
```
This creates a file called reference.fa.fai

Generate the sequence genome.2bit :  
```shell
faToTwoBit genome.fa genome.2bit
```


Last step  : 

Download the transcripts from gencode for your genome version and then inside a dir called transcriptome for instance,
you will create indexes for  Salmon as follows : 

```shell  
salmon index -t gencode.vM15.transcripts.fa.gz -i gencode.m15.transcripts.index --type quasi -k 31  
```
## Set up ConfigFile.json

---

We use a json file to create a configuration file before doing your alignment. 

The json file is a *key:value* listing which defines all parameters for the pipeline.
- number of cores used
- path to output/input directory
- name of analyse
- ...

See config directory. You will find an example called paired.set1_align.json for a test dataset.


## Set up init.json

For the alignement part, tpmcalculator and salmon path are the only ones that are really used from this file. 


## Modified hard coded stuffs in alignement.py

There is a path to a temporary directory used by STAR that is hard coded in alignement.py that need to be modified directly in the code.


## Launch Alignment

---

```shell
	python3 pathTo/alignment.py -c pathToConfigFile/condition.json
```

You can test the pipeline on a  short dataset provided in test directory.

It will launch an alignment with STAR.
It will creates a directory (path defined in your configuration file) with inside :

1. Bam file.
2. Bigwig files per strand normalized by CPM (count per millions) for visualization purpose in UCSC.
3. Reads count per gene - Sample_ReadsPerGene.tab (used for the gene expression step).
4. Intermediate files used in the following steps of the pipeline.


You need to use this on each sample/replicate (or pool the replicate into one and you will get one bigwig) 

Finally you get the following directories as output : 


![alt text](https://github.com/LucoLab/RNASEQ/blob/master/img/output_alignment.png "Outputs")

![alt text](https://github.com/LucoLab/RNASEQ/blob/master/img/output_alignment_open.png "Outputs")

