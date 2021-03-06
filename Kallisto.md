


> # Kallisto

[https://pachterlab.github.io/kallisto/manual](https://pachterlab.github.io/kallisto/manual)
[https://www.kallistobus.tools/getting_started_explained.html](https://www.kallistobus.tools/getting_started_explained.html)

Author = [Lior Patcher](https://en.wikipedia.org/wiki/Lior_Pachter)

[Kallisto](https://pachterlab.github.io/kallisto/)   is a user-friendly algo which extracts ** both gene and transcript level** gene expression directly from fastq files using the raw fastq files. Results can be analysed withThen use  [Sleuth](https://pachterlab.github.io/sleuth/)  (also developed by Pachter lab) for transcript level analysis or with DESeq2 (via tximport). to perform differential gene and transcript expression analysis.

**SVD (singular value decomposition) analysis**

-   For doing this you can use the gene-level count table obtained from Kallisto. I wrote everything in R and I can send you some litterature which explains a bit the underlying math and idea. Also happy to speak about it over skype.

# Rapid Approach: Kallisto - Sleuth pipeline
Author = [Lior Patcher](https://en.wikipedia.org/wiki/Lior_Pachter)

[Kallisto](https://www.youtube.com/watch?v=94wphB3GKBM) quantifies transcript abundances. It pseudoaligns reads against a transcriptome (not genome). Simple count-based approaches underperform when determining transcript level counts as they disregard reads that overlap with more than one gene. If the genomic feature becomes a transcript rather than a gene it keeps many reads that would have been discarded.

Method of pseudoalignment: for each read the program aims to identify the target that it originates from using k-mers. By ignoring exactly where in the genome a read originates from it is much faster than normal alignment. This approach does not generate a BAM file (alignment file) but instead produce a measure of how many reads indicate the presence of each transcript. These use a **deBruikin graph** to assign reads to an isoform if they are compatible with that transcript structure.
![enter image description here](https://www.frontiersin.org/files/Articles/169488/fgene-06-00361-r2/image_m/fgene-06-00361-g002.jpg)

Schema of a simple deBruijn graph-based transcript assembly:
- Read sequences are split into all subsequence k-mers (here: of length 5) from the reads.
- A deBruijn graph is constructed using unique k-mers as the nodes and overlapping k-mers connected by edges (a k-mer shifted by one base overlaps another k-mer by k􀀀1 bases).
- The transcripts are assembled by traversing the two paths in the graph

Although much faster than alignment-counting routines it **cant detect novel isoforms**. However, instead of direct isoform quantification, you can glean more accurate answers from alternative approaches, e.g., quantification of exons (Anders et al., 2012) or estimates of alternative splicing events such as exon skipping, intron retention etc. (e.g., MISO [Katz et al., 2010](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3037023/), rMATS [Shen et al., 2014](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4280593/)).

The main limitations to assigning reads to transcripts are:
- annotation transcripts are inconsistent
- many isoforms with very different lengths
- anti-sense and overlapping transcripts of different genes

**Tools**:
Sailfish and more updated version Salmon
[Kallisto](https://www.nature.com/articles/nbt.3519)

## Kallisto Workflow
ml kallisto

Two steps:
1. Build Index (10mins to run) - only run the first time (or download from patcher lab)
2. Quantify Reads (10mins to run)

##  Kallisto index

Build index from FASTA:
/camp/home/ziffo/working/oliver/genomes/sequences/human/gencode.v29.transcripts.fa
```bash
kallisto index -i kallisto_gencode.v29.idx /camp/home/ziffo/working/oliver/genomes/sequences/human/gencode.v29.transcripts.fa
```
Kallisto index's are available at:
`/camp/home/ziffo/working/oliver/genomes/index/kallisto_gencode.v29.idx`
`/camp/home/ziffo/working/oliver/genomes/index/homo_sapiens/transcriptome.idx` # download from kallisto website - ensembl v96 [https://github.com/pachterlab/kallisto-transcriptome-indices/releases](https://github.com/pachterlab/kallisto-transcriptome-indices/releases)
`
2. Quantify Reads (10mins to run)

For an individual single or paired end fastq file you can run:
```bash
#set changable elements
##set reference transcriptome (cDNA)
REF=/home/camp/ziffo/working/oliver/genomes/sequences/human/gencode.v29.transcripts.fa
IDX=/home/camp/ziffo/working/oliver/genomes/sequences/human/gencode.v29.transcripts.cdna.fa.idx
SAMPLE=CTRL_3
R1=/home/camp/home/ziffo/working/oliver/genomes/index/kallisto_cellranger_GRCh38.3.0.0.idx` # uses the STP cellranger projects/airals/fasta q_file as the reference genome from `/camp/svc/reference/Genomics/10x/10x_transcriptomes/refdata-cellranger-GRCh38-3.0.0`
 NB this index taking >12 hours to run kallisto quant

Or can download prebuilt ensemble reference transcriptome reference from [https://github.com/pachterlab/kallisto-transcriptome-indices/releases](https://github.com/pachterlab/kallisto-transcriptome-indices/releases). This is the v96 ensembl file: `/camp/home/ziffo/working/oliver/genomes/index/homo_sapiens/transcriptome.idx`


##  Kallisto quant

```bash
ml kallisto
source activate rtest
SAMPLE=/camp/home/ziffo/working/oliver/projects/vcp_fractionation/reads/CTRL1_D0_cytoplasmic*.fastq.gz # 
INDEX=/camp/home/ziffo/working/oliver/genomes/index/kallisto_gencode.v29.idx 
OUT=/camp/home/ziffo/working/oliver/projects/vcp_fractionation/expression/kallisto
s/D7_samples/trimmed_depleted/${SAMPLE}.fq
##if you have paired-end data then set the 2nd read files for input
R2=PATH_TO_FASTQ_reverse_strand.fq

#build kallisto index
sbatch -N 1 -c 8 --mem= 40GB --wrap="kallisto quantindex -i $INDEX -o $OUT -b 100 --rf-stranded $SAMPLE"
```
Kallisto will output a folder for each quant command. Kallisto will print:
```bash
[quant] fragment length distribution is truncated gaussian with mean = 150, sd = 5
[index] k-mer length: 31
[index] number of targets: 188,753
[index] number of k-mers: 109,544,288
[index] number of equivalence classes: 760,757
[quant] running in single-end mode
[quant] will process file 1: /camp/home/ziffo/working/oliver/public/astrocyte-clarke-PNAS-2018/reads/SRR6279025.fastq
[quant] finding pseudoalignments for the reads ... done
[quant] processed 27,467,230 reads, 4,239,422 reads pseudoaligned
[   em] quantifying the abundances ... done
[   em] the Expectation-MaximizatioDX $REF"

#create directory for kallisto data 
mkdir -p kallisto
#set subdirectory for output called out
OUTDIR=/home/camp/ziffo/working/oliver/projects/airals/expression/D7_samples/kallisto
#run kalgorithm ran for 1,359 rounds
```

Run multiple at once by creating a bash script with each quant command listed. Run bash script with `bash rename.sh`
```bash
#!/bin/bash
INDEX=/camp/home/ziffo/working/oliver/genomes/index/transcriptome.idx

SAMPLE=/camp/home/ziffo/working/oliver/projects/vcp_fractionation/reads/CTRL1_D0_cytoplasmic*.fastq.gz
OUT=/camp/home/ziffo/working/oliver/projects/vcp_fractionation/expression/kallisto/merged/CTRL1_D0_cytoplasmic
sbatch -N 1 -c 8 --mem=40GB -t 12:00:00 --wrap="listo quantification with quant command. -o sets output directory. -b specifies the bootstap sample number.
##paired-end mode
kallisto quant -i $INDEDX -o $OUTDIR -b 100 --rf-stranded $SAMPLE"

SAMPLE=/camp/home/ziffo/working/oliver/projects/vcp_fractionation/reads/CTRL3_D0_cytoplasmic*.fastq.gz
OUT=/camp/home/ziffo/working/oliver/projects/vcp_fractionation/expression/kallisto/merged/CTRL3_D0_cytoplasmic
sbatch -N 1 -c 8 --mem=40GB -t 12:00:00 --wrap="kallisto quant -i $INDEX -o $OUT -b 100 --rf-$R1 $R2
##single-end mode (set fragment mean length & stranded $SAMPLE"

SAMPLE=/camp/home/ziffo/working/oliver/projects/vcp_fractionation/reads/CTRL4_D0_cytoplasmic*.fastq.gz
OUT=/camp/home/ziffo/working/oliver/projects/vcp_fractionation/expression/kallisto/merged/CTRL4_D0_cytoplasmic
sbatch -N 1 -c 8 --mem=40GB -t 12:00:00 --wrap="kallisto quant -i $INDEX -o $OUT -b 100 --rf-stranded $SAMPLE"
```
Or run as a for loop
```bash
for READ in $SAMPLE;
do
CELLLINE=`echo $READ | grep -E -o 'CTRL1|CTRL3|CTRL4|CTRL5|GLIA|GLIB|CBID|CBIE'`
DAY=`echo $READ | grep -E -o 'D[0-7]+'`
FRACTION=`echo $READ | grep -E -o 'nuclear|cytoplasmic'`
OUT=/camp/home/ziffo/working/oliver/projects/vcp_fractionation/expression/kallisto/merged/$CELLLINE_$DAY_$FRACTION

echo "Running $CELLLINE $DAY $FRACTION"
done
	for CELLLINE in $READ
done

$READ/*Aligned.sortedByCoord.out.bam
echo "Running timepoint $READ"
	for REPLICATE in $FASTQ
	do
	ID=`echo $REPLICATE | grep -E -o 'SRR[0-9]+'`
	sbatch -N 1 -c 8 --mem=40GB --wrap="ard deviation - Illumina generates fragments 180-200bp - acurately determine this from a library quantification with an instrument such as an Agilent Bioanalyzer)
kallisto quant -i $INDEDX -o $OUT $REPLICATE > $READ/${ID}"
	echo "Running sample $ID"
	done
done


# set timepoint folders
TIMEPOINT=/home/camp/ziffo/working/oliver/projects/airals/alignment/D*_samples

for SAMPLE in $TIMEPOINT;
do
BAM=$SAMPLE/*Aligned.sortedByCoord.out.bam
echo "Running timepoint $SAMPLE"
	for REPLICATE in $BAM
	do
	SRRID=`echo $REPLICATE | grep -E -o 'SRR[0-9]+'`
	sbatch -N 1 -c 8 --mem=40GB --wrap="samtools view -h $REPLICATE > $SAMPLE/${SRRID}.sam"
	echo "Running sample $SRRID"
	done
done

# quant for every sample at each timepoint & fraction (72 total)
for
	FASTQ=cytoplasmic.D0.ctrl1
	sbatch -N 1 -c 8 --mem=0 -t 12:00:00 --wrap="kallisto quant -i $INDEX -o $OUT $FASTQ"
```

### D21 Airals kallisto
Ran this kallisto quant for correlating whole cell motor neurons with single cell motor neurons.
This is single unpaired reads, which **needs the read length & SD**. Calculated transcript length from FastQC > MultiQC report - graph showing Sequence Length Distribution.
```bash
ml kallisto
cd ~/working/oliver/projects/airals/alignment/D21_samples/kallisto
INDEX=/camp/home/ziffo/working/oliver/genomes/index/homo_sapiens/transcriptome.idx
SAMPLE=~/working/oliver/projects/airals/reads/D21_samples/trimmed/*.fq.gz

for READ in $SAMPLE;
do
ID=`echo $READ | grep -E -o 'SRR[0-9]+'`
OUT=~/working/oliver/projects/airals/alignment/D21_samples/kallisto/$ID
sbatch -N 1 -c 8 --mem=0 -t 24:00:00 --wrap="kallisto quant --single -l 60 -s 3 -i $INDEX -o $OUT $READ"
echo "Running $ID"
done
```

### D7 Airals kallisto

Run multiple fastq files at once
```bash
Set index
IDX=/home/camp/ziffo/working/oliver/genomes/sequences/human/gencode.v29.transcripts.cdna.fa.idx

for SAMPLE in VCP CTRL;
do
    for REPLICATE in 1 2 3;
    do
        # Build the name of the files.
        R1=/home/camp/ziffo/working/oliver/projects/airals/fastq_files/D7_samples/trimmed_depleted/${SAMPLE}_${REPLICATE}.fq
        # The kallisto output directory.
        OUTDIR=/home/camp/ziffo/working/oliver/projects/airals/expression/D7_samples/kallisto/${SAMPLE}_${REPLICATE}
        # Run kallisto quantification in single-end mode.
        echo "*** Running kallisto on single end sample: $SAMPLE"
        kallisto quant --single -l 187 -s 70 -i $IDX -o $OUTDIR -b 100 $R1

        # Copy the abundance file to a proper name - i.e. remove the long path name so that it only contains information on the sample e.g. VCP_3.tsv
        cp $OUTDIR/abundance.tsv $OUTDIR.counts.tsv
    done
done
```


### Clarke et al 2018 PNAS public data kallisto
```bash
ml kallisto
cd ~/working/oliver/public/astrocyte-clarke-PNAS-2018/kallisto
INDEX=/camp/home/ziffo/working/oliver/genomes/index/homo_sapiens/transcriptome.idx
SAMPLE=~/working/oliver/public/astrocyte-clarke-PNAS-2018/reads/*.fastq

for READ in $SAMPLE;
do
ID=`echo $READ | grep -E -o 'SRR[0-9]+'`
OUT=~/working/oliver/public/astrocyte-clarke-PNAS-2018/kallisto/$ID
sbatch -N 1 -c 8 --mem=0 -t 24:00:00 --wrap="kallisto quant --single -l 150 -s 10 -i $INDEX -o $OUT $READ"
echo "Running $ID"
done
```
### Windrem et al 2018 public astrocyte iPSC kallisto
```bash
ml kallisto
cd ~/working/oliver/public/astrocyte-ipsc-windrem-2018/kallisto
INDEX=/camp/home/ziffo/working/oliver/genomes/index/homo_sapiens/transcriptome.idx
SAMPLE=~/working/oliver/public/astrocyte-ipsc-windrem-2018/reads/*.fastq

for READ in $SAMPLE;
do
ID=`echo $READ | grep -E -o 'SRR[0-9]+'`
OUT=~/working/oliver/public/astrocyte-ipsc-windrem-2018/kallisto/$ID
sbatch -N 1 -c 8 --mem=0 -t 24:00:00 --wrap="kallisto quant --single -l 100 -s 10 -i $INDEX -o $OUT $READ"
echo "Running $ID"
done
```

## Kallisto merge

```bash
INDEX=/camp/home/ziffo/working/oliver/genomes/index/transcriptome.idx
OUT=/camp/home/ziffo/working/oliver/projects/vcp_fractionation/expression/kallisto/merged 
IN=/camp/home/ziffo/working/oliver/projects/vcp_fractionation/expression/kallisto/ID519_A1_CTRL1_iPS-D0-Cyto*

kallisto merge -i $INDEX -o $OUT ID519_A1_CTRL1_iPS-D0-Cyto_L001 ID519_A1_CTRL1_iPS-D0-Cyto_S1_L001 ID519_A1_CTRL1_iPS-D0-Cyto_S1_L002

sbatch -N 1 -c 8 --mem=0 -t 12:00:00 --wrap="kallisto merge -o /camp/home/ziffo/working/oliver/projects/vcp_fractionation/expression/kallisto/merged 

 -i /camp/home/ziffo/working/oliver/genomes/index/gencode.v29.transcripts.cdna.fa.idx /home/camp/ziffo/working/oliver/genomes/sequences/human/gencode.v29.transcripts.fa"
```

## Kallisto Output

Output files:
- abundance.txt
- abundance.h5 (large scale format form of txt file) - used by tximport
- run_info.json
SAMPLE -b 100 --single -l 187 -s 70 $R1
```
However for multiple fastq files use a For Loop:
```bash
# Create output folder
mkdir -p kallisto

# Exit this script on any error.
set -euo pipefail

# Set Reference transcriptome (not genome).
REF=/home/camp/ziffo/working/oliver/genomes/sequences/human/gencode.v29.transcripts.fa
# Set index to build
IDX=/home/camp/ziffo/working/oliver/genomes/sequences/human/gencode.v29.transcripts.cdna.fa.idx

# Build kallisto index
sbatch -N 1 -c 8 --mem=40GB --wrap="kallisto index -i $IDX  $REF"

for SAMPLE in VCP CTRL;
do
    for REPLICATE in 1 2 3;
    do
        # Build the name of the files.
        R1=/home/camp/ziffo/working/oliver/projects/airals/fastq_files/D7_samples/trimmed_depleted/${SAMPLE}_${REPLICATE}.fq
        # The kallisto output directory.
        OUTDIR=/home/camp/ziffo/working/oliver/projects/airals/expression/D7_samples/kallisto/${SAMPLE}_${REPLICATE}
        # Run kallisto quantification in single-end mode.
        echo "*** Running kallisto on single end sample: $SAMPLE"
        kallisto quant --single -l 187 -s 70 -i $IDX -o $OUTDIR -b 100 $R1

        # Copy the abundance file to a proper name - i.e. remove the long path name so that it only contains information on the sample e.g. VCP_3.tsv
        cp $OUTDIR/abundance.tsv $OUTDIR.counts.tsv
    done
done

```

### Kallisto Output

Output files:
- abundance.txt
- abundance.h5 (large scale format form of txt file)
- run_info.json

The main output of Kallisto is the **abundance.tsv** file with columns:
```
target_id   length eff_length est_counts 	tpm
ERCC-00002  1061   891.059      18946      243099
ERCC-00003  1023   853.059      1452       19460.7
ERCC-00004  523    353.059      455        14734.5
ERCC-00009  984	   814.059      319        4480.29
```
Column 3 = eff_length = scales the transcript length by fragment length distribution . (transcript length - mean fragment length + 1)
Column 4 = est_counts = transcript abundance count
Column 5 = tpm = Transcripts Per Million


# Sleuth
https://github.com/pachterlab/sleuth
vignette('intro', package = 'sleuth')
help(package = 'sleuth')
https://pachterlab.github.io/sleuth/walkthroughs
https://pachterlab.github.io/sleuth_walkthroughs/pval_agg/analysis.html
https://groups.google.com/forum/#!forum/kallisto-sleuth-users

Sleuth runs differential analysis on kallisto counts (based on pseudo-alignment using alignment to the transcriptome index). Kallisto output includes transcript abundance estimates along with bootstraps.  Sleuth differential analysis model uses a general linear model, which accounts for biological & technical noise.

analyze the RNA-Seq dataset in order to obtain both gene-level and transcript-level differential expression results that are consistent with each other. 

Run Rstudio markdown pipeline:
`Sleuth.Rmd` located in `/Volumes/lab-luscomben/working/oliver/projects/vcp_fractionation/expression/sleuth`

The Sleuth object can be very large and needs to be run as an Rscript through CAMP cluster batch job:
```bash
ml Anaconda2
ml sleuth
source activate rtest
sbatch -N 2 -c 10 --mem 100G -t 12:00:00 --wrap="Rscript ~/working/oliver/projects/vcp_fractionation/expression/sleuth/sleuth_camp.R"
```
Can submit job as a text script:
```bash
#!/bin/bash -l
#PBS -N JOBNAME
#PBS -l walltime=5:00:00
#PBS -l nodes=2:ppn=10
#PBS -l pmem=6gb
#PBS -j oe

ml Anaconda2
ml sleuth
source activate rtest

cd ~/working/oliver/projects/vcp_fractionation/expression/sleuth

R --file=~/working/oliver/projects/vcp_fractionation/expression/sleuth/sleuth_camp.R
```

Debugging conda:
```bash
conda clean --all
conda update --all
conda update -n base -c defaults conda
conda install anaconda
conda install --channel https://conda.anaconda.org/bioconda r-sleuth
ml sleuth/0.28.0-foss-2016b-R-3.3.1
```

# Kallisto > DESeq2

Load Gene Annotaiton & Import sample details as for Sleuth

## Tximport (and Tximeta)
[https://bioconductor.org/packages/release/bioc/vignettes/tximport/inst/doc/tximport.html](https://bioconductor.org/packages/release/bioc/vignettes/tximport/inst/doc/tximport.html)

Import transcript abundance estimates for samples using the tximport. rows = gene ID, columns = sample ID. kallisto abundance.h5 files can be imported by setting type to "kallisto". 

Alternatively to .h5 files, kallisto abundance.tsv files can be imported, but this is slightly slower. Add an additional argument ignoreAfterBar=TRUE because the Gencode transcripts have names like “ENST00000456328.2|ENSG00000223972.5|…”, and our tx2gene table only includes the first “ENST” identifier. We therefore want to split the incoming quantification matrix rownames at the first bar “|”, and only use this as an identifier. 

```r
files <- file.path(kallistodir, metadata$sample, "abundance.h5")
names(files) <- metadata$sample
# import kallisto count files. NB breaks after 97 files (memory exhausted).
txi.kallisto <- tximport(files, type = "kallisto", txOut = TRUE)
# txi.kallisto <- tximport(files, type = "kallisto", tx2gene = ttg)
head(txi.kallisto$counts)

# These matrices can then be summarized afterwards using the function summarizeToGene. This then gives the identical list of matrices as using  txOut=FALSE (default) in the first tximport call.
txi.kallisto.summary <- summarizeToGene(txi.kallisto, tx2gene)
all.equal(txi.kallisto$counts, txi.kallisto.summary$counts)
```

We could alternatively generate counts from abundances, using the argument countsFromAbundance, scaled to library size, "scaledTPM", or additionally scaled using the average transcript length, averaged over samples and to library size, "lengthScaledTPM". Using either of these approaches, the counts are not correlated with length, and so the length matrix should not be provided as an offset for downstream analysis packages. tximports has a countsFromAbundance option "dtuScaledTPM". This scaling option is designed for use with txOut=TRUE for differential transcript usage analyses. See ?tximport for details on the various countsFromAbundance options.

We can avoid gene-level summarization by setting txOut=TRUE, giving the original transcript level estimates as a list of matrices.
txi.tx <- tximport(files, type = "salmon", txOut = TRUE)

## Manipulate Count Files
Concatenate all counts into 1 file that can be used for DESeq DE analysis.
```bash
# In the kallisto output directory match sample names to conditions
CTRL=(CTRL_1 CTRL_2 CTRL_3)
VCP=(VCP_1 VCP_2 VCP_3)

#set up variables to help create count files
CTRL_FILES=''
for NAME in ${CTRL[@]}; do
    CTRL_FILES+="${NAME}/abundance.tsv "
done

VCP_FILES=''
for NAME in ${VCP[@]}; do
    VCP_FILES+="${NAME}/abundance.tsv "
done

#check names are correct:
ls $CTRL_FILES $VCP_FILES

#create the counts.txt file using all samples abundance.tsv files & remove unnessary columns
paste $CTRL_FILES $VCP_FILES | cut -f 1,4,9,14,19,24,29,34,39 > counts.txt

#Alternatively:
paste /home/camp/ziffo/working/oliver/projects/airals/expression/D7_samples/kallisto/VCP_*.tsv  /home/camp/ziffo/working/oliver/projects/airals/expression/D7_samples/kallisto/CTRL_*.tsv | cut -f 1,4,9,14,19,24,29 > counts.txt
```
This produces a Counts file that can be used as an input into DESeq. This file looks like:
```
target_id          est_counts  est_counts est_counts  est_counts  est_counts  est_counts
ENST00000472225.6    6.11331    1.28737     10.0567     2.23768     2.1543      0
ENST00000496578.3    3.12574    3.97029     8.18468     0           1.6547      1.49019
ENST00000624726.1    6.73189    3.0668      1.16443     2.39545     4.31444     1.26448
```
You can change the header to include the sample names.

## SVD (singular value decomposition) analysis

-   For doing this you can use the gene-level count table obtained from Kallisto. I wrote everything in R and I can send you some literature which explains a bit the underlying math and idea. Also happy to speak about it over skype.

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEyOTIxNzg1OTYsMTQzMjA1Mjg2OCwtMz
YxOTY0MjAzLC0xOTcwOTI5MiwtMTk1ODk2NTU5MiwtOTI0MTM4
MDI5LC0xNzg4MTY1OTI3LDM5Mzk2MzkyMSwtNjcwOTgwNTkxLD
Y1MTgyOTYxOCwtMTczODE5ODQ3NywzODI3ODU5ODUsLTgzNDcx
Mzc1NSw1NjQwMDk0NTksMTA0NDk2MDAyMywxNTUwNzAwMDU0LD
U2NjczOTMyOCwtMTI2NzI4NTIyNSwxMjM3NTQwNjA5LC0xNzQ4
NzE2NzA1XX0=
-->