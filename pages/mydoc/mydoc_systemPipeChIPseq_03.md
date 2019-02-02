---
title: 3. Read preprocessing
last_updated: Sat Feb  2 11:43:49 2019
sidebar: mydoc_sidebar
permalink: mydoc_systemPipeChIPseq_03.html
---

## Experiment definition provided by `targets` file

The `targets` file defines all FASTQ files and sample comparisons of the analysis workflow.


```r
targetspath <- system.file("extdata", "targets_chip.txt", package = "systemPipeR")
targets <- read.delim(targetspath, comment.char = "#")
targets[1:4, -c(5, 6)]
```

```
##                   FileName SampleName Factor SampleLong
## 1 ./data/SRR446027_1.fastq        M1A     M1  Mock.1h.A
## 2 ./data/SRR446028_1.fastq        M1B     M1  Mock.1h.B
## 3 ./data/SRR446029_1.fastq        A1A     A1   Avr.1h.A
## 4 ./data/SRR446030_1.fastq        A1B     A1   Avr.1h.B
##   SampleReference
## 1                
## 2                
## 3             M1A
## 4             M1B
```

## Read quality filtering and trimming

The following example shows how one can design a custom read
preprocessing function using utilities provided by the `ShortRead` package, and then
apply it with `preprocessReads` in batch mode to all FASTQ samples referenced in the
corresponding `SYSargs` instance (`args` object below). More detailed information on
read preprocessing is provided in `systemPipeR's` main vignette.


```r
args <- systemArgs(sysma = "param/trim.param", mytargets = "targets_chip.txt")
filterFct <- function(fq, cutoff = 20, Nexceptions = 0) {
    qcount <- rowSums(as(quality(fq), "matrix") <= cutoff, na.rm = TRUE)
    fq[qcount <= Nexceptions]
    # Retains reads where Phred scores are >= cutoff with N
    # exceptions
}
preprocessReads(args = args, Fct = "filterFct(fq, cutoff=20, Nexceptions=0)", 
    batchsize = 1e+05)
writeTargetsout(x = args, file = "targets_chip_trim.txt", overwrite = TRUE)
```

## FASTQ quality report

The following `seeFastq` and `seeFastqPlot` functions generate and plot a series of useful quality statistics for a set of FASTQ files including per cycle quality box
plots, base proportions, base-level quality trends, relative k-mer
diversity, length and occurrence distribution of reads, number of reads
above quality cutoffs and mean quality distribution. The results are
written to a PDF file named `fastqReport.pdf`.


```r
args <- systemArgs(sysma = "param/tophat.param", mytargets = "targets_chip.txt")
library(BiocParallel)
library(batchtools)
f <- function(x) {
    library(systemPipeR)
    args <- systemArgs(sysma = "param/tophat.param", mytargets = "targets_chip.txt")
    seeFastq(fastq = infile1(args)[x], batchsize = 1e+05, klength = 8)
}
moduleload(modules(args))  # Skip if a module system is not used
resources <- list(walltime = 120, ntasks = 1, ncpus = cores(args), 
    memory = 1024)
param <- BatchtoolsParam(workers = 4, cluster = "slurm", template = "batchtools.slurm.tmpl", 
    resources = resources)
fqlist <- bplapply(seq(along = args), f, BPPARAM = param)

pdf("./results/fastqReport.pdf", height = 18, width = 4 * length(fqlist))
seeFastqPlot(unlist(fqlist, recursive = FALSE))
dev.off()
```

![](./pages/mydoc/systemPipeChIPseq_files/fastqReport.png)
<div align="center">Figure 1: FASTQ quality report for 18 samples</div>

<br><br><center><a href="mydoc_systemPipeChIPseq_02.html"><img src="images/left_arrow.png" alt="Previous page."></a>Previous Page &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Next Page
<a href="mydoc_systemPipeChIPseq_04.html"><img src="images/right_arrow.png" alt="Next page."></a></center>