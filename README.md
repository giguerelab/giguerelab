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
