


> # Differential Splicing Analysis
[Video RNA seqblog](https://www.rna-seqblog.com/differential-splicing-analysis-with-rna-seq-current-applications-approaches-limitations/)
[Coursera tools video](https://www.coursera.org/lecture/genomic-tools/tools-for-transcriptomics-2-rna-seq-ESKKW)
![enter image description here](https://lh3.googleusercontent.com/mAVrLXkGSEAroRb6hIXjSFkHJhrHrXo_T0wKWqVrTXofmVU1I_NTXU70mhs3LG6OuvMPcdzSLz9JyQ)


![enter image description here](https://lh3.googleusercontent.com/BGrCH8FUIUWYDSY2U1b_kBzc0AtCXguL7EQdOR3ozQ1wCLdZPvjk58ImA96UpRVRJWUX4-b5IjuyhA)

Cassette exon = Exon skipping
Alternative splice sites incorporate different regions of an exon in the mRNA isoform
Mutally exclusive exons = include one or the other exon
Concatenate = link exons into a chain to form mRNA

Alternative splicing is used normally to enable a gene to produce different proteins which maybe tissue specific. However aberrant splicing can cuase disease by inappropriate exon skipping (e.g. BRCA1 exon 18 in cancer) and intron retention.
![enter image description here](https://lh3.googleusercontent.com/jeQ92hUjro9rmqVMyY53dnH1cGWIRAN466rpSTAceqTWdWFm8xN0aO3DgUfebTQZsvWNWg8UhYEgXA)

In Differential Gene Expression you compare mutant vs control read counts for each gene. 
In **Differential Splicing Expression** you compare all the different mRNA isoforms between mutant and control - there are many isoforms for each gene so there are many comparisons.
![enter image description here](https://lh3.googleusercontent.com/VyUh9L2RvsCzGjGbh1fV5pPq2gGCU4gadhSaCk7BvpvceELVaUiMngWYwXIwFhbvU1xTQmAR0f4DFg)

In **Differential Isoform Usage** you compare the usage of all the alternatively spliced transcripts **RELATIVE** to the total gene expression between mutant & control. I.e. take the Isoform expression counts divide by that gene expression counts. This is the important calculation with Diffential Splicing Analysis.
![enter image description here](https://lh3.googleusercontent.com/yc1EU-oIRVA1EuDyUkmW0rQewMRlBPWaOtgLIkpdAxMqaMr74KSbzfc4kekQKSEpV62ZaWv0WX1ElA)
![enter image description here](https://lh3.googleusercontent.com/vCWCaScTZWxndCmsKzKNrrQULS51WUsJNo-e64d5YP-s1m6kfWd_0pyNPBsItwIlo-Rhaz1O7N5vpw)



# Approaches to Differential Splicing Analysis
**1. Isoform Resolution Approach**
Differences in complete isoform proportions (expression) between samples. Does NOT rely on transcriptome annotation as with the other approaches.
Method: Assemble all full length isoforms > Quantify expression of each isoform > Test differeneces in relative abundance
Advantages: investigate full length isoforms. Doesnt rely on transcriptome annotation > better for identifying novel events.
Limitations: complex, ambiguity with different reads aligning to same position.
Tools: **Cuffdiff**, DiffSplice, SplicingCompass

**2. Exon Usage Approach**
Differential exon-level expression between conditions. Compares individual exon expression count.
Method: Avoids complexity of transcript assembly & expression estimation. Assumes differential usage of non-terminal exons equates to differential splicing. **Requires transcriptome annotation** to identify all possible exon units for each gene. Genes models are flattened into exon counting bins (tourquoise in image below). Overalpping exons and exons with alternative boundaries are split into separate bins (dotted lines in image). Then quantifies individual exon bins. Then tests differential usage of each exon bin using generalised linear model (same as with differential gene expression)
Tools: **DEXSeq**, DSGSeq, GPSeq, SOLAS
Advantages: easier as doesnt resolve full length isoforms, doesnt make an abundance estimation at the transcript level
![enter image description here](https://lh3.googleusercontent.com/00opX631NuAn6nJrNQatEU2G9n6Hk-e0UxaMGqVwGV6vJUI3VHrUEaQ3CPcnd1DpIqpqEFpoVV8uaA)
![enter image description here](https://lh3.googleusercontent.com/yLcLWxmo7DlPLJyzSeLxqlae97F9a69sXdXJeDOxf2ct-_e7wj9iiNcWAxF6hMC4UQccwrSbR-r7gQ)

**3. Splicing Event Approach**
Differential inclusion of alternative splicing events between samples. Focus on localised alternative splicing patterns. Builds on exon-based approach by incorporating splice site information. Compresses  alternative transcripts into a single unit:
![enter image description here](https://lh3.googleusercontent.com/mxwnemNNIlCnwrJdpNKroGcigW-yZHFv0C8jIhXaSdxHhoX7eCElMdybH2mi7DDZySNQzmIx_fir0Q)

Method: Differential splicing is assumed from **differential inclusion** of a particular splicing event (5 simple AS events):
![enter image description here](https://lh3.googleusercontent.com/Yp1gfeFb5Vv45TQHCfigT4J29TwTpT4NqcS9QUNAPB2Zp9XvK-uE1ZLqpSv-m8znWoYIilXvpKu6RA)
Identifies the different alternative splicing patterns from transcriptome annotation. Exons from all transcripts with the same flanking exons and binary event exon are collapsed into single units (torquoise below).

Splicing event is quantified as **percent spliced in (PSI)**. 
PSI = Alternative Splicing Event / Normal Splicing Event. 0 < PSI < 1. Image below PSI 0.76 = 76% of the genes expression is coming from the exon inclusion event.
**PSI index = (a + b)/(a + b + 2c)**, where a and b = the number of splice-junction reads connecting the alternative exon to the upstream and downstream constitutive exons, respectively, and c = the number of junction reads connecting the two constitutive exons.
Calculates change in inclusion (delta PSI) for all potential splicing events between conditions. Tests if delta PSI exceeds a set threshold e.g. 5% (likelihood-ratio test). Visualise results in a sashimi plot.
**Advantages**: doesnt make assumptions that is required with full isoform resolution, 
**Limitations**: only tests the 5 simple classical splicing patterns (skipped exons, retained introns, alt 5' splice site, alt 3' splice site, mut excl splice sites )& **misses complex patterns not classified** (compound events eg Alt 5'ss + ES + Alt 3'ss, multiple skipped exons, non binary splicing - account for ~20% of AS events); splicing events are **strictly binary** (inclusion/exclusion of event):
![enter image description here](https://lh3.googleusercontent.com/IdaivGvJ6g2Y4oNTLGwnrY4pqhbv_wemYzT_sYYX71_fWnTn9jNp1bLZcLKM6WSuK0RVzaGiYDhR4Q)

![enter image description here](https://lh3.googleusercontent.com/D52SoO2TmOgf9tzlCamVAVRvDkZGGqoBAO1zJbcz-bNg6zntuJ2T8VUrUs7t6NtPDc4grPlkyKihZw)
Tools: **VAST-TOOLS**, rMATS, JuncBASE, JETTA, SpliceSeq

![enter image description here](https://lh3.googleusercontent.com/k05ELwI_1WON11e6TULfOIQ8Mc9w-248qVu2LkCJ148MrOcK3wA9EsVOqJcTWus24yGIVNV3zRugVg)

![enter image description here](https://lh3.googleusercontent.com/V1TmFNGiyJK7TehC_I6G0n_G9l71OA282Q1RXPD1qEJCLUC1xB8cFQWB1jIEJEpI_1sPk5uolLAGmA)

![enter image description here](https://lh3.googleusercontent.com/Y8GI4VPHpZ6V6Rp99jtcERChY83IWskw6PC-4Fc4NuIUOUlKbJJ-VLrky9bhcuEWWn7VNq31jloC7A)

**4. Alternative Splicing Graphs Approach**

Method: build splicing graph from transcriptome annotations, representing all possible AS variants. Then identify & quantify AS events.
Advantages: utilising read alignment information, can identify novel splicing events (not possible with DEU & AS event approaches). Can annotate & quantify complex splicing patterns.
Tools: ASGAL, Whippet, SGSeq
![enter image description here](https://lh3.googleusercontent.com/vLTaa0NevRDtwSSIIHXQusqYmSolMCtMYJPLFzc53zC3SfoKh_mnNwSs61W0iAla6trRXRFo5ifH2A)

# Limitations of Differential Splicing Analysis

## 1. Overlapping Transcriptome Features
Transcriptome features found at same gene location: completely overlap, partially overlap, shared boundaries. Introduces complexity & ambiguity for splicing analysis. 
DNA is double stranded: the 2 strands can produce different isoforms. But even on same strand you can produce different isoforms. Can work out which reads come from which strand:
![enter image description here](https://lh3.googleusercontent.com/sO5sJcsyqVa7u2FfuTooa5YOUbIM7X2Vali9zWR45BBUj0m3CIh4eOFCPLvL3dMSs0eFeaj7vWN5QA)

Deal with opposite strand issue with strand-specific library prep:

Isoform resolution approach deals with same strand isoforms calculate the **maximum likelihood estimation** to determine probability of a read belonging to a particular isoform. **Overlapping genes are a major problem**: they increase the number of transcripts & potential for overlapping exons.

Exon based approach deals with this by **aggregating overlapping genes** into a single group and testing differential exon usage within the group. Then removals all overlapping exons and tests remaining structures. This looses exons. For overlapping genes it has ambuguity at gene level for differential splicing - mistake differential gene expression for differential exon usage.

Splicing event approach deals with this by identifying splicing events based on transcriptome annotation. Then annotates events separately for each gene. Then tests each event of each gene independently (doesnt group genes together). The disadvantage of this is that ambiguious reads aligning to more than 1 gene will be counted towards the expression of both isoforms.

![enter image description here](https://lh3.googleusercontent.com/iHZ1u2hZF4WkZZW_D4AOviQO9rtSTJ_uemJ9fYQ_W3ibJ8xpdYOv0np1GDvcY7XNaONrc1ELabmhCA)

## 2. Complex Splicing Events

Neglected by the differential exon usage & alternative splicing event approaches.

![enter image description here](https://lh3.googleusercontent.com/IdaivGvJ6g2Y4oNTLGwnrY4pqhbv_wemYzT_sYYX71_fWnTn9jNp1bLZcLKM6WSuK0RVzaGiYDhR4Q)


# Tools
[http://www.rna-seqblog.com/tag/alternative-splicing/](http://www.rna-seqblog.com/tag/alternative-splicing/)

**1. Isoform Resolution Approach**

**2. Exon Usage Approach**
- [DEXSeq](https://bioconductor.org/packages/release/bioc/html/DEXSeq.html): focused on differential exon usage. [Vignette](http://127.0.0.1:12657/library/DEXSeq/doc/DEXSeq.pdf).
- JunctionSeq is like DEXSeq with junction reads included (and is written by the QoRTs team). JunctionSeq vignette - they have a great walkthrough that ... walks you through the whole process from beginning to end inc.

**3. Splicing Event Approach**
- [VAST-TOOLS](https://github.com/vastgroup/vast-tools): Ben Blancoe's lab. Used by Raphaelle. Looks at intron retention & 2 junction reads within each exon to look at Microexons.
- [Matt](https://academic.oup.com/bioinformatics/article/35/1/130/5053311): UNIX command line tool. Downstream analysis of VAST-Tools PSI output table to provide exon comparisons; motif RNA maps; [http://matt.crg.eu/](http://matt.crg.eu/)
- [PSI Sigma](https://github.com/wososa/PSI-Sigma): A new [PSI index](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5848607/). Traditionally, the PSI index is denoted as (a + b)/(a + b + 2c), where a and b = the number of splice-junction reads connecting the alternative exon to the upstream and downstream constitutive exons, respectively, and c = the number of junction reads connecting the two constitutive exons. We modified the PSI index as follows: 
![enter image description here](https://lh3.googleusercontent.com/7AkyPb4-l3ftfBmhfl0gqHpvCB85STqY3iAAiFf7PGtVUD1x4KrcIYKC_NJNGQEVAoIka-aLPKVq6A)
C1 and C2 = the upstream and downstream constitutive exons, respectively.  C1Si =the total number of junction reads whose 5′ splice site is connected to the upstream constitutive exon in a given splicing event. C2Sj = the junction reads whose 3′ splice site is connected to the downstream constitutive exon.
- [SpliceDetector](https://www.nature.com/articles/s41598-018-23245-1): SpliceGraph forms based on freq. of active splice sites in pre-mRNA. Then, compares transcript exons to SpliceGraph exons. Discovers AS events from known transcripts. Simple & Fast. Transcript ID > build SpliceGraph using Exon coordinates > identify AS events

**4. Alternative Splicing Graphs Approach**
- [ASGAL](https://asgal.algolab.eu/): predicts events that use splice sites which are novel with respect to a splicing graph.  Directly align reads to a splicing graph.
- [SGSeq](https://bioconductor.org/packages/release/bioc/html/SGSeq.html) - Jack Humphrey uses this. Rstudio based. Input = BAM. Novel events found from splice graph. Author = Leonard Goldstein from Genentech. See [vignette]([https://bioconductor.org/packages/release/bioc/vignettes/SGSeq/inst/doc/SGSeq.html](https://bioconductor.org/packages/release/bioc/vignettes/SGSeq/inst/doc/SGSeq.html))
.
- [Portcullis](https://github.com/TGAC/portcullis): removes invalid splice junctions from pre-aligned RNA seq data. Splice aware aligners often produce many false positive splice junctions. Filters culls splice sites which are unlikely to be genuine. 
 - QoRTs
- [rMATS](http://rnaseq-mats.sourceforge.net/): useful for comparing with other ENCODE datasets. only lists novel events that use annotated splice sites
- [MISO](https://github.com/yarden/MISO): Bayesian inference to estimate the probability for a read to be issued from a particular isoform. supplies confidence intervals (CIs) for: (i) estimating of exon and isoform abundance, (ii) identifying differential expression. It can be applied for analyzing isoform regulation.
- [MAJIQ] is also good but parsing the output is a bit annoying (but the default was the best looking one of the lot!). quantifies the relative abundances of a set of Local Splicing Variations which implicitly represent combinations of AS events involving both annotated and novel splice sites
- Whippet is new and lightweight, but you can't really see what it is up to or the reads it has aligned
- [LeafCutter](https://www.nature.com/articles/s41588-017-0004-9) https://github.com/davidaknowles/leafcutter quantifies differential intron usage across samples, allowing the detection of novel introns which model complex splicing events. requires as input the spliced alignments. 
- [IsoformSwitchAnalyzeR](https://bioconductor.org/packages/release/bioc/vignettes/IsoformSwitchAnalyzeR/inst/doc/IsoformSwitchAnalyzeR.html#overview-of-alternative-splicing-workflow)
- [IR Finder](https://github.com/williamritchie/IRFinder) from Massachusetts General, utilises [IRBase](http://mimirna.centenary.org.au/irfinder/database/) - a database of >2000 public human RNAseq samples. Uses STAR > IR detection > IR quantification > compare samples: [User Manual](https://github.com/williamritchie/IRFinder/wiki) & [paper](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-017-1184-4)
- SUPPA2: only able to detect AS events that are in the annotation. requires the quantification of the input transcripts, which can be obtained by using Salmon
- [DARTS](https://github.com/Xinglab/DARTS) & [paper](https://www.nature.com/articles/s41592-019-0351-9) uses deep learning to analyse alternative splicing
- [Cuffdiff](http://cufflinks.cbcb.umd.edu/manual.html#cuffdiff)
- [ALEXA-seq](http://www.alexaplatform.org/alexa_seq/)
- [SplicingCompass](http://www.ichip.de/software/SplicingCompass.html)
- [Flux Capacitor](http://flux.sammeth.net/capacitor.html), [JuncBASE](http://compbio.berkeley.edu/proj/juncbase/Home.html), 
- [SpliceR](http://www.bioconductor.org/packages/2.13/bioc/html/spliceR.html)
- [FineSplice](http://nar.oxfordjournals.org/content/early/2014/02/25/nar.gku166.full)
- [ARH-seq](http://nar.oxfordjournals.org/content/early/2014/06/11/nar.gku495.full)
IUTA [[25283306](http://www.ncbi.nlm.nih.gov/pubmed/25283306)]
PennSeq [[24362841](http://www.ncbi.nlm.nih.gov/pubmed/24362841)]
FlipFlop [[24813214](http://www.ncbi.nlm.nih.gov/pubmed/24813214)]
SNPlice [[25481010](http://www.ncbi.nlm.nih.gov/pubmed/25481010)]
GESS [[24447644](http://www.ncbi.nlm.nih.gov/pubmed/24447644)]
DiffSplice [[23155066](http://www.ncbi.nlm.nih.gov/pubmed/23155066)]
SigFuge [[25030904](http://www.ncbi.nlm.nih.gov/pubmed/25030904)]
CLASS [bioRXiv]
SplAdder [bioRXiv]
SplicePie [25800735](http://www.ncbi.nlm.nih.gov/pubmed/25800735)


## Analysis approach
https://www.nature.com/articles/nmeth.1503.pdf

1. Infer structural infromation about the transcript 
2. Infer the **DNA strand** (+ or -) by examining splice site spanning reads

Each transcript isoform has very few exons & **exon-exon splice junctions** that are unique to that isoform.
Ignore structure of full length transcript & focus on individual sequence features.

![enter image description here](https://journals.plos.org/ploscompbiol/article/figure/image?size=large&id=info:doi/10.1371/journal.pcbi.1004393.g006)
yellow gene = noncoding RNA gene.
brown & green genes = coding genes
Few exon-exon spanning genes.

# Gene Isoform counting

Gene isoforms are mRNA produced from the same locus but with different protein coding sequences.  5 modes of alternative splicing are recognised:
1.  Exon skipping or cassette exon.
2.  Mutually exclusive exons.
3.  Alternative donor (5') site.
4.  Alternative acceptor (3') site.
5.  Intron retention.

# VAST-TOOLS

## Overview

VAST-tools output = any splicing changes OVER TIME. i.e not just retained introns
Focus is how splicing patterns changed over time (Day 0; Day 7; Day 14 & Day 21) in the CTRL & VCP groups. Rather than directly compare splicing differences at each stage between CTRL & VCP, I compared the groups of genes at each stage which were exhibiting changes in splicing over time in control group versus VCP group. 
Most changes in IR in CTRL occurred at day 14 whilst in VCP the same events were occurring at day 7, premature IR/splicing in VCP mutants.

The `Splicing_VASTOOLS.sh` script is located in `/home/camp/ziffo/working/oliver/scripts/intron_retention`

There are 6 steps:
1. Align (this is redone in VAST-TOOLS using bowtie - needs loading)
2. Combine outputs into 1 summary table
3. Merge technical repeats (optional)
4. Tidy (optional)
5. Differential Splicing analysis
6. Plot the output

## Alignment
ml R
ml Bowtie

https://github.com/vastgroup/vast-tools#alignment

Use untrimmed fastq files - use raw reads. Define reference genome species (Hsa = human). 
```bash
cd /home/camp/ziffo/working/oliver/projects/airals/splicing/raphaelle_vast_tools
# Not necessary to create output folder as VAST-tools auto creates an output folder within your current working directory named "vast_out" and within that is "tmp" and "to_combine" folders

#set shortcuts
FASTQ=/home/camp/ziffo/working/oliver/projects/airals/reads/D0_samples/SRR*_1.fastq
FASTQ=/home/camp/ziffo/working/oliver/projects/airals/reads/D7_samples/SRR*_1.fastq
FASTQ=/home/camp/ziffo/working/oliver/projects/airals/reads/D14_samples/SRR*_1.fastq
FASTQ=/home/camp/ziffo/working/oliver/projects/airals/reads/D21_samples/SRR*_1.fastq
FASTQ=/home/camp/ziffo/working/oliver/projects/airals/reads/D35_samples/SRR*_1.fastq
FASTQ=/home/camp/ziffo/working/oliver/projects/airals/reads/D112_samples/SRR*_1.fastq

#run vast-tools on each FASTQ file separately at each time point. Dont specify output as all files need to be in same subfolder > output auto goes into a folder called vast_out. Run from the vast-tools directory. Takes ~ 40mins per fastq.
for SAMPLE in $FASTQ
do
	sbatch -N 1 -c 8 --mem=40GB --wrap="vast-tools align $SAMPLE -sp Hsa"
done
```

## Merging Outputs
https://github.com/vastgroup/vast-tools#merging-outputs

Merge the aligned output files for technical replicates when read coverage for independent replicates is not deep enough for a complete AS analysis.  Ideally have >150 million reads per sample for VAST-TOOLS AS analysis.  Raphaelle merged the 3 samples for each of VCP & CTRL at each time point.
If no technical replicates then skip this.

```bash
cd /home/camp/ziffo/working/oliver/projects/airals/splicing/raphaelle_vast_tools
#Prepare config_file from sample group txt file (needs 2 columns: fastq file name & group separated by a tab)
awk '{print $3"\t"$2}' /home/camp/ziffo/working/oliver/scripts/splicing/VASTOOLS_merge_groups.txt | tail -31 > /home/camp/ziffo/working/oliver/projects/luisier-nature-comms-2018-sfpq-ir/splicing/raphaelle_vast_tools/config_file

CONFILE=/home/camp/ziffo/working/oliver/projects/airals/splicing/D7vsD0_VCP_vast_tools/config_file
OUT=/home/camp/ziffo/working/oliver/projects/airals/splicing/D7vsD0_VCP_vast_tools/vast_out

#Run it -- takes about 10 minutes on the 31 samples
sbatch -N 1 -c 8 --mem=40GB --wrap="vast-tools merge --groups ${CONFILE} --o $OUT --sp Hsa"
```
The output files from `merge` are in `to_combine/` and named according to their group name from the `CONFILE`. Now need to remove the individual un-merged files from `to_combine/`:
```bash
cd  ~/working/oliver/projects/airals/splicing/D7vsD0_VCP_vast_tools/vast_out/to_combine
rm SRR*
```

## Combining results
ml R
https://github.com/vastgroup/vast-tools#combining-results

About 2G of memory required; completed in about 10 min
Combine aligned files that are stored in the folders `to_combine`  to form one final table called `INCLUSION_LEVELS_FULL-Hsa6-hg19.tabz` (if using old legacy version) or 5 tables in v.2.0.0. This is the table that you send to differential splicing command (and to Matt). **Option to specify hg38 (default hg19)**. The output directory contains the sub-folders to combine. 
```bash
#move to to_combine directory with merged aligned files
cd /home/camp/ziffo/working/oliver/projects/airals/splicing/D7vsD0_VCP_vast_tools/vast_out/to_combine
#set aligned output file
OUT=/home/camp/ziffo/working/oliver/projects/airals/splicing/D7vsD0_VCP_vast_tools/vast_out

# run combine command
sbatch -N 1 -c 8 --mem=40GB --wrap="vast-tools combine -o $OUT -sp Hsa -v"

# This produces 5 INCLUSION_TABLE files in the raw_incl folder. 
##ANNOT = identifies & profiles annotated exons. This is the novel feature of v.2.0.0
##COMBI = splice site based pipeline
##EXSK = only alternative Exons are considered. Transcript based pipeline, single.
##MIC = microexon pipeline
##MULTI = transcript based pipeline, multiexon

#check output
## old legacy table
cd vast_out
head INCLUSION_LEVELS_FULL-Hsa6-hg19.tab
## new v2.0.0
cd vast_out/raw_incl
head INCLUSION_LEVELS_ANNOT-Hsa14-n.tab
head INCLUSION_LEVELS_COMBI-Hsa14-n.tab
head INCLUSION_LEVELS_EXSK-Hsa14-n.tab
head INCLUSION_LEVELS_MIC-Hsa14-n.tab
head INCLUSION_LEVELS_MULTI-Hsa14-n.tab
```
Format of the combine output is as follows: 
https://github.com/vastgroup/vast-tools/blob/master/README.md#combine-output-format

Can skip remainder of VAST tools and perform downstream analysis with Matt using INCLUSION_LEVELS_FULL-Hsa6-hg19.tab

## Compare Groups & Differential Splicing Analysis
https://github.com/vastgroup/vast-tools#comparing-psis-between-samples
https://github.com/vastgroup/vast-tools#differential-splicing-analysis

-   `compare`: pre-filters the events based on read coverage, imbalance and other features, and simply compares average and individual dPSIs. It looks for non-overlapping PSI distributions based on fixed dPSI cut-offs. For more than 3 replicates, it is likely to be too stringent.
-   `diff`: performs a statistical test to assess whether the PSI distributions of the two compared groups are significantly different. It is possible to pre-filter the events based on the minimum number of reads per sample, but subsequent filtering is highly recommended (e.g. overlapping the results with the output of  `tidy`). For more than 5 samples per group it may also be over stringent.

As CAMP R module doesn't have psiplots R package need to create a conda environment to install packages. Once this environment has been made it is saved for future and can be activated using `source activate`. Then load relevant R module tools within the conda environment.
```bash
ml Anaconda2
ml R/3.5.1-foss-2016b-bare

source activate rtest
R
library("ggplot2")
library("optparse")
library("MASS")
library("RColorBrewer")
library("reshape2")
library("grid")
library("parallel")
library("devtools")
library("psiplot")
#quit R in cluster
q()
#save workspace image
y

```
To create a conda environment from scratch:
```bash
conda create -n rtest r-essentials r-devtools
source activate rtest
# install the package normally by calling R
R
install.packages("optparse")
install.packages("ggplot2")
install.packages("MASS")
install.packages("RColorBrewer")
install.packages("reshape2")
install.packages("devtools")
install.packages("psiplot")
devtools::install_github("kcha/psiplot")

#to deactivate environment
conda deactivate
```
run `vast-tools compare` and `vast-tools diff` in this rtest conda environment outside of R

There are 2 approaches to comparing samples: 
1. Time effect (delta change in differential splicing between different time points of iPSC differentiation in CTRLs and VCPs independently). Effectively is a pair-wise analysis like repeated measures but between 2 time points only.
2.  Mutant effect (VCP vs CTRL at specified time points)

### Time Effect

```bash
mkdir /home/camp/ziffo/working/oliver/projects/airals/splicing/raphaelle_vast_tools/time_effect
INFILE=/home/camp/ziffo/working/oliver/projects/airals/splicing/D7vsD0_VCP_vast_tools/vast_out/INCLUSION_LEVELS_FULL-Hsa2-hg19.tab
```
Run the Compare TIME-EFFECT section of the script in `/home/camp/ziffo/working/oliver/scripts/intron_retention/Splicing_VASTOOLS.sh`
This performs the VAST-tools Compare & Diff commands on each of the WT & VCP time-point comparisons
```bash
SAMPLE_A=VCP.d0
SAMPLE_B=VCP.d7
mkdir ~/working/oliver/projects/airals/splicing/D7vsD0_VCP_vast_tools/time_effect
OUT=~/working/oliver/projects/airals/splicing/D7vsD0_VCP_vast_tools/vast_out
cd /home/camp/ziffo/working/oliver/projects/airals/splicing/raphaelle_vast_tools/time_effect/VCP_d7
cp $INFILE .
sbatch -N 1 -c 8 --mem=40GB --wrap="vast-tools diff -i $INFILE -c 8 -a $SAMPLE_B -b $SAMPLE_A -o $OUT -d INCLUSION-FILTERED"
sbatch -N 1 -c 8 --mem=40GB --wrap="vast-tools compare INCLUSION_LEVELS_FULL-Hsa2-hg19.tab -a VCP.d7 -b VCP.d0 --min_dPSI 15 --min_range 5 --GO -sp Hsa --print_sets"

# run with more stringent cut offs to reduce number of events:
sbatch -N 1 -c 8 --mem=40GB --wrap="vast-tools diff -i $INFILE -c 8 -a $SAMPLE_B -b $SAMPLE_A -o $OUT -d -r 0.99 -m 0.2 INCLUSION-STRINGENT-FILTERED"
sbatch -N 1 -c 8 --mem=40GB --wrap="vast-tools compare INCLUSION_LEVELS_FULL-Hsa2-hg19.tab -a VCP.d7 -b VCP.d0 --min_dPSI 15 --min_range 5 --GO -sp Hsa --outFile Diff_INCLUSION-STRINGENT-FILTERED.tab --print_sets --plot_PSI"
```

### Mutant Effect

Run the Compare MUTATION-EFFECT section of the script in `/home/camp/ziffo/working/oliver/scripts/intron_retention/Splicing_VASTOOLS.sh`

Can use VAST-TOOLS here to calculate differentially expressed genes: `compare_expr`
Output file is created in directory of input file. This reports the differentially spliced AS events between the 2 groups (based on difference in average inclusion levels - delta PSI)

Output file of diff command = INCLUSION-FILTERED.tab

### Explore diff output files
Numbers in Luisier et al. Fig 1 are taken from these:
```bash
# order tab file by MV value: view in Atom
more INCLUSION-FILTERED.tab | sort -k6 -r | awk '{ if ($6 >= 0.2) { print } }' | awk '{ if ($5 >= 0) { print } }' > INCLUSION-ORDERED-BY-MV.tab

# count number of events with MV > 0.2:
awk '{ if ($6 >= 0.2) { print } }' INCLUSION-FILTERED.tab | wc -l
# count number of events with intron retention (E[dPsi] > 0):
awk '{ if ($6 >= 0.2) { print } }' INCLUSION-FILTERED.tab | awk '{ if ($5 >= 0) { print } }' | wc -l
# count number of events with Alternative Exon:
awk '{ if ($6 >= 0.2) { print } }' INCLUSION-FILTERED.tab | awk '{ if ($5 >= 0) { print } }' | wc -l
```
## Plotting VAST-TOOLS
https://github.com/vastgroup/vast-tools#plotting
uses psiplot in R

The input format follows the same format from the `combine` step. The output is a pdf of scatterplots (one per AS event) of PSI values. To execute from VAST-TOOLS, use the subcommand `plot`:

```bash
# filter INCLUSION table to only significant IR-C & IR-S events, retain header
awk 'NR==1;{ if ($2 ~ "HsaINT*") { print } }' DiffAS-Hsa2-hg19-dPSI15-range5-min_ALT_use25_VCP.d7-vs-VCP.d0.tab > significant_IRevents.tab

# run plot command
INFILE=/home/camp/ziffo/working/oliver/projects/airals/splicing/D7vsD0_VCP_vast_tools/vast_out/significant_IRevents.tab
vast-tools plot $INFILE

```

```bash
# filter INCLUSION table to only significant IR-C & IR-S events, retain header
awk 'NR==1;{ if ($2 ~ "HsaINT*") { print } }' INCLUSION-STRINGENT-FILTERED.tab > stringant_significant_IRevents.tab

# run plot command
INFILE=/home/camp/ziffo/working/oliver/projects/airals/splicing/D7vsD0_VCP_vast_tools/vast_out/DiffAS-Hsa2-hg19-dPSI15-range5-min_ALT_use25_VCP.d7-vs-VCP.d0.tab
vast-tools plot $INFILE
# specify specific genes to plot as sole PDF. Go through ORDERED-by-MV.tab genes
vast-tools plot $INFILE --gene=RPS24 -o PSI_plots/
```

## GO analysis on VAST-tools output

Use [DAVID](https://www.biostarhandbook.com/ontology/david-analysis.html)
https://david.ncifcrf.gov/home.jsp

Step 1: Load a gene list into the [site](https://david.ncifcrf.gov/tools.jsp)
Paste the output of vast-tools txt files from atom into the website:
IR_DOWN-INCLUSION-STRINGENT-FILTERED.txt
AltEx-INCLUSION-STRINGENT-FILTERED.txt

Step 2: Select Gene identifier Ensembl_gene_ID 

Step 3: Clarify List Type: Gene List as you only pasted in significant events (Background is if you paste in entire gene list)

Submit > Select input species (Homo sapiens)

Explore details through the Functional Annotation Tools:  table, chart & clustering reports

![enter image description here](https://www.biostarhandbook.com/ontology/images/summary.png)
Click Pathways > Chart in KEGG_PATHWAY row > note the Terms (these are the enriched GO terms) > click Genes bar to see individual genes

Download table as txt file > open in excel > copy the gene term & P-value columns > paste in to [revigo](http://revigo.irb.hr/) to further intepret 

Export & save results

**Plot GO p-values as Barplot in Rstudio:**
https://cran.r-project.org/web/packages/GOplot/vignettes/GOplot_vignette.html

```r
#If you want only to plot the GO terms in one condition
dat        <- enrich$value # the -log10(P-value)
names(dat) <- enrich$Terms #the description of your GO term
par(mfrow=c(1,1),mar=c(3,10,2,3),cex=0.7)
barplot(height = dat,horiz=T,las=1)

#If you want to plot side by side the same GO term but for 2 different conditions
dat               <- -cbind(enrich.bp$p.value.norm.ngf,enrich.bp$p.value.norm.nt3)[match(least.transported,enrich.bp$names.goterms.),]
rownames(dat)    <- enrich.bp$Term[match(least.transported,enrich.bp$names.goterms.)]
dat              <- dat[order(dat[,1],-dat[,2]),]

mycols           <- c("#81A4D6","#AF71AF")
L                <- nrow(dat)
val1             <- -as.vector(dat[,1])
val2             <-  as.vector(dat[,2])
par(mfrow=c(1,1),mar=c(3,10,2,3),cex=0.7)
plot(c(L+3,0),xlim=c(min(val1)-10,max(val2)),type = "n",frame=F,yaxt="n",ylab="")
mp=barplot(height = val2,add = TRUE,axes = FALSE,horiz=T,col=mycols[2])
barplot(height = val1,add = TRUE,axes = FALSE,horiz=T,col=mycols[1])
text(x=(min(val1)-5),y=mp,lab=rownames(dat),cex=0.6)
mtext(side=1,line=2,text="Z-score",cex=0.6)
abline(v=c(-2.5,2.5),col="grey",lty=2)
```

# Matt
[http://matt.crg.eu/](http://matt.crg.eu/)

Matt is a **Unix command-line toolkit** for analyzing genomic sequences with focus on the down-stream analysis of alternative splicing events. Being a POSIX-style command-line toolkit, Matt runs on the terminal of any Unix-like system. **Matt is modular**: It comprises many commands, each of which tailored to one specific task. Combining commands allows the user to solve complex tasks:  the whole is more than the sum of its parts. Matt works on tables as its data structure, where rows correspond to objects, e.g., exons, genes, etc., and columns correspond to features of these objects.

Matt usage goes through  **two phases**:
1. Data preparation: Fit the output from other programs, e.g., VASTOOLS estimation of inclusion levels of alternative splicing events, to the input format of Matt analyses. Works on the INCLUSION_LEVELS_FULL-Hsa2-hg19.tab file from `vast-tools combine` output.
2. Data analysis

The main focus of Matt is high-level analyses, but Matt also offers commands for data preparation/table manipulation. Commands produce **reports as PDF documents, PDF graphics, or web pages** summarizing the results.

To make the work with Matt more efficient,  **many Matt commands are designed to be combined by piping**. Exploiting this capability improves the work experience with Matt.

## Load R environment on cluster

Matt requires R.utils package for Rscript so load the rtest environment:
```bash
ml Anaconda2
#create conda environment
source activate rtest
# load R & load R.utils pacakge
R
library("R.utils")
# quit R
q()
y

```
Access help pages:
`matt`
`matt COMMAND`

NB to deactivate `rtest` environment:
`source deactivate rtest`

## Data Preparation / Table manipulation

The workflow is: 
1. pre-process the output table of VAST-TOOLS for extracting PSI values with [get_vast](http://matt.crg.eu/#get_vast)
2. define groups of events to be analyzed with [def_cats](http://matt.crg.eu/#def_cats)
3. run high-level analysis e.g. [ir_features](http://matt.crg.eu/#get_ifeatures), [cmpr_introns](http://matt.crg.eu/#cmpr_introns)

![enter image description here](https://oup.silverchair-cdn.com/oup/backfile/Content_public/Journal/bioinformatics/35/1/10.1093_bioinformatics_bty606/1/m_bty606f1.png?Expires=1562078645&Signature=Ynp8fUlsBpcu2bphIP33~x0DH9ar6~4Mypdshqp6jPiqyaCDdApsmtqUfv-AYcQ3xCkxx2JlbmxIO2v0saHJlXhednFoUjkn7-bETSpF5ENgGy5x6wIoF36GgSAApntzlVofhChuMSpX8bURt1vhSN5fU~-qWmBcr-G6ajkBwV3s~~zdW7yBeyGqk7MvH5gW~hWbR9bzCNEpsdjNIlqHwKxyTzGWBy4iAxFJ9JMA1uUkgnOwDOcJJw33uMluPlSM9m09nFCzvp4T7DbV-nsAnysG-sBSmYQguRL4GCVG56b4-Mj~qNbC3UM4hEDT34~4jTN92UkeKCOAalwcwhMrvA__&Key-Pair-Id=APKAIE5G5CRDK6RD3PGA)

### Import VAST-TOOLS results tables
[http://matt.crg.eu/#get_vast](http://matt.crg.eu/#get_vast)

`matt get_vast` command is tailored to the result tables of [VAST-TOOLS](https://github.com/vastgroup/vast-tools). Use this command to transform the output table from VAST-TOOLS into Matt format and extract sub-sets of reported alternative splicing events. VAST-TOOLs reports skipped exons, retained introns, alt3, alt5 events in one output table together with estimated PSI values across several data sets. Matt allows you to retrieve form this table events of specific types (skipped exons, retained introns, Alt3, Alt5) or events that have a PSI value within a user specified range across all samples. 

**Workflow**
Uses INCLUSION_LEVELS_FULL-Hsa6-hg19.tab being a final results table from VAST-TOOLS `combine` command:

Inspect this INCLUSION_LEVELS_FULL table:
```bash
# extract all intron retention events & PSI values (min, max & mean) from vast tools output table for the comparison of samples -a vs -b (if merged then use the group names rather than the individual sample names). To be included PSI values need to have a minimum quality flag of LOW across all samples:
vts_file=~/working/oliver/projects/airals/splicing/D7vsD0_VCP_vast_tools/vast_out/INCLUSION_LEVELS_FULL-Hsa2-hg19.tab

#inspect columns of this table
matt get_colnms $vts_file
# print table to screen (inclusion levels range [0;100])
matt prnt_tab $vts_file -W 10 | less -S -
# back to normal view
q
# list and check all distinct entries in column COMPLEX, which encode the type of AS event:
matt col_uniq $vts_file COMPLEX
```

## Skipped Exons Workflow
as per shell script chapter 6: [http://matt.crg.eu/examples/nova2/all_analysis_steps.html](http://matt.crg.eu/examples/nova2/all_analysis_steps.html) 
Extract from vast-tools output table **skipped exons**

Use [get_vast](http://matt.crg.eu/#get_vast)  later which works with VAST-tools standard types of alternative splicing events which are: S, C1, C2, C3, MIC, IR-S, IR-C, Alt3, Alt5. It outputs an augmented table, from which we select only intron events by re-directing this table to [get_rows](http://matt.crg.eu/#get_rows). The final output table gets re-directed into file introns.tab.

```bash
vts_file=~/working/oliver/projects/airals/splicing/D7vsD0_VCP_vast_tools/vast_out/INCLUSION_LEVELS_FULL-Hsa2-hg19.tab
GTF=~/working/oliver/genomes/annotation/Homo.gtf
matt get_vast $vts_file -complex S,C1,C2,C3,MIC -a VCP.d7 -b VCP.d0 -minqab VLOW -minqglob N -gtf $GTF -f gene_id > exons.tab

# check output
# check number of events
matt col_uniq exons.tab COMPLEX
# check column names
matt get_colnms exons.tab

# define groups of skipped exons to be compared
matt def_cats exons.tab GROUP 'silenced=DPSI_GRPA_MINUS_GRPB[25,100]' 'enhanced=DPSI_GRPA_MINUS_GRPB[-100,-25]' 'unregulated=DPSI_GRPA_MINUS_GRPB[-1,1] PSI_VCP.d0[10,90]' | matt add_cols exons.tab -
# check of number events per group in exons.tab
matt col_uniq exons.tab GROUP
# print exons.tab to screen. exit with q
matt prnt_tab exons.tab -W 10 | less -S -

# Feature extraction of exons
GTF=~/working/oliver/genomes/annotation/Homo.gtf
FASTA=~/working/oliver/genomes/sequences/human/Hsa19_gDNA.fasta
outdir=cmpr_exons
sbatch -N 1 -c 8 --mem=40GB --wrap="matt cmpr_exons exons.tab START END SCAFFOLD STRAND GENEID $GTF $FASTA Hsap 150 GROUP[silenced,enhanced,unregulated] $outdir -colors:deepskyblue,firebrick4,lightgreen"
```
Output folder contains a
1.) summary PDF report with box plots (summary.pdf)
2.) a table with all events and extracted features (exons_with_efeatures.tab)
```
matt prnt_tab exons_with_efeatures.tab -W 10 | less -S -
```
3.) a table with all p values and information on performed statistical tests (overview_of_features_and_comparisons.tab)
```
matt prnt_tab overview_of_features_and_comparisons.tab -W 10 | less -S -
```
4.) all box plots as PDFs in directory summary_graphics

Generate motif RNA-map comparing 3 groups: silenced, enhanced, unregulated
```bash
FASTA=~/working/oliver/genomes/sequences/human/Hsa19_gDNA.fasta
outdir=exons_cisbprna_maps
sbatch -N 1 -c 8 --mem=40GB --wrap="matt rna_maps_cisbp exons.tab UPSTRM_EX_BORDER START END DOSTRM_EX_BORDER SCAFFOLD STRAND GROUP[silenced,enhanced,unregulated] 31 35 135 $FASTA cisbprna_regexps -d $outdir"
```
Output folder contains:
1.) summary with all motif RNA-maps (0_all_motif_rna_maps.pdf); each motif RNA-map is printed twice: with and without a data coverage plot. The maps are ordered according to differences of the enrichment scores between groups
(silenced vs. unregulated and enhanced vs unregulated) from most positive differences to most negative differences. Hence, maps with most differences come first and last.
2.) all motif RNA-maps as PDFs

## Intron Retention Workflow
Extract from vast-tools output table **intron retention** events.

```bash
vts_file=~/working/oliver/projects/airals/splicing/D7vsD0_VCP_vast_tools/vast_out/INCLUSION_LEVELS_FULL-Hsa2-hg19.tab
# specify Hg19 GTF as Homo.gtf - chromosome scaffolds ID added chr to start using: perl -ne 'unless(/^#|^GL/){$_="chr$_"}print' < $GTF > Homo.gtf   - help from Manuel Irimia with this.
GTF=~/working/oliver/genomes/annotation/Homo.gtf
matt col_uniq $vts_file COMPLEX
matt get_vast $vts_file -complex IR-C,IR-S -a VCP.d7 -b VCP.d0 -minqab VLOW -minqglob N -gtf $GTF -f gene_id > introns.tab
# check events
matt col_uniq introns.tab COMPLEX
# check col names
matt get_colnms introns.tab
# define groups
matt def_cats introns.tab GROUP 'silenced=DPSI_GRPA_MINUS_GRPB[25,100]' 'enhanced=DPSI_GRPA_MINUS_GRPB[-100,-25]' 'unregulated=DPSI_GRPA_MINUS_GRPB[-1,1] PSI_VCP.d0[10,90]' | matt add_cols introns.tab -
# check events/group
matt col_uniq introns.tab GROUP

# Run Feature extraction: cmpr_introns http://matt.crg.eu/#cmpr_introns
GTF=~/working/oliver/genomes/annotation/Homo.gtf
FASTA=~/working/oliver/genomes/sequences/human/Hsa19_gDNA.fasta
output_dir=cmpr_introns

# run cmpr_introns - output goes into cmpr_1 folder - takes ~20mins
sbatch -N 1 -c 8 --mem=40GB --wrap="matt cmpr_introns introns.tab START END SCAFFOLD STRAND GENEID $GTF $FASTA Hsap 150 GROUP[silenced,enhanced,unregulated] $output_dir -colors:deepskyblue,firebrick4,lightgreen"
```
Output folder contains:
1.) summary PDF report with box plots (summary.pdf)
2.) a table with all events and extracted features (introns_with_efeatures.tab)
3.) a table with all p values and information on performed statistical tests (overview_of_features_and_comparisons.tab)
4.) all box plots as PDFs in directory summary_graphics

Generate RNA-maps comparing the intron groups
```bash
FASTA=~/working/oliver/genomes/sequences/human/Hsa19_gDNA.fasta
output_dir=introns_cisbprna_maps
sbatch -N 1 -c 8 --mem=40GB --wrap="matt rna_maps_cisbp introns.tab UPSTRM_EX_BORDER START END DOSTRM_EX_BORDER SCAFFOLD STRAND GROUP[silenced,enhanced,unregulated] 31 35 135 $FASTA cisbprna_regexps -d $output_dir"
```
Output folder contains:
1.) summary with all motif RNA-maps (0_all_motif_rna_maps.pdf); each motif RNA-map is printed twice: with and without a data coverage plot. The maps are ordered according to differences of the enrichment scores between groups
(silenced vs. unregulated and enhanced vs unregulated) from most positive differences to most negative differences. Hence, maps with most differences come first and last.
2.) all motif RNA-maps as PDFs



![enter image description here](http://matt.crg.eu/graphics/ov_introns.png)
 `get_ifeatures` command is included in cmpr_introns - it retrieves 50 features of interest for introns. Introns need to be described by a table with basic information (genomic coordinates, gene ID of genes the introns belong to). 
```bash
GTF=~/working/oliver/genomes/annotation/Homo.gtf
FASTA=~/working/oliver/genomes/sequences/human/Hsa19_gDNA.fasta
#run get_ifeatures - takes ~20mins
sbatch -N 1 -c 8 --mem=40GB --wrap="matt get_ifeatures ir_events.tab START END SCAFFOLD STRAND GENEID $GTF $FASTA Hsap -f gene_id > ifeatures.tab"
```
This writes a table with columns containing the feature values & extracted sequence of intron, up/downstream exon & splice sites. If only one or a few features are of interest, users can apply [get_cols](http://matt.crg.eu/#cmpr_exons)  and extract specific feature columns only.

---
additional to remove  
#### Create introns.tab 
 
introns.tab  describes introns with these columns (and maybe more)
1.  START
2.  END
3.  SCAFFOLD
4.  STRAND
5.  GENEID_ENSEMBL
6.  DATASET: containing group IDs of introns (down, up, ndiff)

`introns.tab` contains for all intron information about their genomic coordinate, gene, and their intron features including intron sequence, splice site sequences, sequences of up/down-stream exons. 
  
`introns.tab` is used as input for  [cmpr_introns](http://matt.crg.eu/#cmpr_introns), which extracts features for all introns and appends them to the input table. As a consequence, the input table  **must not**  already contain columns with identical column names because column names in a table must be unique. Hence, from introns.tab we select only the important columns and neglect the already added columns with intron features.

```bash
matt get_cols introns.tab SCAFFOLD START END STRAND ENSEMBL_GENEID COMPLEX VCP.d0 VCP.d0-Q VCP.d7 VCP.d7-Q > tmp.tab
mv tmp.tab introns_testsets.tab
# Check the number of introns in the final table introns_testsets.tab confirms that the sampling worked as expected.
matt col_uniq introns.tab COMPLEX
```
Color names can be found [here](http://www.stat.columbia.edu/~tzheng/files/Rcolor.pdf)
  
The  [PDF report](http://matt.crg.eu/examples/cmpr_exons/cmpr_1/summary.pdf)  contains all details and plots and is organized into:
1.  Overview of results of permutation tests across all comparisons and features
2.  Box-plots for all numerical features with all intron groups tested
3.  Comparative sequence plots for all groups of splice sites, branch points for all comparisons
4.  Details of results of permutation tests
  
DATASET[down,ndiff]: the last group specified should be the reference group (like non-differentially spliced).  
DATASET[down,ndiff] compares group down vs. ndiff, DATASET[down,up,ndiff] compares down vs. up, down vs. ndiff, up vs. ndiff.

#### Run RNA maps
[http://matt.crg.eu/#rna_maps](http://matt.crg.eu/#rna_maps)

Produce motif RNA-maps for comparing enrichment of binding motifs of RNA/DNA binding proteins for different groups of exons or introns. Motifs can be defined as **position weight matrix models (PWMs)** or **Perl regular expressions**. This command extracts sequences from proximity of given exons/introns, predict hits of the specified motifs therein, and computes motif enrichment scores. A **permutation test** is applied for estimating p values - regions with significant differences in motif enrichment are highlighted in the motif RNA-map.  

**Output:**
1.  PDF report containing all motif RNA-maps
2.  PDF report with motif RNA-maps with significant enrichment only (with arguments -p, -fdr)
3.  For each map, there are two versions, one with a data coverage section, and one without. The data coverage section shows the data overage along the map, which might be less then 100% if the data set contains exons/introns shorter than the specified exon/intron sections to be visualized.
4.  The motif RNA-maps are ordered according to largest differences of motif enrichment scores; Maps with largest positive differences come first, maps with largest negative differences last. The reference group for computing differences is always the last group specified in the program call.
5.  all motif RNA-maps as PDF graphics

## RNA motif maps
LOOK UP
UNDERSTAND GROUPS: enhanced vs silenced vs unregulated

---

RAPHAELLE APPROACH:

## Coverage for introns of interest
To perform the a focussed analysis of the 167 retained introns identified using VAST-tools, 
Run Sections A, B,  C "Import time effect" from `import_VASTOOLS.R` script located in `/home/camp/ziffo/working/oliver/scripts/intron_retention` 
Then run `get_relative_coverage_inteactive.R` script.  
Also note other R scripts:
`characterise_introns.R`
This approach uses a combination of threshold but accounts for the depth of coverage between different samples. Uses [MaxEntScan of 5' and 3' end](http://genes.mit.edu/burgelab/maxent/Xmaxentscan_scoreseq.html) to identify splice sites. 
First run the global analysis to get the big picture of splicing events in VCP vs CTRL.  SVD & PCA analysis clustering by mutation. 
Then run detailed analysis according to specific genes or features. Multivariate analysis of the 2 VCP mutants. 
to be run in R which calculates a ratio of intron sequence coverage and surrounding exons. 
1. First check that the gene where the event is occurring is expressed. 
2. Then check the event exhibits a change over time of at least 10%. 
3. Finally inspect visually all selected IR events occurring at Day 7 NPC stage in VCP mutant and at Day 14 pMN stage in CTRL to end up with the list of 167 events to get a high confidence list.
Import the results obtained from VAST-tools (remember analysis in VCP & CTRL over time performed initially independently) 
To then get a value of IR across diverse data-sets I then wrote the custom code that computed the ratio between coverage of the intron versus average coverage of the neighbouring exons. Then select the events of interest. The `get_relative_coverage_interactive.R` script is located in `/home/camp/ziffo/working/oliver/scripts/intron_retention`


# SGSeq
Jack Humphrey uses this
Vignette](http://127.0.0.1:28705/library/SGSeq/doc/SGSeq.html)
[Bioconductor page](https://bioconductor.org/packages/release/bioc/html/SGSeq.html)



# DEXSeq

https://bioconductor.org/packages/release/bioc/html/DEXSeq.html
https://bioconductor.org/packages/release/bioc/vignettes/DEXSeq/inst/doc/DEXSeq.pdf
https://bioconductor.org/packages/release/bioc/manuals/DEXSeq/man/DEXSeq.pdf
https://genome.cshlp.org/content/22/10/2008

Measures differential exon usage (DEU) which indicates alternative splicing. DEU also measures alternative transcript start sites & polyadenylation sites (differential usage of exons at 5' and 3' boundary of transcripts). 

Calculates ratio = number of transcripts from the gene containing this exon / total number of transcripts from the gene.
Then compares this ratio between conditions assessing the strength of the fluctuations (dispersion). 

Steps:
1. Alignment to genome using splice aware aligner eg STAR
2. Counts using DEX Seq Python scripts (utilises HTSeq)

```bash
ml HTSeq
ml Python/2.7.15-GCCcore-7.3.0-bare
ml R

# use 2 Python scripts (dexseq_prepare_annotation.py & dexseq_count.py)
# load R environment with DEXSeq in
source activate rtest
R
library("DEXSeq")
#exit R
q()

# Create output folder
mkdir -p DEXSeq

### RUN ANNOTATION SCRIPT
# set GTF - Ensembl (gencode)
GTF=/home/camp/ziffo/working/oliver/genomes/annotation/gencode.v28.primary_assembly.annotation.gtf
# set GFF output
OUT=/home/camp/ziffo/working/oliver/genomes/annotation/DEXSeq.homo_sapiens.GRCh38.gencode.v28.gff
SCRIPT=/camp/home/ziffo/R/x86_64-pc-linux-gnu-library/3.5/DEXSeq/python_scripts/dexseq_prepare_annotation.py

#run dexseq_prepare_annotation.py script
python $SCRIPT $GTF $OUT


### RUN COUNT SCRIPT
# set GFF input made in annotation script
GFF=/home/camp/ziffo/working/oliver/genomes/annotation/DEXSeq.homo_sapiens.GRCh38.gencode.v28.gff
SCRIPT=/camp/home/ziffo/R/x86_64-pc-linux-gnu-library/3.5/DEXSeq/python_scripts/dexseq_count.py
#set BAM input file
SAM=/home/camp/ziffo/working/oliver/projects/airals/alignment/D7_samples/trimmed_filtered_depleted/SRR5483796.sam
OUT=/home/camp/ziffo/working/oliver/projects/airals/splicing/DEXSeq/SRR5483796.txt

#run dexseq_count.py
python $SCRIPT $GFF $SAM $OUT

#to deactivate environment
source deactivate
```
Now switch to R to run the DEXSeq analysis.

```r
# make new Rproject & save it in the relevant CAMP Cluster splicing folder.
setwd("/Volumes/lab-luscomben/working/oliver/projects/airals/splicing/DEXSeq")
suppressPackageStartupMessages( library( "DEXSeq" ) )

### PREPARE DATA ###
# read in files
countFiles = list.files("/Volumes/lab-luscomben/working/oliver/projects/airals/splicing/DEXSeq/", pattern=".txt", full.names=TRUE) 
basename(countFiles)
flattenedFile = list.files("/Volumes/lab-luscomben/working/oliver/genomes/annotation/", pattern="gff", full.names=TRUE) 
basename(flattenedFile)

# create sample table: 1 row for each library. columns for file name & read counts, sample
sampleTable = data.frame(
  row.names = c( "SRR5483788", "SRR5483789", "SRR5483790", "SRR5483794", "SRR5483795", "SRR5483796" ), 
  condition = c("VCP", "VCP", "VCP", "CTRL", "CTRL", "CTRL" ) )
# check table
sampleTable
# create DEXSeqDataSet 
dxd = DEXSeqDataSetFromHTSeq(
  countFiles,
  sampleData=sampleTable,
  design= ~ sample + exon + condition:exon,
  flattenedfile=flattenedFile )
# see structure of dxd
sampleAnnotation( dxd )
colData(dxd)
# see first 5 rows from count data - 12 columns (first 6 = reads mapping to exons, last 6 = counts mapping to rest of exons from same gene)
head( counts(dxd), 5 )
#show details of exon bins annotation
head( rowRanges(dxd), 3 )

### DEXSeq ANALYSIS ###
#normalise for different sequencing depths between samples
dxd = estimateSizeFactors( dxd )
# Perform differential exon usage test in a single command (this by passes the steps below - takes 10 mins)
dxd = DEXSeq(dxd,
       fullModel=design(dxd),
       reducedModel = ~ sample + exon,
       BPPARAM=MulticoreParam(workers=1),
       fitExpToVar="condition")

# OPTIONAL: dispersion estimation - takes 30 mins!
dxd = estimateDispersions( dxd )
# plot the per-exon dispersion estimates vs mean normalised count
plotDispEsts( dxd )

# test for differential exon usage (DEU) between conditions
dxd = testForDEU( dxd )
# estimate relative exon usage fold changes
dxd = estimateExonFoldChanges( dxd, fitExpToVar="condition")

### RESULTS ###
# summarise the resuls
dxr1 = DEXSeqResults( dxd )
# description of each column in DEXSeqResults
mcols(dxr1)$description
# how many exons are significant (with false discovery rate 10% - can change this threshold to more stringent)
table ( dxr1$padj < 0.1 )
# how many genes are affected
table ( tapply( dxr1$padj < 0.1, dxr1$groupID, any ) )
# MA plot: log fold change vs avg normalised count per exon. Significant exons in red
plotMA( dxr1, cex=0.8 )
### Detailed overview of analysis results in HTML - see the geneIDs of differentially spliced genes
DEXSeqHTML( dxr1, FDR=0.1, color=c("#FF000080", "#0000FF80") )

### VISUALISATION ###
# visualise results for individual genes
plotDEXSeq( dxr1, "ENSG00000116560", legend=TRUE, cex.axis=1.2, cex=1.3, lwd=2 )

# visualise transcript models to see isoform regulation
plotDEXSeq( dxr1, "ENSG00000116560", displayTranscripts=TRUE, legend=TRUE, cex.axis=1.2, cex=1.3, lwd=2 )

# visualise individual samples rather than model effect estimates.
plotDEXSeq( dxr1, "ENSG00000116560", expression=FALSE, norCounts=TRUE,
            legend=TRUE, cex.axis=1.2, cex=1.3, lwd=2 )

# remove overall changes from the plots
plotDEXSeq( dxr1, "ENSG00000116560", expression=FALSE, splicing=TRUE,
            legend=TRUE, cex.axis=1.2, cex=1.3, lwd=2 )
```

# rMATS
ml Python/2.7.15-GCCcore-7.3.0-bare
ml numpy
ml OpenBLAS
ml STAR
ml ScaLAPACK
ml GSL
ml GCC

[User Tutorial](http://rnaseq-mats.sourceforge.net/user_guide.htm)
http://rnaseq-mats.sourceforge.net/
https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4280593/

MATS is a computational tool to detect differential alternative splicing events from RNA-Seq data. The statistical model of MATS calculates the P-value and false discovery rate that the difference in the isoform ratio of a gene between 2 conditions exceeds a given user-defined threshold. From the RNA-Seq data, MATS can automatically detect and analyze alternative splicing events corresponding to all major types of alternative splicing patterns. MATS handles replicate RNA-Seq data from both paired and unpaired study design.

![enter image description here](http://rnaseq-mats.sourceforge.net/splicing.jpg)

# Perform GO analysis of gene list of differentially spliced genes

### Data Preparation

```bash
# convert splicing_diff.tab > .csv
perl -lpe 's/"/""/g; s/^|$/"/g; s/\t/","/g' < input.tab > output.csv
```
Open CSV in Excel
Copy the significant Gene Symbols into DAVID (see Gene Enrichment Chapter)

## topGO

### Prepare data from VAST tools splicing output

```r
library(tidyverse)
setwd("/Volumes/lab-luscomben/working/oliver/projects/airals/splicing/vast_tools/vast_out") #set working directory where splicing files are stored on cluster
diff_splicing <- read_tsv("diff.splicing_mv0.2.tab") #import differential splicing table .tab file
diff_splicing <- data.table(diff_splicing)

diff_splicing$pval <- as.numeric(diff_splicing$`E[dPsi]`)
diff_splicing$MV   <- as.numeric(diff_splicing$`MV[dPsi]_at_0.95`)

genelistUp <- factor( as.integer( diff_splicing$MV > .2 & diff_splicing$pval > 0) )
names(genelistUp) <- rownames(diff_splicing)
genelistDown <- factor( diff_splicing$MV > 0.2 & diff_splicing$pval < 0 )
names(genelistDown) <- rownames(diff_splicing)
```

### Run the topGO commands & make bar plots for BP, CC & MF (up & downregulated)

```r
### GO analysis
library(topGO)
library(GOstats)
library(goseq)
library(org.Mm.eg.db)
library(ggplot2)

### Test UPREGULATED GENES
#Test Biological Processes BP sub-ontology
myGOdata      <- new( "topGOdata", ontology = "BP", allGenes = genelistUp, nodeSize = 10,
                 annot = annFUN.org, mapping = "org.Hs.eg.db", ID="Entrez" )
goTestResults <- runTest( myGOdata, algorithm = "elim", statistic = "fisher" )
UR_BP_table   <- GenTable( myGOdata, goTestResults )
enrich.ur.bp  <- transform(UR_BP_table,result1 = as.numeric(result1))
enrich.ur.bp$value <- -log10(enrich.ur.bp$result1)
# Plot GO p-values as as bar plot
dat.ur.bp        <- enrich.ur.bp$value # the -log10(P-value)
names(dat.ur.bp) <- enrich.ur.bp$Term #the description of your GO term
barplot(height = dat.ur.bp, space = 0.5, horiz=T,las=1,font.size = 10)

#Test Cellular Compartment (CC) sub-ontology
myGOdata <- new( "topGOdata", ontology = "CC", allGenes = genelistUp, nodeSize = 10,
                 annot = annFUN.org, mapping = "org.Hs.eg.db", ID="Entrez" )
goTestResults <- runTest( myGOdata, algorithm = "elim", statistic = "fisher" )
UR_CC_table <- GenTable( myGOdata, goTestResults )
enrich.ur.cc <- transform(UR_CC_table,result1 = as.numeric(result1))
enrich.ur.cc$value <- -log10(enrich.ur.cc$result1)
# Plot GO p-values as as bar plot
dat.ur.cc        <- enrich.ur.cc$value # the -log10(P-value)
names(dat.ur.cc) <- enrich.ur.cc$Term #the description of your GO term
par(mfrow=c(1,1),mar=c(3,20,3,3),cex=0.7)  # artificially set margins for barplot to follow (need large left hand margin for names)
barplot(height = dat.ur.cc,horiz=T,las=1, font.size = 20)

#Test Molecular function (MF) sub-ontology
myGOdata <- new( "topGOdata", ontology = "MF", allGenes = genelistUp, nodeSize = 10,
                 annot = annFUN.org, mapping = "org.Hs.eg.db", ID="Entrez" )
goTestResults <- runTest( myGOdata, algorithm = "elim", statistic = "fisher" )
UR_MF_table <- GenTable( myGOdata, goTestResults )
enrich.ur.mf <- transform(UR_MF_table,result1 = as.numeric(result1))
enrich.ur.mf$value <- -log10(enrich.ur.mf$result1)
### Plot GO p-values as as bar plot
dat.ur.mf        <- enrich.ur.mf$value # the -log10(P-value)
names(dat.ur.mf) <- enrich.ur.mf$Term #the description of your GO term
par(mfrow=c(1,1),mar=c(3,20,3,3),cex=0.7)  # artificially set margins for barplot to follow (need large left hand margin for names)
barplot(height = dat.ur.mf,horiz=T,las=1, font.size = 20)

### TEST DOWNREGULATED GENES
#Test Biological Processes BP sub-ontology
myGOdata <- new( "topGOdata", ontology = "BP", allGenes = genelistDown, nodeSize = 10,
                 annot = annFUN.org, mapping = "org.Hs.eg.db", ID="Entrez" )
goTestResults <- runTest( myGOdata, algorithm = "elim", statistic = "fisher" )
DR_BP_table <- GenTable( myGOdata, goTestResults )
enrich.dr.bp <- transform(DR_BP_table,result1 = as.numeric(result1))
enrich.dr.bp$value <- -log10(enrich.dr.bp$result1)
### Plot GO p-values as as bar plot
dat.dr.bp        <- enrich.dr.bp$value # the -log10(P-value)
names(dat.dr.bp) <- enrich.dr.bp$Term #the description of your GO term
par(mfrow=c(1,1),mar=c(3,20,3,3),cex=0.7)  # artificially set margins for barplot to follow (need large left hand margin for names)
barplot(height = dat.dr.bp,horiz=T,las=1, font.size = 20)

#Test Cellular Compartment (CC) sub-ontology
myGOdata <- new( "topGOdata", ontology = "CC", allGenes = genelistDown, nodeSize = 10,
                 annot = annFUN.org, mapping = "org.Hs.eg.db", ID="Entrez" )
goTestResults <- runTest( myGOdata, algorithm = "elim", statistic = "fisher" )
DR_CC_table <- GenTable( myGOdata, goTestResults )
enrich.dr.cc <- transform(DR_CC_table,result1 = as.numeric(result1))
enrich.dr.cc$value <- -log10(enrich.dr.cc$result1)
### Plot GO p-values as as bar plot
dat.dr.cc        <- enrich.dr.cc$value # the -log10(P-value)
names(dat.dr.cc) <- enrich.dr.cc$Term #the description of your GO term
par(mfrow=c(1,1),mar=c(3,20,3,3),cex=0.7)  # artificially set margins for barplot to follow (need large left hand margin for names)
barplot(height = dat.dr.cc,horiz=T,las=1, font.size = 20)

#Test Molecular function (MF) sub-ontology
myGOdata <- new( "topGOdata", ontology = "MF", allGenes = genelistDown, nodeSize = 10,
                 annot = annFUN.org, mapping = "org.Hs.eg.db", ID="Entrez" )
goTestResults <- runTest( myGOdata, algorithm = "elim", statistic = "fisher" )
DR_MF_table <-GenTable( myGOdata, goTestResults )
enrich.dr.mf <- transform(DR_MF_table,result1 = as.numeric(result1))
enrich.dr.mf$value <- -log10(enrich.dr.mf$result1)
### Plot GO p-values as as bar plot
dat.dr.mf        <- enrich.dr.mf$value # the -log10(P-value)
names(dat.dr.mf) <- enrich.dr.mf$Term #the description of your GO term
par(mfrow=c(1,1),mar=c(3,20,3,3),cex=0.7)  # artificially set margins for barplot to follow (need large left hand margin for names)
barplot(height = dat.dr.mf,horiz=T,las=1, font.size = 20)
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjQxMjY3MDI1LC0xMzU0NDUxMTExLDk4Nj
cxMTA4Myw0NDM1NjI0NzUsMjA1OTQ0NzQ1MSwtMTgyNDY2OTQx
NywtMTcwMDM1NDc1OSwyMDY3NDAxOTA2LC0xNzA5NTU3NjM5LD
E5Mjc1MjExNzksNzczNjc2NDU2LC0xMDcwMTU1NjM1LDEyNTIz
NDg5NTcsMTExNTMzMjU4NCwzMzc1MDI1MTYsLTIwNzYyNjM2MT
AsMTI0NzA4OTEwNywtMTUyNjYwNjc2NSwtNTIzNjc1Mjc5LC0x
MzEzMTM3MTA0XX0=
-->