# Class 06: R functions
Raneem Kassar (PID: A17803411)

- [Background](#background)
- [A first function](#a-first-function)
- [A generate_dna() function](#a-generate_dna-function)
- [Generate random protein sequences of length 6 to
  13](#generate-random-protein-sequences-of-length-6-to-13)
- [Are Our peptides “unique in
  nature”?](#are-our-peptides-unique-in-nature)
- [Connecting your findings to
  immunology](#connecting-your-findings-to-immunology)

## Background

All functions in R have at least 3 things

- a *name* (we pick that and use it to call the function)
- input *arguments* (one or more comma seperated inputs that go inside
  the brackets when we call the function)
- the *body* (the lines of R code that do the work of the function).

## A first function

Here we will create a function to add some numbers. Let’s call it
‘add()’

Input arguments can be either **“required”** or **“optional”**, the
later have fall-back default values that will be used if the user does
not specify them.

> **Q1a:** Your first version of the function should add two input
> numbers together. For example, add(4,7) should return 11. \[1 pt\]

``` r
add <- function(x, y=0) {
  x + y
}
```

Can we use our new function:

``` r
add(10, 100)
```

    [1] 110

``` r
add(10)
```

    [1] 10

> **Q1b:** For you second version, adapt your first function so it can
> take a single input vector or two inputs as before. For example,
> add(4, 7) and add( c(4,7,10) ). \[1 pt\]

``` r
add <- function(x, y=0) {
  sum(x, y)
}
```

``` r
add(4, 7)
```

    [1] 11

``` r
add( c(4,7,10), y= )
```

    [1] 21

> **Q1c.** c. Finally, on your own (outside of class) create a third
> version of your function that can add any number of inputs that the
> user provides. For example, add(1, 2, 3, -4)

``` r
add <- function(...) {
  sum(...)
}
```

``` r
add(10,100)
```

    [1] 110

``` r
add(1, 2, 3, -4)
```

    [1] 2

We can explicitly set a **return** value output from a function (rather
than the default last line) by using the ‘return()’ function call.

## A generate_dna() function

A useful function here is the “base R” ‘sample(’ function:)

``` r
sample(1:5, size=3)
```

    [1] 4 3 2

``` r
sample(1:5, size=60, replace = TRUE)
```

     [1] 5 3 5 1 2 4 2 3 5 3 1 3 5 1 2 2 1 1 5 2 4 4 2 3 2 1 5 4 1 5 3 3 5 5 5 1 3 2
    [39] 4 3 5 3 3 4 2 4 4 4 5 2 2 5 3 1 1 5 1 3 4 1

We can use this to make a random nucleotide generator if we draw from
“A”, “C”, “G” and “T”…

``` r
sample(x=c("A", "C", "G", "T"), size = 10, replace = TRUE)
```

     [1] "C" "G" "T" "A" "G" "T" "A" "C" "T" "T"

> **Q2.** Write a function generate_dna() that returns a random DNA
> sequence of a length specified by the user. Your first version should
> return a multi-element vector of single character nucleotides. For
> example generate_dna(6) might return “A”, “T”, “T”, “G”, “A”, “C”. \[1
> pt\]

``` r
generate_dna <- function(len) {
  sample(c("A", "C", "G", "T"), size = len, replace = TRUE)
}
```

``` r
generate_dna(6)
```

    [1] "A" "T" "T" "A" "T" "A"

``` r
generate_dna(len=100)
```

      [1] "T" "G" "T" "G" "A" "C" "C" "C" "T" "A" "C" "A" "A" "A" "G" "A" "G" "C"
     [19] "A" "C" "G" "C" "G" "C" "G" "G" "C" "C" "T" "A" "C" "A" "T" "A" "A" "G"
     [37] "A" "G" "A" "G" "A" "G" "C" "C" "C" "G" "A" "G" "A" "A" "G" "G" "G" "T"
     [55] "C" "T" "G" "A" "G" "G" "G" "A" "G" "T" "G" "G" "A" "G" "A" "C" "C" "C"
     [73] "T" "G" "T" "A" "G" "A" "A" "G" "A" "C" "C" "T" "C" "G" "G" "C" "A" "A"
     [91] "A" "G" "C" "C" "T" "G" "A" "T" "G" "A"

> **Q2b:** Your second version should *optionally* be able to return
> either a multi-element vector of single character nucleotides (as
> before) or a **single character string** (not a vector of single
> letters but a singe vector of multiple letters). For example
> “AAGGCTG”. \[1 pt\]

``` r
generate_dna <- function(len=10, single.element=TRUE) {
  
  ans <- sample(x=c("A", "C", "G", "T"), size=len, replace = TRUE)
    #cat("Randomly Select from DNA nucleotides") 
  
  if(single.element) {
    ##cat("Combine the bases into one single DNA string")
   ans <-  paste( ans, collapse = "")
  }
  
  ## ("Return DNA Sequence")
  
  return(ans)
}
```

Functions that could be useful here are `paste()`, `if()`, `cat()` and
`return()`

``` r
generate_dna()
```

    [1] "AAACTCCTTA"

``` r
generate_dna(single.element = FALSE)
```

     [1] "C" "T" "T" "T" "C" "T" "T" "G" "C" "T"

> **Q2c.** Finally, create a final version of your function that prints
> out a FASTA format sequence with an id line indicating the sequence
> length.

    >len9
    CGAAGGCTG

``` r
cat("hello \n there")
```

    hello 
     there

``` r
generate_dna <- function(len=10, single.element=TRUE) {
  
  ans <- sample(x=c("A", "C", "G", "T"), size=len, replace = TRUE)
  #cat("Randomly select nucleotide sequence")
  
  if(single.element) {
    #cat("Combine bases into one single DNA sequence")
   ans <-  paste( ans, collapse = "")
  }
  ## Format as FASTA with sequence length
  cat(paste(">len", len, "\n", sep="") )
  cat(ans)
  cat("\n")
  #("Print the DNA sequence on the next line")
  
  return(ans)
}
```

``` r
x <- generate_dna(20)
```

    >len20
    GCATGTAGAGTTCAGCGGAT

\##Write a `generate_protein()` function

> **Q3.** Write a function generate_protein() that returns a random
> peptide/protein sequence of a length specified by the user. For
> example `generate_protein(6)` might return`"WQRTAG"`. Your function
> should:

``` r
generate_protein <- function(len) {
  aa <- c("A", "R", "N", "D", "C", "E", "Q", "G", "H", "I",
          "L", "K", "M", "F", "P", "S", "T", "W", "Y", "V")
  ans <- sample(aa, len, replace = TRUE)
  ans <- paste(ans, collapse = "")
  return(ans)
}
```

``` r
generate_protein(6)
```

    [1] "WSFKKH"

## Generate random protein sequences of length 6 to 13

> **Q4.:** dapt and use your `generate_protein()` function to generate a
> series of random protein sequences ranging from 6 to 13 amino acids in
> length (one sequence per length). Take advantage of the base R
> function `for()` or `sapply()` so that you do not have to call
> generate_protein() eight times by hand.

``` r
for(l in 6:13) {
  cat(">", l, "\n", sep= "")
  cat( generate_protein(l), "\n")
}
```

    >6
    LSACFG 
    >7
    DGGGEQR 
    >8
    TRVYKYTI 
    >9
    DWYFQWQCS 
    >10
    DICENQYHDC 
    >11
    NNIQILLQVVV 
    >12
    EMSDYGEMVMMS 
    >13
    LRKLPVLIECFNH 

## Are Our peptides “unique in nature”?

> **Q5.**: Take your FASTA-formatted peptides from Q4 and run them as a
> single BLASTp search against the Non-redundant protein sequences (nr)
> database at https://blast.ncbi.nlm.nih.gov/. For this question do not
> restrict the organism (leave the Organism field blank so that the full
> nr database is searched)

| length | Ide  | Cov  | Unique |
|--------|------|------|--------|
| 6      | 100% | 100% | N      |
| 7      | 100% | 100% | N      |
| 8      | 100% | 100% | N      |
| 9      | 100% | 89%  | Y      |
| 10     | 90%  | 100% | Y      |
| 11     | 100% | 82%  | Y      |
| 12     | 100% | 75%  | Y      |
| 13     | 75%  | 92%  | Y      |

> **Q5a**: a. At which sequence length do your randomly generated
> peptides start to look “unique in nature” (i.e. no 100% coverage +
> 100% identity hit)? \[1 pt\]

At sequence 9, randomly generated peptides start to look “unique in
nature”.

> **Q5b**: Speculate why very short random peptides are almost always
> found in nr, while longer ones typically are not.

Very short random peptides are almost always found in nr because their
sequence space is much smaller, so there are a limited number of
combinations and many of them exist in the database. The total number of
possible peptide sequences is 20^L, where L is the peptide length, so
shorter peptides have far fewer possible sequences than longer ones.
Longer ones are typically not found because they have many more possible
sequence combinations, so a random long peptide is less likely to exist.
Finally, short sequences can match by chance more easily and longer
sequences require more exact similarity

## Connecting your findings to immunology

> **Q6.**: In 3–6 sentences total and using your Q5 data and the
> reasoning from Q5b, what do you think this minimum length is and why
> might it be a bad design choice for the immune system to present very
> short peptides? \[3 pt\]

Based on my Q5 data and the reasoning from Q5b, the minimum length is
around 9 amino acids because this is usually where random peptides begin
to look more “unique in nature” rather than matching more sequences in
the nr database by chance. Very short peptides would not be a good
design choice as they are way too common and more likely to resemble
unrelated proteins which include self proteins. MHC class II present
very short peptides, and CD4+ T cells could be activated by peptides
that are not specific to a pathogen, reducing the immune system’s
ability to distinguish self. Also, this would increase the risk of
nonspecific immune responses which would harm the immunity of a system.
