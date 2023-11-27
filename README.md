# Introduction
Welcome to the Giguère lab's GitHub page!

Here you will find shared lab resources and workflows for **ChIP-Seq**, **RNA-Seq**, and **ATAC-Seq** data analysis.

## Table of Contents
> 1. [Getting Started](https://github.com/giguerelab/giguerelab/edit/main/README.md#getting-started)
> 2. Downloading Data from Databases
> 3. [Pipelines](https://github.com/giguerelab/giguerelab/edit/main/README.md#pipelines)
>    - [RNA-Seq](https://github.com/giguerelab/giguerelab/edit/main/README.md#rna-seq)
>    - [ChIP-Seq](https://github.com/giguerelab/giguerelab/edit/main/README.md#chip-seq)
>    - [ATAC-Seq](https://github.com/giguerelab/giguerelab/edit/main/README.md#atac-seq)
> 4. [Additional Resources](https://github.com/giguerelab/giguerelab/edit/main/README.md#additional-resources)
>    - [Databases/Data Visualization Tools](https://github.com/giguerelab/giguerelab/edit/main/README.md#databases)
>    - [AI Tools](https://github.com/giguerelab/giguerelab/edit/main/README.md#ai-tools)
  
## Getting Started
In order to perform these workflows, you will need access to high-performance computing (HPC). To do this, you will need to apply for an account on [Compute Canada](https://ccdb.alliancecan.ca/). You will need your PI to approve your account. Once your account has been approved, you will have access to the CCDB HPC. This account gives you access to the following clusters:
- [Béluga (ÉTS, Montréal)(1 TB Project Storage)](https://docs.alliancecan.ca/wiki/B%C3%A9luga/en)
- [Cedar (SFU, Vancouver)(1 TB Project Storage)](https://docs.alliancecan.ca/wiki/B%C3%A9luga/en)
- [Graham (UWaterloo, Waterloo)(1 TB Project Storage)](https://docs.alliancecan.ca/wiki/Graham)
- [Narval (ÉTS, Montréal)(1 TB Project Storage)](https://docs.alliancecan.ca/wiki/Narval/en)
- [Niagara (UToronto, Toronto)](https://docs.alliancecan.ca/wiki/Niagara)

> I recommend that you log in to the Béluga cluster.

To securely connect to a remote cluster, open a shell or terminal (bash preferably) and type the following command to login to a secure shell (SSH) which will look something like this: 

`ssh myaccount@beluga.computecanada.ca`

Afterwards, you will be prompted to enter your Compute Canada password. 

Once you're connected to the beluga cluster, you will need modify your bash_profile to set up your user environment to access the software and scripts used by GenPipes. To add the tool path to your bash_profile, copy/paste the following command into your bash shell:

```
## open bash_profile
nano $HOME/.bash_profile
```

To load the GenPipes modules, paste the following lines of code and save the file by exiting (Ctrl-X) and saving the changes. Don't change the name of the file.

```
umask 0002

## GenPipes/MUGQIC genomes and modules
export MUGQIC_INSTALL_HOME=/cvmfs/soft.mugqic/CentOS6
module use $MUGQIC_INSTALL_HOME/modulefiles
module load mugqic/python/3.9.1
module load mugqic/genpipes/<latest_version>
export JOB_MAIL=<my.name@my.email.ca>
export RAP_ID=<my-rap-id>
```

**You will need to replace the text in “<>” with your account and GenPipes software version specific information.** Your RAP_ID will have the format **def-groupname**. To check out what the latest available version of GenPipes is, use the following command:

```
module avail 2>&1 | grep mugqic/genpipes
```

For MUGQIC analysts, you can add the following lines to your $HOME/.bash_profile, but this is not required to access GenPipes modules. 

```
umask 0002

## MUGQIC genomes and modules for MUGQIC analysts

HOST=`hostname`;

DNSDOMAIN=`dnsdomainname`;

export MUGQIC_INSTALL_HOME=/cvmfs/soft.mugqic/CentOS6

if [[ $HOST == abacus* || $DNSDOMAIN == ferrier.genome.mcgill.ca ]]; then

  export MUGQIC_INSTALL_HOME_DEV=/lb/project/mugqic/analyste_dev

elif [[ $HOST == ip* || $DNSDOMAIN == m  ]]; then

  export MUGQIC_INSTALL_HOME_DEV=/project/6007512/C3G/analyste_dev

elif [[ $HOST == cedar* || $DNSDOMAIN == cedar.computecanada.ca ]]; then

  export MUGQIC_INSTALL_HOME_DEV=/project/6007512/C3G/analyste_dev


elif [[ $HOST == beluga* || $DNSDOMAIN == beluga.computecanada.ca ]]; then

  export MUGQIC_INSTALL_HOME_DEV=/project/6007512/C3G/analyste_dev

fi

module use $MUGQIC_INSTALL_HOME/modulefiles $MUGQIC_INSTALL_HOME_DEV/modulefiles
module load mugqic/python/3.9.1
module load mugqic/genpipes/<latest_version>

export RAP_ID=<my-rap-id>
```

When you modify your bash_profile, you will need to log out and log back in for the changes to take effect, or you can run the following command:

```
source $HOME/.bash_profile
```

Now you have set up the environment to deploy the GenPipes pipelines! You also have access to hundreds of bioinformatics tools. To see this list, type the following command:

```
module avail mugqic/
```

To load a tool available on Compute Canada servers (e.g., samtools) use the following command:

```
# module add mugqic/<tool><version>
module add mugqic/samtools/1.4.1

# Now samtools 1.4.1 is available for use in your account environment. To check, run the following command:
samtools
```

Several of the GenPipes pipelines may require referencing genomes. The reference genomes are stored in `$MUGQIC_INSTALL_HOME/genomes/species/` and can be viewed using the command 

```
ls $MUGQIC_INSTALL_HOME/genomes/species
```

All genome-related files, including indices for different aligners and annotation files can be found in:

```
$MUGQIC_INSTALL_HOME/genomes/species/<species_scientific_name>.<assembly>/
## so for Homo Sapiens hg19 assembly, that would be:
ls $MUGQIC_INSTALL_HOME/genomes/species/Homo_sapiens.hg19/
```

Now you are all set to run GenPipes analysis pipelines!

## Downloading Data from Databases

To download genomic data from an experiment, enter the Gene Expression Omnibus (GEO) accession number (you can find this in the Data Availabilty section of a paper) into GEO. 

## Pipelines

### RNA-Seq
Our lab uses the [MUGQIC RNA-Seq pipline](https://bitbucket.org/mugqic/genpipes/src/master/pipelines/rnaseq/README.md#markdown-header-usage) from GenPipes. 

RNA-Seq Stringtie Workflow:
1. Picard SAM to FastQ
2. [Trimmomatic](http://www.usadellab.org/cms/index.php?page=trimmomatic)
3. Merge Trimmomatic Stats
4. [SortMeRNA](https://github.com/sortmerna/sortmerna)
5. [STAR](https://github.com/alexdobin/STAR/releases)
6. [Picard](https://broadinstitute.github.io/picard/) Merge SAM Files
7. Picard Sort SAM
8. Picard Mark Duplicates
9. Picard RNA Metrics
10. Estimate Ribosomal RNA ([BWA]() and )
11. [RNA-SeQC2](https://github.com/getzlab/rnaseqc)
12. Wiggle
13. [Raw Counts](https://htseq.readthedocs.io/en/release_0.11.1/count.html)
14. Raw Counts Metrics
15. [Stringtie](https://ccb.jhu.edu/software/stringtie/index.shtml)
16. Stringtie Merge
17. Stringtie Abund
18. [Ballgown Gene Expression](https://bioconductor.org/packages/release/bioc/html/ballgown.html)
19. Differential Expression ([DESeq2](https://bioconductor.org/packages/devel/bioc/vignettes/DESeq2/inst/doc/DESeq2.html) and [edgeR](https://www.bioconductor.org/packages/release/bioc/html/edgeR.html))
20. [MultiQC](https://multiqc.info/)
21. CRAM Output

To run the pipeline, you will need to create two files: a readset.txt file and a design.txt file. 

A readset file is a tab-separated plain text file that contains the following information:
- Sample: must contain letters A-Z, numbers 0-9, hyphens (-) or underscores (_) only; BAM files will be merged into a file named after this value **(mandatory)**. Think of them as biological replicates.
- Readset: a unique readset name with the same allowed characters as above**(mandatory)**. Think of them as technical replicates.
- Library: optional.
- RunType: PAIRED_END or SINGLE_END **(mandatory)**.
- Run: optional.
- Lane: optional.
- Adapter1: sequence of the forward trimming adapter.
- Adapter2: sequence of the reverse trimmign adapter.
- QualityOffset: quality score offset integer used for trimming (optional).
- BED: relative or absolute path to BED file (optional).
- FASTQ1: relative or absolute path to first FASTQ file for paired-end readset or single FASTQ file for single-end readset **(mandatory if BAM value is missing)**.
- FASTQ2: relative or absolute path to second FASTQ file for paired-end readset **(mandatory if RunType value is “**`PAIRED_END`**”)**.
- BAM: relative or absolute path to BAM file which will be converted into FASTQ files if they are not available **(mandatory if FASTQ1 value is missing, ignored otherwise)**.

Example:

```
Sample Readset Library RunType Run Lane Adapter1 Adapter2 QualityOffset BED FASTQ1 FASTQ2 BAM
sampleA readset1 lib0001 PAIRED_END run100 1 AGATCGGAAGAGCACACGTCTGAACTCCAGTCA AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT 33 path/to/file.bed path/to/readset1.paired1.fastq.gz path/to/readset1.paired2.fastq.gz path/to/readset1.bam
sampleA readset2 lib0001 PAIRED_END run100 2 AGATCGGAAGAGCACACGTCTGAACTCCAGTCA AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT 33 path/to/file.bed path/to/readset2.paired1.fastq.gz path/to/readset2.paired2.fastq.gz path/to/readset2.bam
sampleB readset3 lib0002 PAIRED_END run200 5 AGATCGGAAGAGCACACGTCTGAACTCCAGTCA AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT 33 path/to/file.bed path/to/readset3.paired1.fastq.gz path/to/readset3.paired2.fastq.gz path/to/readset3.bam
sampleB readset4 lib0002 PAIRED_END run200 6 AGATCGGAAGAGCACACGTCTGAACTCCAGTCA AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT 33 path/to/file.bed path/to/readset4.paired1.fastq.gz path/to/readset4.paired2.fastq.gz path/to/readset4.bam
```

If some optional information is missing, leave its position empty. 

A design file is a tab-separated plain text file with one line per sample and the following columns:

|Sample  |Contrast_AB | Contrast_AC |
|:------:|:----------:|:-----------:|
|sampleA |1           |1            |
|sampleB |2           |0            |
|sampleC |0           |2            |
|sampleD |0           |0            |

>You can add several contrasts per design

Human

```
#!/bin/bash
rnaseq.py -c $MUGQIC_PIPELINES_HOME/pipelines/rnaseq/rnaseq.base.ini $MUGQIC_PIPELINES_HOME/pipelines/common_ini/beluga.ini $MUGQIC_INSTALL_HOME/genomes/species/Homo_sapiens.GRCh38/Homo_sapiens.GRCh38.ini -r readset.txt -d design.txt -t stringtie -s 2-21 -g rnaseqScript.txt
bash rnaseqScript.txt
```

Mouse

```
#!/bin/bash
rnaseq.py -c $MUGQIC_PIPELINES_HOME/pipelines/rnaseq/rnaseq.base.ini $MUGQIC_PIPELINES_HOME/pipelines/common_ini/beluga.ini $MUGQIC_INSTALL_HOME/genomes/species/Mus_musculus.GRCm38/Mus_musculus.GRCm38.ini -r readset.txt -d design.txt -t stringtie -s 2-21 -g rnaseqScript.txt
bash rnaseqScript.txt
```

### ChIP-Seq/ATAC-Seq

#### ChIP-Seq
Human

```
#!/bin/bash
chipseq.py -c $MUGQIC_PIPELINES_HOME/pipelines/chipseq/chipseq.base.ini $MUGQIC_PIPELINES_HOME/pipelines/common_ini/beluga.ini $MUGQIC_INSTALL_HOME/genomes/species/Homo_sapiens.GRCh38/Homo_sapiens.GRCh38.ini -r readset.txt -t chipseq -s 2-23 -g chipseqScript.txt
bash chipseqScript.txt
```

Mouse

```
#!/bin/bash
chipseq.py -c $MUGQIC_PIPELINES_HOME/pipelines/chipseq/chipseq.base.ini $MUGQIC_PIPELINES_HOME/pipelines/common_ini/beluga.ini $MUGQIC_INSTALL_HOME/genomes/species/Mus_musculus.GRCm38/Mus_musculus.GRCm38.ini -r readset.txt -t chipseq -s 2-23 -g chipseqScript.txt
bash chipseqScript.txt
```

#### ATAC-Seq
Human

```
#!/bin/bash
chipseq.py -c $MUGQIC_PIPELINES_HOME/pipelines/chipseq/chipseq.base.ini $MUGQIC_PIPELINES_HOME/pipelines/common_ini/beluga.ini $MUGQIC_INSTALL_HOME/genomes/species/Homo_sapiens.GRCh38/Homo_sapiens.GRCh38.ini -r readset.txt -t atacseq -s 2-23 -g atacseqScript.txt
bash atacseqScript.txt
```

Mouse

```
#!/bin/bash
chipseq.py -c $MUGQIC_PIPELINES_HOME/pipelines/chipseq/chipseq.base.ini $MUGQIC_PIPELINES_HOME/pipelines/common_ini/beluga.ini $MUGQIC_INSTALL_HOME/genomes/species/Mus_musculus.GRCm38/Mus_musculus.GRCm38.ini -r readset.txt -t atacseq -s 2-23 -g atacseqScript.txt
bash atacseqScript.txt
```

## Additional Resources

### Databases/Data Visualization Tools
- [cBioPortal](https://www.cbioportal.org/)
- [ChIP-Atlas](https://chip-atlas.org/)
- [Cistrome Project](http://cistrome.org/)
- [ENCODE Project](https://www.encodeproject.org/)
- [GepLiver](http://www.gepliver.org/#/home)
- [Gene Expression Omnibus (GEO)](https://www.ncbi.nlm.nih.gov/geo/)
- [GTEx](https://www.gtexportal.org/home/)
- [RCSB Protein Data Bank](https://www.rcsb.org/)
- [UCSC Genome Browser](https://genome.ucsc.edu/)

### AI Tools*
*note that
- [AlphaFold Protein Structure Database](https://alphafold.ebi.ac.uk/)
- [ChatGPT](https://chat.openai.com/)
- [Consensus](https://consensus.app/search/)
- [Elicit](https://elicit.com/)
- [Research Rabbit](https://www.researchrabbit.ai/)
- [SciSpace](https://typeset.io/)
- [Scite](https://scite.ai/assistant)
- [Semantic Scholar](https://www.semanticscholar.org/)
