# Problem Set

# Clean sequences

Write a bash script to clean up some raw sequence.

# Create a FASTA file

Given an input file of sequences, one-per-line, create a FASTA-formatted file.  Extra credit: block the sequences to a maximum column width, default 50.

# FASTA stats

Given one or more input files of sequences in FASTA format, recreate this output:

```
$ seqmagick info mouse.fa | awk '{print $1,$3,$4,$5,$6}' | column -t
name      min_len  max_len  avg_len  num_seqs
mouse.fa  50       100      84.32    500
```

# Compute GC content

Solve the GC content problem on Rosalind (http://rosalind.info/problems/gc/).  Then use that program to profile FASTA files and predict species.

# 3. Find motifs

Solve http://rosalind.info/problems/subs/ to find motifs in strings.  Use this to find ORFs.

# 4. Compute Hamming

Solve http://rosalind.info/problems/hamm/ to find the number of SNP/SNVs in two sequences.  Use this to determine sequence similarity.

# 5. Protein translation

Solve http://rosalind.info/problems/prot/.  Read the translation table from this:

```
AAA	K
AAC	N
AAG	K
AAU	N
ACA	T
ACC	T
ACG	T
ACU	T
AGA	R
AGC	S
AGG	R
AGU	S
AUA	I
AUC	I
AUG	M
AUU	I
CAA	Q
CAC	H
CAG	Q
CAU	H
CCA	P
CCC	P
CCG	P
CCU	P
CGA	R
CGC	R
CGG	R
CGU	R
CUA	L
CUC	L
CUG	L
CUU	L
GAA	E
GAC	D
GAG	E
GAU	D
GCA	A
GCC	A
GCG	A
GCU	A
GGA	G
GGC	G
GGG	G
GGU	G
GUA	V
GUC	V
GUG	V
GUU	V
UAA	Stop
UAC	Y
UAG	Stop
UAU	Y
UCA	S
UCC	S
UCG	S
UCU	S
UGA	Stop
UGC	C
UGG	W
UGU	C
UUA	L
UUC	F
UUG	L
UUU	F
```

# Shared k-mers

Create a program that will find the number of shared k-mers of a given size among a set of sequences in a FASTA file.  Use this to determine sequence similarity.