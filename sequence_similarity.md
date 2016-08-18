# Sequence similarity

As our field deals with the sequences of unknown communities of organisms, we are very interested in comparing our samples to known sequences.  There are many programs and algorithms for comparing strings, from the venerable BLAST (Basic Local Alignment Search Tool) to hashing to kmers.  

# K-mers

A "k-mer" is a *k*-length *mer* (from Greek *meros* "part") of contiguous sequence as in the word "polymer."  So the given sequence has 3-mers like so:

```
AGCTTTTC
AGC
 GCT
  CTT
   TTT
    TTT
     TTC
```

