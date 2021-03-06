> # Gene Ontology

[https://www.biostarhandbook.com/gene-set-enrichment.html](https://www.biostarhandbook.com/gene-set-enrichment.html)
Now that you have identified the differentially expressed genes (either up or down regulated), we need to identify their function. This is performed in a Gene Set Overlap analysis. Do the up or down regulated genes have anything in common?  

# Gene Ontology
[https://www.biostarhandbook.com/gene-ontology.html](https://www.biostarhandbook.com/gene-ontology.html)

Gene Ontology (GO) is a vocabulary used to describe genes functions with respect to 3 aspects:
- Cellular Component (CC): cellular location **where** product exhibits its effect
- Molecular function (MF): **How** does gene work?
- Biological Process (BP): **What** is the gene product purpose?

GO terms are orgnised in directed acyclic graphs where relationships and heirachy are represented. View GO relationships at http://geneontology.org/ or https://www.ebi.ac.uk/QuickGO/


# GO functional analysis
https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1002375
https://www.bioconductor.org/packages/release/bioc/vignettes/topGO/inst/doc/topGO.pdf

There is 2 steps to this:
### 1. Overrepresentation analysis ORA (using Fisher test)

Look at collection of genes within a pathway found in a gene set (rather than individual genes) 
Is the more differentially expressed genes within a gene collection (e.g. RNA metabolism) than we would expect if genes were expressed randomly.

### 2. Enrichment sets

Use reporting tools to write a table of GO analysis results to a HTML file. 
Select genes of interest > run hyperGTest > make GO report

The hypergeometric test (Fisher's test) is used to test whether genes enriched in the list of up or down regulated genes occur more often than expected by chance.
Fisher Exact test = is there a relationship between 2 categorical variables? 
2x2 contingency table: present in gene set vs member of gene set:

|  | Differentially Expressed | Not Differentially Expressed
|--|--|--|
| Gene Ontology Term | 50 | 200
| No Gene Ontology Term| 1950 | 17800

GO Download page: http://geneontology.org/page/download-annotations
2 files to download: 
1. definition (term) file `wget http://purl.obolibrary.org/obo/go.obo`
2. association file `wget http://geneontology.org/gene-associations/goa_human.gaf.gz`. In GAF compressed format defined at http://geneontology.org/page/go-annotation-file-gaf-format-21
Contains both gene and protein IDs.
To search the function of a gene use the [GeneCards](http://www.genecards.org/) database to easily locate the gene by name.

### Gene Set Enrichment Analysis
- Identify common characteristics within a list of genes. When using GO terms, this is called "functional enrichment"
- Most common variant is the ORA (over-representation analysis): 

Examine genes in a list > summarises the GO annotations for each gene > determines if annotations are statistically over-represented.

 **GO enrichment tools** 

- [topGO](https://bioconductor.org/packages/release/bioc/html/topGO.html) Bioconductor package (used by Luisier et al 2018)
- AgriGO Web-based GO Analysis Toolkit and Database for Agricultural Community.
- DAVID This is the GO tool biologists love - produces copious amounts of output. 
- Panther is offered directly from the GO website. It produces very limited information and no visualization.
- goatools a command line Python scripts.
- ermineJ Standalone tool with easy to use interface. It has detailed documentation.
- GOrilla Web-based; good visualization; downloadable results (as Excel files); easily see which genes contribute to which enriched terms; results pages indicate date of last update GO database (often within the last week).
- ToppFun - a suite of tools for human genomes that integrates a surprising array of data sources.
- [g:Profiler](https://biit.cs.ut.ee/gprofiler/) performs functional enrichment analysis and analyses gene lists for enriched features.  Very good visualiser of GO terms. 
- [g:sorter](https://biit.cs.ut.ee/gprofiler/gsorter.cgi) finds similar genes in public  transcroptomic data. Input = single gene & dataset of interest. Result = sorted gene list similarly expressed with gene of interest. For global gene expression analyses, across different species use [Multi Experiment Matrix](https://biit.cs.ut.ee/mem/) tool.

# Workflow

#### Gene Set Enrichment Analysis Methodology
1. Enter gene names to be studies
2. Enter background gene names (usually all genes for the organism)
3. Perform statistical comparison

The **Functional Annotation Tool** maps the genes to annotation content providing a summary of the biological interpretation of the data.

Perform **Fisher's Exact Test** to measure gene enrichment in annotation terms. The EASE score is a slightly modified Fisher's Exact p-value. The smaller to p-value, the more enriched the term.

## Prepare the data for analysis
```r
### Gene Enrichment Functional Annotation
columns(Homo.sapiens)
# remove decimal string & everything that follows from row.names (ENSEMBL) and call it "tmp": https://www.biostars.org/p/178726/ (alternatively use biomaRt )
tmp=gsub("\\..*","",row.names(res))
# use mapIds to add columns to results table. Use Ensebml nomenclature. 
res$symbol <- mapIds(Homo.sapiens,
                     keys=tmp,
                     column="SYMBOL",
                     keytype="ENSEMBL",
                     multiVals="first")
# ignore the error: 'select()' returned 1:many mapping between keys and columns
res$entrez <- mapIds(Homo.sapiens,
                     keys=tmp,
                     column="ENTREZID",
                     keytype="ENSEMBL",
                     multiVals="first")
res$symbol <- mapIds(Homo.sapiens,
                     keys=tmp,
                     column="GENENAME",
                     keytype="ENSEMBL",
                     multiVals="first")
resOrdered <- res[order(res$padj),] #order by padj
head(resOrdered)

### Prepare table for gene enrichment
resTested <- resLFC1[ !is.na(resLFC1$padj), ] # use DESeq2 resLFC1 command to subset results table to only genes with sufficient read coverage
resTested <- res[order(res$padj),] #order by padj
temp=gsub("\\..*","",row.names(resTested)) # remove decimal string & everything that follows from row.names (ENSEMBL)
rownames(resTested) <- temp #set the temp as the new rownames

#view the dataframe with ENSEMBL symbol external gene IDs & associated padj value
resTestedDF <- as.data.frame(resTested) # convert to dataframe
col_symbol <- grep("symbol", names(resTestedDF)) # select only ENSEMBL symbols column
resTestedDF <- resTestedDF[, c(col_symbol, (1:ncol(resTestedDF))[-col_symbol])] # move symbol column within DF to first column
head(resTestedDF) #check symbol is now first
resTestedDT <- data.table(resTestedDF) # Convert data frame to data table
head(resTestedDT)
revigo <- resTestedDT[, .(entrez, padj)] # Select only columns entrez & padj
write.csv(revigo, file="revigo.csv") # make table file of entrez & padj

# export top 100 DE genes to CSV file
resTestedDT_top100 <- resTestedDT[seq_len(100),]
write.csv(resTestedDT_top100, file="DESeq2_results.csv") #this table can be opened in Excel & pasted into DAVID or REVIGO

# construct binary variable for UP and DOWN regulated
genelistUp <- factor( as.integer( resTestedDT$padj < .1 & resTestedDT$log2FoldChange > 0 ) )
names(genelistUp) <- rownames(resTestedDT)
genelistDown <- factor( as.integer( resTestedDT$padj < .1 & resTestedDT$log2FoldChange < 0 ) )
names(genelistDown) <- rownames(resTestedDT)
```

## Getting Gene Symbols
Section needs adapting

```r
require("GO.db")
require("topGO")
require("biomaRt")
require("org.Hs.eg.db")

#How to get GeneSymbols
x                  <- org.Hs.eg.db
mapped_genes       <- mappedkeys(x)
GO2geneID          <- as.list(x[mapped_genes])
geneID2GO          <- as.list(org.Hs.eg.db)
goterms            <- Term(GOTERM)
geneNames          <- names(GO2geneID)
mart              <- biomaRt::useMart(biomart = "ENSEMBL_MART_ENSEMBL",dataset = "hsapiens_gene_ensembl",host = 'ensembl.org')
anno              <- biomaRt::getBM(attributes = c("ensembl_transcript_id", "ensembl_gene_id","external_gene_name","entrezgene"), mart = mart)
myGS              <- anno$external_gene_name
eid               <- anno$entrezgene
eid               <- eid[!is.na(eid)]
geneList          <- factor(as.numeric(geneNames%in%as.character(eid)))
names(geneList)   <- geneNames
mysampleGO        <- new("topGOdata",description = "Simple session", ontology = "BP",allGenes = geneList, geneSel = as.character(eid),nodeSize = 10,annot = annFUN.GO2genes,GO2gene=geneID2GO)


goID              <-"GO:0008088"
go.entrez         <- genesInTerm(mysampleGO, goID)
go.entrez         <- go.entrez[[1]]#To extract all genes related to this term
go.gs             <- unique(as.character(anno$external_gene_name)[which(anno$entrezgene%in%go.entrez)])
```



## Revigo: Filter out redundant terms 

Take resTestedDT datatable
Filter out the non-significant say padj<0.05
if there are more than 150 then select only the top 100.
Extract columns GO term & padj
Paste into Revigo box via Excel
To filter out redundant terms first run [Revigo](http://revigo.irb.hr/). 

Manually curate the list by looking at genes content in the GO as even with Revigo they can be very redundant.


## TopGO
Use the output of DSeq2: genelistUp & genelistDown
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

# Plotting GO p-values as Barplot
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

## DAVID
https://www.biostarhandbook.com/ontology/david-analysis.html
https://david.ncifcrf.gov/home.jsp

Step 1: Load a gene list into the [site](https://david.ncifcrf.gov/tools.jsp)
Using the output of DESeq2 resOrderedDT datatable can extract the Entrez terms.
Paste in only the significant events (padj < 0.05) or top 100 events

Step 2: Select Gene identifier
Entrez (or Ensembl depending on which column from the DT you extract)

Step 3: Clarify List Type
Specify Gene List as you only pasted in significant events (Background is if you paste in entire gene list)

Submit > Select input species (Homo sapiens)

Explore details through the Functional Annotation Tools:  table, chart & clustering reports

![enter image description here](https://www.biostarhandbook.com/ontology/images/summary.png)
Click Pathways > Chart in KEGG_PATHWAY row > note the Terms (these are the enriched GO terms) > click Genes bar to see individual genes

Download table as txt file > open in excel > copy the gene term & P-value columns > paste in to [revigo](http://revigo.irb.hr/) to further intepret 

Export & save results

# GOSeq

```r
### GO analysis
library(topGO)
library(GOstats)
library(goseq)
library(org.Mm.eg.db)
# subset results table to only genes with sufficient read coverage
resTested <- resLFC1[ !is.na(resLFC1$padj), ]
# remove decimal string & everything that follows from row.names (ENSEMBL) and call it "tmp": https://www.biostars.org/p/178726/ (alternatively use biomaRt )
temp=gsub("\\..*","",row.names(resTested))
#set the temp as the new rownames
rownames(resTested) <- temp
# construct binary variable for UP and DOWN regulated
genelistUp <- factor( as.integer( resTested$padj < .1 & resTested$log2FoldChange > 0 ) )
names(genelistUp) <- rownames(resTested)
genelistDown <- factor( as.integer( resTested$padj < .1 & resTested$log2FoldChange < 0 ) )
names(genelistDown) <- rownames(resTested)

### Test UPREGULATED GENES
#Test Biological Processes BP sub-ontology
myGOdata <- new( "topGOdata", ontology = "BP", allGenes = genelistUp, nodeSize = 10,
                 annot = annFUN.org, mapping = "org.Hs.eg.db", ID="Ensembl" )
goTestResults <- runTest( myGOdata, algorithm = "elim", statistic = "fisher" )
GenTable( myGOdata, goTestResults )
#Test Cellular Compartment (CC) sub-ontology
myGOdata <- new( "topGOdata", ontology = "CC", allGenes = genelistUp, nodeSize = 10,
                 annot = annFUN.org, mapping = "org.Hs.eg.db", ID="ensembl" )
goTestResults <- runTest( myGOdata, algorithm = "elim", statistic = "fisher" )
GenTable( myGOdata, goTestResults )
#Test Molecular function (MF) sub-ontology
myGOdata <- new( "topGOdata", ontology = "MF", allGenes = genelistUp, nodeSize = 10,
                 annot = annFUN.org, mapping = "org.Hs.eg.db", ID="ensembl" )
goTestResults <- runTest( myGOdata, algorithm = "elim", statistic = "fisher" )
GenTable( myGOdata, goTestResults )

### TEST DOWNREGULATED GENES
#Test Biological Processes BP sub-ontology
myGOdata <- new( "topGOdata", ontology = "BP", allGenes = genelistDown, nodeSize = 10,
                 annot = annFUN.org, mapping = "org.Hs.eg.db", ID="Ensembl" )
goTestResults <- runTest( myGOdata, algorithm = "elim", statistic = "fisher" )
UR_BP_table <- GenTable( myGOdata, goTestResults )
#Test Cellular Compartment (CC) sub-ontology
myGOdata <- new( "topGOdata", ontology = "CC", allGenes = genelistDown, nodeSize = 10,
                 annot = annFUN.org, mapping = "org.Hs.eg.db", ID="ensembl" )
goTestResults <- runTest( myGOdata, algorithm = "elim", statistic = "fisher" )
GenTable( myGOdata, goTestResults )
#Test Molecular function (MF) sub-ontology
myGOdata <- new( "topGOdata", ontology = "MF", allGenes = genelistDown, nodeSize = 10,
                 annot = annFUN.org, mapping = "org.Hs.eg.db", ID="ensembl" )
goTestResults <- runTest( myGOdata, algorithm = "elim", statistic = "fisher" )
GenTable( myGOdata, goTestResults )

### Plot GO p-values as as bar plot
library("GOplot")

barplot(UR_BP_table, drop=TRUE, font.size = 10,showCategory=10000, title="Up Regulated Biological Process")


UR_BP_table$result1
table(UR_BP_table$Term)

UR_BP_table$value <- (UR_BP_table$result1)+1

pvalue <- UR_BP_table$result1
na.omit(pvalue)
log10(pvalue)

barplot(table(UR_BP_table$result1), names.arg=Term, main="Up Regulated Biological Process")  


height <- log10(result1)


circ <- circle_dat(UR_BP_table, sorted.df)
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjEzMDkzMjY0MCwxMzE0Mjg3NDM3LDc1Mz
c3MTcxNCwyMDExMTIwNjg2LDExMzMzMTgzNTksNTgzMjE0NjEz
LDE2MTA1OTM1MTEsLTE1OTQ5ODczMjcsLTE0MjkwNTYyNzEsMT
M1ODc5OTI5NCwxNDA3ODAxMTg4LDc1MTMyODA0OCwxMDg4MTUy
MDc1LC0xMjI5MzU4NzY4LDIwOTkxOTc4MjYsMTI1MTk3MjYxOS
wtMjAwNTI4ODc5NywtODg4Mjg2MTE4LDEyMjU1NjE1NDgsMTEx
MzU4MDE2Ml19
-->