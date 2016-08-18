# Problem Set

Be sure to check out https://github.com:kyclark/abe487.git for the tests!

# Hello!

Alter the greeting script to accept the flag "--excited" that adds an exclamation point to the end of the greeting:

```
$ ./greet.pl6 --excited --greeting="Hola" --name="Amigo"
Hola, Amigo!
```

# Clean sequences

Write a bash script to clean up some raw sequence.

# Create a FASTA file

Create a script called "txt2fasta.pl6" that accepts an input file of sequences, one-per-line, and emits FASTA-formatted sequences.  Sequence IDs should be an incrementing integer value starting at 1.  Extra credit: block the sequences to a maximum column width, default 50.

# FASTA stats

Given one or more input files of sequences in FASTA format, recreate this output:

```
$ seqmagick info mouse.fa | awk '{print $1,$3,$4,$5,$6}' | column -t
name      min_len  max_len  avg_len  num_seqs
mouse.fa  50       100      84.32    500
```

# Compute GC content

Solve the GC content problem on Rosalind (http://rosalind.info/problems/gc/).  Then use that program to profile FASTA files and predict species.

# Find motifs

Solve http://rosalind.info/problems/subs/ to find motifs in strings.  Use this to find ORFs.

# Compute Hamming

Solve http://rosalind.info/problems/hamm/ to find the number of SNP/SNVs in two sequences.  Use this to determine sequence similarity.

# Protein translation

Solve http://rosalind.info/problems/prot/.  Read the translation table from the given "table.txt."

# Shared k-mers

Create a program that will find the number of shared k-mers of a given size among a set of sequences in a FASTA file.  Use this to determine sequence similarity.