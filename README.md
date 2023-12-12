# nPhase
Ploidy agnostic phasing pipeline and algorithm

![Alt text](/img/nPhasePipeline.jpg?raw=true "nPhase Pipeline")

nPhase is a ploidy agnostic tool developed in python which predicts the haplotypes of a sample that was sequenced by both long and short reads by aligning them to a reference. It should work with any ploidy.

- [Quick-start](#quick-start)
- [Installation](#installation)
  * [With bioconda](#with-bioconda)
  * [Without bioconda](#without-bioconda)
- [Usage](#usage)
    + [nphase pipeline or algorithm](#nphase-pipeline-or-algorithm)
    + [nphase partial](#nphase-partial)
- [Parameters](#parameters)
    + [nPhase pipeline or algorithm](#nphase-pipeline-or-algorithm)
    + [nPhase cleaning](#nphase-cleaning)
- [Outputs](#outputs)
- [Paper](#paper)
- [Media](#media)
- [Recommendations](#recommendations)
- [Contact me](#contact-me)

# Quick-start

[nPhase on conda](https://conda.anaconda.org/oakheart/nphase/)

If you have bioconda you can install nPhase by running the following commands in your terminal:

```
conda create -n polyploidPhasing
conda activate polyploidPhasing
conda install -c oakheart nphase
conda install matplotlib=3.5.3
conda install pandas=1.5.3
```

Then you can phase your data with the following command:

```
nphase pipeline --sampleName Individual_1 --reference /path/to/Individual_referenceGenome.fasta 
                --R1 /path/to/Individual_1_shortReads_R1.fastq.gz --R2 /path/to/Individual_1_shortReads_R2.fastq.gz
                --longReads /path/to/Individual_1_longReads.fastq.gz --longReadPlatform [ont|pacbio]
                --output /path/to/outputFolder --threads 8
```

Once you've phased your data, you can use our automated cleaning method on the raw results folder (`/path/to/outputFolder/Individual_1`) with the following command:



```
nphase cleaning --sampleName Cleaning_1 --resultFolder /path/to/outputFolder/Individual_1
                --longReads /path/to/Individual_1_longReads.fastq.gz --threads 8
```

# Installation

## With bioconda

Install bioconda and set the correct channels by following the first two steps here: https://bioconda.github.io/user/install.html

Then you can create a new environment and install nPhase with the following commands:
```
conda create -n polyploidPhasing python=3.8
conda activate polyploidPhasing
conda install -c oakheart nphase
conda install matplotlib=3.5.3
conda install pandas=1.5.3
```

## Without bioconda

### Pre-requisites

You will have to install the following software before nPhase:

[Python ≥3.8](https://www.python.org/downloads/)

[bwa mem](https://github.com/lh3/bwa)

[GATK 4.0](https://github.com/broadinstitute/gatk)

[samtools v1.9](https://github.com/samtools/samtools/tree/1.9)*

[ngmlr](https://github.com/philres/ngmlr)

\*Currently, installing samtools v1.10 or higher will cause an error to occur due to [a bug in the way ngmlr handles MAPQ scores](https://github.com/philres/ngmlr/issues/83).

### Installation via PyPI

You can now install nPhase via

`pip install -U nPhase`

# Usage
### nphase pipeline or algorithm

There are two main ways to run nPhase:

`nphase pipeline` will run the entire pipeline from start to finish and requires the following inputs:

```
nphase pipeline --sampleName SAMPLE_NAME --reference REFERENCE --output OUTPUT_FOLDER --longReads LONG_READ_FILE
                --longReadPlatform {ont,pacbio} --R1 SHORT_READ_FILE_R1 --R2 SHORT_READ_FILE_R2 --threads 8
```
Optional parameters are described in [Parameters](#parameters).

`nphase algorithm` will only run the phasing algorithm, it requires inputs generated by `nphase pipeline`. This is useful if you want to test different paramaters on your dataset after having generated the pre-processed files once. Here are the inputs required by `nphase algorithm`:

```
nphase algorithm --sampleName SAMPLE_NAME --reference REFERENCE --output OUTPUT_FOLDER --longReads LONG_READ_FILE
                 --contextDepth CONTEXT_DEPTHS_FILE --processedLongReads VALIDATED_SNP_ASSIGNMENTS_FILE
```

Optional parameters are described in [Parameters](#parameters).

### nphase partial

Alternatively, if you already mapped your short reads to a reference, or your long reads, or already variant called your mapped short reads, you can try to use

`nphase partial` which will run only the parts of the pipeline that you need to run. So for example if you provide it with a vcf file, then it will only try to map the long reads. If you provide it with mapped short reads and mapped long reads, it will only variant call the short reads, etc. This is not recommended since I can't control what you input but it can save people time or allow for more creative uses of nPhase.

Here are the use cases of `nphase partial`:

You have mapped long reads and a vcf file of your short reads:
```
nphase partial --sampleName SAMPLE_NAME --reference REFERENCE --output OUTPUT_FOLDER --longReads LONG_READ_FILE
               --vcf VCF_FILE --mappedLongReads MAPPED_LONG_READ_FILE --threads 8
```
You have mapped long reads and mapped short reads, but not vcf:
```
nphase partial --sampleName SAMPLE_NAME --reference REFERENCE --output OUTPUT_FOLDER --longReads LONG_READ_FILE
               --mappedLongReads MAPPED_LONG_READ_FILE --mappedShortReads MAPPED_SHORT_READ_FILE --threads 8
```
You have mapped long reads, but no mapped short reads and no vcf:
```
nphase partial --sampleName SAMPLE_NAME --reference REFERENCE --output OUTPUT_FOLDER --longReads LONG_READ_FILE
               --mappedLongReads MAPPED_LONG_READ_FILE --R1 SHORT_READ_FILE_R1 --R2 SHORT_READ_FILE_R2 --threads 8
```
You have a short read vcf, but no mapped long reads:
```
nphase partial --sampleName SAMPLE_NAME --reference REFERENCE --output OUTPUT_FOLDER --longReads LONG_READ_FILE
               --vcf VCF_FILE --longReadPlatform {ont,pacbio} --threads 8
```
You have mapped short reads, but no mapped long reads and no vcf:
```
nphase partial --sampleName SAMPLE_NAME --reference REFERENCE --output OUTPUT_FOLDER --longReads LONG_READ_FILE
               --mappedShortReads MAPPED_SHORT_READ_FILE --longReadPlatform {ont,pacbio} --threads 8
```

# Parameters
### nPhase pipeline or algorithm

```
nphase pipeline [-h] [--version] [--threads [THREADS]] [--maxID [MAXID]] [--minOvl [MINOVL]] [--minSim [MINSIM]] [--minLen [MINLEN]] [--nPairs [NPAIRS]] --sampleName SAMPLE_NAME
                       --reference REFERENCE --output OUTPUT_FOLDER --longReads LONG_READ_FILE --longReadPlatform {ont,pacbio} --R1 SHORT_READ_FILE_R1 --R2
                       SHORT_READ_FILE_R2
or
nphase algorithm [-h] [--threads [THREADS]] [--maxID [MAXID]] [--minOvl [MINOVL]] [--minSim [MINSIM]] [--minLen [MINLEN]] [--nPairs [NPAIRS]] --sampleName SAMPLE_NAME
                        --reference REFERENCE --output OUTPUT_FOLDER --longReads LONG_READ_FILE --contextDepth CONTEXT_DEPTHS_FILE --processedLongReads
                        VALIDATED_SNP_ASSIGNMENTS_FILE

positional arguments:
    pipeline            Run the entire nPhase pipeline on your sample
    algorithm           Only run the nPhase algorithm. NOTE: This will require files generated by running the pipeline mode

arguments always required:
  --sampleName STRAINNAME
                        Name of your sample, ex: "Individual_1"
  --reference REFERENCE
                        Path to fasta file of reference genome to align to, ex: /home/reference/Individual_reference.fasta
  --output OUTPUTFOLDER
                        Path to output folder, ex: /home/phased/
  --longReads LONGREADFILE
                        Path to long read FastQ file, ex: /home/longReads/Individual_1.fastq.gz

additional arguments required by nphase pipeline:
  --longReadPlatform {ont,pacbio}
                        Long read platform, must be 'ont' or 'pacbio'
  --R1 SHORTREADFILE_R1
                        Path to paired end short read FastQ file #1, ex: /home/shortReads/Individual_1_R1.fastq.gz
  --R2 SHORTREADFILE_R2
                        Path to paired end short read FastQ file #2, ex: /home/shortReads/Individual_1_R2.fastq.gz
  --nPairs [NPAIRS]
                        Number of overlapping clusters to merge per iterative step in the nPhase algorithm. Higher values can speed up runtime at the cost of accuracy. Default 1

additional arguments required by nphase algorithm:
  --contextDepth CONTEXTDEPTHSFILE
                        Path to context depths file, ex: /home/phased/Individual_1/Overlaps/Individual_1.contextDepths.tsv
  --processedLongReads VALIDATEDSNPASSIGNMENTSFILE
                        Path to validated long read SNPs, ex:
                        /home/phased/Individual_1/VariantCalls/longReads/Individual_1.hetPositions.SNPxLongReads.validated.tsv
  --nPairs [NPAIRS]
                        Number of overlapping clusters to merge per iterative step in the nPhase algorithm. Higher values can speed up runtime at the cost of accuracy. Default 1

additional arguments potentially required by nphase partial:
  --mappedShortReads MAPPEDSHORTREADS
                        Path to mapped, sorted short read file in BAM format, ex: /home/phased/Individual_1/Mapped/shortReads/Individual_1.final.bam
  --vcf VCFFILE         Path to VCF file (must be generated with GATK --ploidy=2), ex:/home/phased/Individual_1/VariantCalls/shortReads/Individual.vcf
  --mappedLongReads MAPPEDLONGREADS
                        Path to mapped long read file in SAM format and should be filtered to keep split reads (flag 260), ex: /home/phased/Individual_1/Mapped/longReads/Individual_1.sorted.sam
  --longReadPlatform {ont,pacbio}
                        Long read platform, must be 'ont' or 'pacbio'
  --R1 SHORTREADFILE_R1
                        Path to paired end short read FastQ file #1, ex: /home/shortReads/Individual_1_R1.fastq.gz
  --R2 SHORTREADFILE_R2
                        Path to paired end short read FastQ file #2, ex: /home/shortReads/Individual_1_R2.fastq.gz
  --nPairs [NPAIRS]
                        Number of overlapping clusters to merge per iterative step in the nPhase algorithm. Higher values can speed up runtime at the cost of accuracy. Default 1

optional arguments:
  -h, --help            show this help message and exit
  --version             show program's version number and exit
  --threads [THREADS]   Number of threads to use on some steps, default 8
  --maxID [MAXID]       MaxID parameter, determines how different two clusters must be to prevent them from merging. Default 0.05
  --minOvl [MINOVL]     minOvl parameter, determines the minimal percentage of overlap required to allow a merge between two clusters that have fewer than
                        100 heterozygous SNPs in common. Default 0.1
  --minSim [MINSIM]     minSim parameter, determines the minimal percentage of similarity required to allow a merge between two clusters. Default 0.01
  --minLen [MINLEN]     minLen parameter, any cluster based on fewer than N reads will not be output. Default 0

```

### nPhase cleaning

```
nPhase cleaning [-h] [--version] --sampleName STRAINNAME --longReads LONGREADFILE --resultFolder PHASINGRESULT [--filterPct [PERCENTKEPT]]
                       [--fillGaps [DEDUPLICATE]] [--filterFirst [FFBOOL]] [--setMaxDiscordance [MAXDISCORDANCE]

optional arguments:
  -h, --help            show this help message and exit
  --version             show program's version number and exit

required arguments:
  --sampleName STRAINNAME
                        Name of your sample, ex: "Individual_1"
  --longReads LONGREADFILE
                        Path to long read FastQ file, ex: /home/longReads/Individual_1.fastq.gz
  --resultFolder PHASINGRESULT
                        Path to output folder, ex: /home/nphaseResults/Individual_1/
  --filterPct [PERCENTKEPT]
                        Percentage of results to filter out. Default 2.0
  --fillGaps [DEDUPLICATE]
                        Attempt to fill gaps by redistributing reads from most covered cluster. Set to 0 to disable this step. Default 1
  --filterFirst [FFBOOL]
                        Perform the filtering step before the merging step. Set to 1 to enable. Default 0
  --setMaxDiscordance [MAXDISCORDANCE]
                        If this is set higher than 0, use this percentage as the stopping rule for merging instead of using the mean discordance of raw
                        results. Inputting 6.8 means 6.8% discordance max allowed. Default 0
```

# Outputs

All files will be named with the prefix "SAMPLE_NAME_minOvl_minSim_maxID", which we will shorten to $prefix in this section.

* One fastQ file per predicted haplotig in OUTPUT_FOLDER/Phased/FastQ/

* Three plots in OUTPUT_FOLDER/Phased/Plots/:
  * **$prefix_coverageVis.{png|svg|pdf}**: Coverage of each haplotig plotted against your reference genome
  * **$prefix_phasedVis.{png|svg|pdf}**: Displays the haplotigs along the genome
  * **$prefix_discordanceVis.{png|svg|pdf}**: Violin plot for each haplotig showing the distribution of allele frequency within the haplotig. Values around 0.5 here are a sign that two distinct haplotypes were erroneously merged into one. **Note: These are currently generated by plotnine and suffer from memory issues. For now nPhase will only try to generage the svg plot, and may error out, but the raw data is still available. More info: https://github.com/has2k1/plotnine/issues/430**

* Six tab-separated value (.tsv) files in OUTPUT_FOLDER/Phased/:
    * **$prefix_variants.tsv**: For each haplotig, the haplotig name, chromosome, position, base
    * **$prefix_clusterReadNames.tsv**: For each haplotig, the names of reads that comprise it
    * **$prefix_phasedDataFull.tsv**: For each haplotig, the position, chromosome, y position of each phased SNP. Raw data, not used to generate the phasedVis plot.
    * **$prefix_discordanceVis.tsv**: For each haplotig, the haplotig name, chromosome, position, base, frequency, coverage. Used to generate the discordanceVis plot.
    * **$prefixD_phasedDataSimple.tsv**: For each haplotig, the start position, stop position, chromosome, y position. Used to generate the phasedVis plot.
    * **$prefix_covVis.tsv**: For each haplotig, the haplotig name, chromosome, 5kb window, mean coverage for the window

# Paper

If you use nPhase in your work or if you're interested in better understanding how nPhase works, please cite/read the following publication: [nPhase: An accurate and contiguous phasing method for polyploids](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-021-02342-x/metrics)

# Media

Online lightning talk I gave about nPhase for an Oxford Nanopore event [5:44]: [link](https://drive.google.com/file/d/16quLufiNhICXqmAAGRYEIy1MivWVMXVI/view?usp=sharing)

# Recommendations

Current recommendations for default parameters are at least 20X coverage per haplotype (so 3\*20=60X for a triploid) and a heterozygosity level of at least 0.4% (average of 1 heterozygous SNP every 250 bp).

It is currently untested on pacbio data so if you have a pacbio dataset (with a known ground truth) please contact me (raise an issue on github or email me), especially if you have errors.

There is an example dataset on this github, in the example folder, which you can test nPhase on. It contains a reference sequence, short reads and long reads. The result should look like a triploid and it should run quickly.

For significantly more exhaustive testing, please check out the [Phasing Toolkit repo](https://github.com/OmarOakheart/Phasing-Toolkit). You can use it to automatically perform benchmarks, compare nPhase to other polyploid phasing tools, calculate accuracy metrics on tests with ground truth datasets generated in the same way as described in the nPhase paper.

# Contact me

email: omaroakheart@gmail.com
