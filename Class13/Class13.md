# Class13:DeSeq Lab
Raneem Kassar(PID:A17803411)

- [Background](#background)
- [Data Import](#data-import)
- [DESeq analysis](#deseq-analysis)
- [Volcano plot](#volcano-plot)
- [Save our results to date](#save-our-results-to-date)
- [Adding annotation data](#adding-annotation-data)
- [Pathway analysis](#pathway-analysis)
- [Save our annotated results](#save-our-annotated-results)

## Background

TOday we’re going to do an RNA-seq analysis of a data set on the common
glucocorticoid steroid dexamethasone (dex), and we’ll use DESeq for this
analysis.

## Data Import

Let’s read the `count` data and `metadata` about this experiment setup
from the supplied CSV files:

``` r
counts <- read.csv("airway_scaledcounts.csv", row.names = 1)
  metadata <- read.csv("airway_metadata.csv")
```

Have a wee peak:

``` r
head(counts)
```

                    SRR1039508 SRR1039509 SRR1039512 SRR1039513 SRR1039516
    ENSG00000000003        723        486        904        445       1170
    ENSG00000000005          0          0          0          0          0
    ENSG00000000419        467        523        616        371        582
    ENSG00000000457        347        258        364        237        318
    ENSG00000000460         96         81         73         66        118
    ENSG00000000938          0          0          1          0          2
                    SRR1039517 SRR1039520 SRR1039521
    ENSG00000000003       1097        806        604
    ENSG00000000005          0          0          0
    ENSG00000000419        781        417        509
    ENSG00000000457        447        330        324
    ENSG00000000460         94        102         74
    ENSG00000000938          0          0          0

and the metadata that tells us what is actually in the columns of our
`counts` object:

``` r
metadata
```

              id     dex celltype     geo_id
    1 SRR1039508 control   N61311 GSM1275862
    2 SRR1039509 treated   N61311 GSM1275863
    3 SRR1039512 control  N052611 GSM1275866
    4 SRR1039513 treated  N052611 GSM1275867
    5 SRR1039516 control  N080611 GSM1275870
    6 SRR1039517 treated  N080611 GSM1275871
    7 SRR1039520 control  N061011 GSM1275874
    8 SRR1039521 treated  N061011 GSM1275875

> Q1. How many genes are in this dataset?

There are 38694 genes in this dataset

> Q2. How many ‘control’ cell lines do we have?

``` r
sum(metadata$dex == "control")
```

    [1] 4

There are 4 control cell lines.

``` r
ncol(counts)
```

    [1] 8

> Q3. How would you make the above code in either approach more robust?
> Is there a function that could help here?

To make the code more robust, I would use the `match()` function to make
sure the metadata rows are ordered the same way as the count data
columns. This function helps prevent sample information from being
matched to the wrong column.

``` r
metadata <- metadata[match(colnames(counts), metadata$id), ]
```

``` r
all(colnames(counts) == metadata$id)
```

    [1] TRUE

- Find the “control” columns in our `counts` object
- Extract just the “control” column values for all genes
- Calculate the average value per gene in these “control” column

``` r
control.inds <- metadata$dex == "control"
control.counts <- counts[ , control.inds ]
control.mean <- rowMeans(control.counts)
```

Now do the same for the “treated” columns. Then make a plot of
`control.mean` vs `treated.mean`

> Q4. Follow the same procedure for the treated samples (i.e. calculate
> the mean per gene across drug treated samples and assign to a labeled
> vector called treated.mean)

``` r
treated.mean <- rowMeans( counts[, metadata$dex == "treated"])
```

For book-keeping let’s store these together as a new object called
`meancounts`

``` r
meancounts <- data.frame(control.mean, treated.mean)
head(meancounts)
```

                    control.mean treated.mean
    ENSG00000000003       900.75       658.00
    ENSG00000000005         0.00         0.00
    ENSG00000000419       520.50       546.00
    ENSG00000000457       339.75       316.50
    ENSG00000000460        97.25        78.75
    ENSG00000000938         0.75         0.00

> Q5 (a). Create a scatter plot showing the mean of the treated samples
> against the mean of the control samples. Your plot should look
> something like the following.

``` r
plot(meancounts)
```

![](Class13_files/figure-commonmark/unnamed-chunk-11-1.png)

We must often use log2 transform for this kind of data in bioinformatics
because it makes the interpretation much easier.

> Q5 (b).You could also use the ggplot2 package to make this figure
> producing the plot below. What geom\_?() function would you use for
> this plot?

``` r
library(ggplot2)

ggplot(meancounts) +
  aes(x = control.mean, y = treated.mean) +
  geom_point()
```

![](Class13_files/figure-commonmark/unnamed-chunk-12-1.png)

> Q6. Try plotting both axes on a log scale. What is the argument to
> plot() that allows you to do this?

The argument to plot() is log = “xy”, which puts both the x-axis and
y-axis on a log scale.

Our count data is highly skewed and when we see a pattern like this plot
it SCREAMS log transform me!

``` r
plot(meancounts$control.mean,
     meancounts$treated.mean,
     log = "xy",
     xlab = "Control mean",
     ylab = "Treated mean")
```

    Warning in xy.coords(x, y, xlabel, ylabel, log): 15032 x values <= 0 omitted
    from logarithmic plot

    Warning in xy.coords(x, y, xlabel, ylabel, log): 15281 y values <= 0 omitted
    from logarithmic plot

![](Class13_files/figure-commonmark/unnamed-chunk-13-1.png)

> Q7. What is the purpose of the `arr.ind` argument in the `which()`
> function call above? Why would we then take the first column of the
> output and need to call the `unique()` function?

The `arr.ind = TRUE` argument makes `which()` return the row and column
positions where the condition is TRUE. It correlates2 to identify which
genes and which samples have zero counts. Since we want to remove genes
with zero counts in any sample, we focus on the row numbers. We use
`unique()` so that each gene row is only counted once.

``` r
# Treated / Control

log2(20 / 20 )
```

    [1] 0

``` r
log2( 40/ 20)
```

    [1] 1

``` r
log2( 10 / 20 )
```

    [1] -1

``` r
log2( 80 / 20)
```

    [1] 2

We call this little fraction the **“log2 fold change”** as it tells us
how much or less gene expression we have in units of doubling, etc.

Let’s calculate the log2 fold change for our `treatment.mean` and
`control.mean` counts and call this `log2fc`

``` r
meancounts$log2fc <- log2(meancounts$treated.mean /
meancounts$control.mean)
head(meancounts)
```

                    control.mean treated.mean      log2fc
    ENSG00000000003       900.75       658.00 -0.45303916
    ENSG00000000005         0.00         0.00         NaN
    ENSG00000000419       520.50       546.00  0.06900279
    ENSG00000000457       339.75       316.50 -0.10226805
    ENSG00000000460        97.25        78.75 -0.30441833
    ENSG00000000938         0.75         0.00        -Inf

A common “rule of thumb” threshold for calling a gene “up regulated” or
“down regulated” is a log2 fold change value of +2 or -2 (or greater)

``` r
zero.vals <- which(meancounts[,1:2]==0, arr.ind=TRUE)
to.rm <- unique(zero.vals[,1])
mycounts <- meancounts[-to.rm,]
head(mycounts)
```

                    control.mean treated.mean      log2fc
    ENSG00000000003       900.75       658.00 -0.45303916
    ENSG00000000419       520.50       546.00  0.06900279
    ENSG00000000457       339.75       316.50 -0.10226805
    ENSG00000000460        97.25        78.75 -0.30441833
    ENSG00000000971      5219.00      6687.50  0.35769358
    ENSG00000001036      2327.00      1785.75 -0.38194109

``` r
up.ind <- mycounts$log2fc > 2
down.ind <- mycounts$log2fc < (-2)

paste("Up:", sum(up.ind, na.rm = TRUE))
```

    [1] "Up: 250"

``` r
paste("Down:", sum(down.ind, na.rm = TRUE))
```

    [1] "Down: 367"

> Q8. Using the up.ind vector above can you determine how many up
> regulated genes we have at the greater than 2 fc level?

There are 250 up-regulated genes with a log2 fold-change greater than 2.

> Q9. Using the down.ind vector above can you determine how many down
> regulated genes we have at the greater than 2 fc level?

There are 367 down-regulated genes with a log2 fold-change less than -2.

> Q10. Do you trust these results? Why or why not?

I do not fully trust these results because this analysis only uses fold
change as a gene can have a large fold change but still not be
statistically significant. Since we have not calculated p-values or
adjusted p-values yet, these results could be misleading.

## DESeq analysis

Let’s do this analysis properly and not forget about the significance of
the differences.

For this we will use **DESeq2** package.

``` r
library(DESeq2)
```

To run a DESeq analysis we need at least two inputs:

- `countData` i.e. are gene counts across different experiments
  -`colData` i.e. our metadata about those count columns

``` r
dds <- DESeqDataSetFromMatrix(countData = counts,
                              colData = metadata,
                              design = ~dex)
```

    converting counts to integer mode

Now we can run the DESeq analysis pipeline using this `dds` object that
has all the inputs we need.

``` r
dds <- DESeq(dds)
```

    estimating size factors

    estimating dispersions

    gene-wise dispersion estimates

    mean-dispersion relationship

    final dispersion estimates

    fitting model and testing

``` r
res <- results(dds)
head(res)
```

    log2 fold change (MLE): dex treated vs control 
    Wald test p-value: dex treated vs control 
    DataFrame with 6 rows and 6 columns
                      baseMean log2FoldChange     lfcSE      stat    pvalue
                     <numeric>      <numeric> <numeric> <numeric> <numeric>
    ENSG00000000003 747.194195      -0.350703  0.168242 -2.084514 0.0371134
    ENSG00000000005   0.000000             NA        NA        NA        NA
    ENSG00000000419 520.134160       0.206107  0.101042  2.039828 0.0413675
    ENSG00000000457 322.664844       0.024527  0.145134  0.168996 0.8658000
    ENSG00000000460  87.682625      -0.147143  0.256995 -0.572550 0.5669497
    ENSG00000000938   0.319167      -1.732289  3.493601 -0.495846 0.6200029
                         padj
                    <numeric>
    ENSG00000000003  0.163017
    ENSG00000000005        NA
    ENSG00000000419  0.175937
    ENSG00000000457  0.961682
    ENSG00000000460  0.815805
    ENSG00000000938        NA

## Volcano plot

This is a ubiquitous and common visualization for this type of data that
puts the log2 fold change and the adjusted p-value together in one plot
that people can get insight for what is going on in the whole dataset
results.

``` r
library(ggplot2)
```

``` r
ggplot(res) + 
  aes(log2FoldChange, padj) +
  geom_point(alpha=0.3)
```

    Warning: Removed 23549 rows containing missing values or values outside the scale range
    (`geom_point()`).

![](Class13_files/figure-commonmark/unnamed-chunk-22-1.png)

That plot is not very useful because we don’t care about genes with high
p-values, we want the very low values below our alpha threshold
(e.g. 0.01)

Let’s log the y-axis so we can see these genes/points more clearly.

``` r
ggplot(res) + 
  aes(log2FoldChange, log(padj)) +
  geom_point(alpha=0.3)
```

    Warning: Removed 23549 rows containing missing values or values outside the scale range
    (`geom_point()`).

![](Class13_files/figure-commonmark/unnamed-chunk-23-1.png)

We need to flip the y-axis so our “volcano” is not upside down

``` r
ggplot(res) + 
  aes(log2FoldChange, -log(padj)) +
  geom_point(alpha=0.3)
```

    Warning: Removed 23549 rows containing missing values or values outside the scale range
    (`geom_point()`).

![](Class13_files/figure-commonmark/unnamed-chunk-24-1.png)

> Q. Add annotation to this volcano plot including the log2 fold-change
> thresholds of +2 and -2 and the p-value threshold of 0.05. Also color
> up just the genes that meet both these thresholds. These are the ones
> we will focus on next day!

``` r
# Setup our custom point color vector 
mycols <- rep("gray", nrow(res))
mycols[ abs(res$log2FoldChange) > 2 ]  <- "red" 

inds <- (res$padj < 0.05) & (abs(res$log2FoldChange) > 2)
mycols[ inds ] <- "blue"

# Volcano plot with custom colors 
plot( res$log2FoldChange,  -log(res$padj), 
 col=mycols, ylab="-Log(P-value)", xlab="Log2(FoldChange)" )

# Cut-off lines
abline(v=c(-2,2), col="gray", lty=2)
abline(h = -log(0.05), col = "gray", lty = 2)
```

![](Class13_files/figure-commonmark/unnamed-chunk-25-1.png)

## Save our results to date

``` r
write.csv(res, file="myresults.csv")
```

## Adding annotation data

We need to “map” or “translate” our ENSEMBLE gene identifiers in our
results object to date to the identifiers used in different databases we
want to use for leaning more about these genes.

For this we will use a couple of BioConductor packages that we can
install with:
’BiocManager::install(“AnnotationDbi”)`and`BiocManager::install(“org.Hs.eg.db”)\`

``` r
library(AnnotationDbi)
library(org.Hs.eg.db)
```

We can see the columns in `org.Hs.eg.db` that list the different
databases we can map between:

``` r
columns(org.Hs.eg.db)
```

     [1] "ACCNUM"       "ALIAS"        "ENSEMBL"      "ENSEMBLPROT"  "ENSEMBLTRANS"
     [6] "ENTREZID"     "ENZYME"       "EVIDENCE"     "EVIDENCEALL"  "GENENAME"    
    [11] "GENETYPE"     "GO"           "GOALL"        "IPI"          "MAP"         
    [16] "OMIM"         "ONTOLOGY"     "ONTOLOGYALL"  "PATH"         "PFAM"        
    [21] "PMID"         "PROSITE"      "REFSEQ"       "SYMBOL"       "UCSCKG"      
    [26] "UNIPROT"     

> Q11. Run the mapIds() function two more times to add the Entrez ID and
> UniProt accession and GENENAME as new columns called
> res$entrez, res$uniprot and res\$genename.

We can now use the `mapIDs()` function to map between these different
database identifiers formats:

``` r
res$symbol <- mapIds(org.Hs.eg.db,
                keys=row.names(res),
              keytype="ENSEMBL",
               column="GENENAME")
```

    'select()' returned 1:many mapping between keys and columns

> Q. Can you map to “GENENAME” and add as a new “column” to our `res`
> obj

``` r
res$genename <- mapIds(org.Hs.eg.db,
                       keys = row.names(res),
                       keytype = "ENSEMBL",
                       column = "GENENAME",
                       multiVals = "first")
```

    'select()' returned 1:many mapping between keys and columns

> Q. Add “ENTREZID” as a `res$entrez`?

``` r
res$entrez <- mapIds(org.Hs.eg.db,
                     keys = row.names(res),
                     keytype = "ENSEMBL",
                     column = "ENTREZID",
                     multiVals = "first")
```

    'select()' returned 1:many mapping between keys and columns

``` r
res$uniprot <- mapIds(org.Hs.eg.db,
                      keys = row.names(res),
                      keytype = "ENSEMBL",
                      column = "UNIPROT",
                      multiVals = "first")
```

    'select()' returned 1:many mapping between keys and columns

``` r
head(res)
```

    log2 fold change (MLE): dex treated vs control 
    Wald test p-value: dex treated vs control 
    DataFrame with 6 rows and 10 columns
                      baseMean log2FoldChange     lfcSE      stat    pvalue
                     <numeric>      <numeric> <numeric> <numeric> <numeric>
    ENSG00000000003 747.194195      -0.350703  0.168242 -2.084514 0.0371134
    ENSG00000000005   0.000000             NA        NA        NA        NA
    ENSG00000000419 520.134160       0.206107  0.101042  2.039828 0.0413675
    ENSG00000000457 322.664844       0.024527  0.145134  0.168996 0.8658000
    ENSG00000000460  87.682625      -0.147143  0.256995 -0.572550 0.5669497
    ENSG00000000938   0.319167      -1.732289  3.493601 -0.495846 0.6200029
                         padj                 symbol               genename
                    <numeric>            <character>            <character>
    ENSG00000000003  0.163017          tetraspanin 6          tetraspanin 6
    ENSG00000000005        NA            tenomodulin            tenomodulin
    ENSG00000000419  0.175937 dolichyl-phosphate m.. dolichyl-phosphate m..
    ENSG00000000457  0.961682 SCY1 like pseudokina.. SCY1 like pseudokina..
    ENSG00000000460  0.815805 FIGNL1 interacting r.. FIGNL1 interacting r..
    ENSG00000000938        NA FGR proto-oncogene, .. FGR proto-oncogene, ..
                         entrez     uniprot
                    <character> <character>
    ENSG00000000003        7105  A0A087WYV6
    ENSG00000000005       64102      Q9H2S6
    ENSG00000000419        8813      H0Y368
    ENSG00000000457       57147      X6RHX1
    ENSG00000000460       55732      A6NFP1
    ENSG00000000938        2268      B7Z6W7

## Pathway analysis

Now we have our annotated results with their log2 fold-change and
p-values we can figure out which biological pathways and process these
genes are involved with.

We will use the **gage** and **pathview** packages for this step and we
can install them with:
`BiocManager::install( c("pathview", "gage", "gageData") )`

``` r
library(gage)
library(gageData)
library(pathview)
```

Let’s have a wee peak at gageData

``` r
data(kegg.sets.hs)
# Examine the first 2 pathways in this kegg set for humans
head(kegg.sets.hs, 2)
```

    $`hsa00232 Caffeine metabolism`
    [1] "10"   "1544" "1548" "1549" "1553" "7498" "9"   

    $`hsa00983 Drug metabolism - other enzymes`
     [1] "10"     "1066"   "10720"  "10941"  "151531" "1548"   "1549"   "1551"  
     [9] "1553"   "1576"   "1577"   "1806"   "1807"   "1890"   "221223" "2990"  
    [17] "3251"   "3614"   "3615"   "3704"   "51733"  "54490"  "54575"  "54576" 
    [25] "54577"  "54578"  "54579"  "54600"  "54657"  "54658"  "54659"  "54963" 
    [33] "574537" "64816"  "7083"   "7084"   "7172"   "7363"   "7364"   "7365"  
    [41] "7366"   "7367"   "7371"   "7372"   "7378"   "7498"   "79799"  "83549" 
    [49] "8824"   "8833"   "9"      "978"   

We need a vector of importance (e.g. fold-change values) that has gene
ids as names. These names need to be in the correct format (using the
correct database format for the IDs)

``` r
x <- c(10, 9, 7)
names(x) <- c("alice", "chandea", "barry")
x
```

      alice chandea   barry 
         10       9       7 

``` r
names(x)
```

    [1] "alice"   "chandea" "barry"  

Here we will make a wee input vector called `foldchanges` that has
“entrez” ids as names.

``` r
foldchanges <- res$log2FoldChange
names(foldchanges) <- res$entrez
```

Now we can run `gage()` to do our pathway analysis.

``` r
keggres = gage(foldchanges, gsets=kegg.sets.hs)
```

``` r
attributes(keggres)
```

    $names
    [1] "greater" "less"    "stats"  

``` r
head(keggres$less, 3)
```

                                          p.geomean stat.mean        p.val
    hsa05332 Graft-versus-host disease 0.0004250607 -3.473335 0.0004250607
    hsa04940 Type I diabetes mellitus  0.0017820379 -3.002350 0.0017820379
    hsa05310 Asthma                    0.0020046180 -3.009045 0.0020046180
                                            q.val set.size         exp1
    hsa05332 Graft-versus-host disease 0.09053792       40 0.0004250607
    hsa04940 Type I diabetes mellitus  0.14232788       42 0.0017820379
    hsa05310 Asthma                    0.14232788       29 0.0020046180

> Q12. Can you do the same procedure as above to plot the pathview
> figures for the top 2 down-reguled pathways?

``` r
# Get the top 2 downregulated pathway IDs
down.pathways <- rownames(keggres$less)[1:2]

# Extract the KEGG pathway IDs
down.ids <- substr(down.pathways, start = 1, stop = 8)

down.ids
```

    [1] "hsa05332" "hsa04940"

``` r
# Plot the top 2 downregulated pathways
pathview(gene.data = foldchanges,
         pathway.id = down.ids,
         species = "hsa")
```

    'select()' returned 1:1 mapping between keys and columns

    Info: Working in directory /Users/raneemkassar/BIMM143/Class15/bimm143_github/Class13

    Info: Writing image file hsa05332.pathview.png

    'select()' returned 1:1 mapping between keys and columns

    Info: Working in directory /Users/raneemkassar/BIMM143/Class15/bimm143_github/Class13

    Info: Writing image file hsa04940.pathview.png

``` r
knitr::include_graphics("hsa05332.pathview.png")
```

![](hsa05332.pathview.png)

``` r
knitr::include_graphics("hsa04940.pathview.png")
```

![](hsa04940.pathview.png)

![](hsa04940.png)

The code extracts the top 2 down-regulated KEGG pathway IDs from
`keggres$less` and uses `pathview()` to generate pathway figures for the
pathways.

Now we can use the**pathview** package with the found KEGG pathways IDs
(e.g. “hsa05310” for the Asthma pathway) to make a pathway figure
showing our Differential expressed genes (DEGs)

``` r
pathview(gene.data=foldchanges, pathway.id="hsa05310")
```

    Info: Downloading xml files for hsa05310, 1/1 pathways..

    Info: Downloading png files for hsa05310, 1/1 pathways..

    'select()' returned 1:1 mapping between keys and columns

    Info: Working in directory /Users/raneemkassar/BIMM143/Class15/bimm143_github/Class13

    Info: Writing image file hsa05310.pathview.png

![](hsa05310.png)

## Save our annotated results

``` r
write.csv(res, file="myresults_annotated.csv")
```
