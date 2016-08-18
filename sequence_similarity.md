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

Here's a simple Perl script to get kmers from a sequence:

```
$ cat -n kmer1.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	sub MAIN (Str $input!, Int :$k=10) {
     4 	    my $seq = $input.IO.f ?? $input.IO.slurp.chomp !! $input;
     5 	    my $n   = $seq.chars - $k + 1;
     6
     7 	    if $n < 1 {
     8 	        note "Cannot extract {$k}-mers from seq length {$seq.chars}";
     9 	        exit;
    10 	    }
    11
    12 	    for 0..^$n -> $i {
    13 	        put $seq.substr($i, $k);
    14 	    }
    15 	}
$ ./kmer1.pl6 -k=20 input.txt
AGCTTTTCATTCTGACTGCA
GCTTTTCATTCTGACTGCAA
CTTTTCATTCTGACTGCAAC
TTTTCATTCTGACTGCAACG
TTTCATTCTGACTGCAACGG
TTCATTCTGACTGCAACGGG
```

At line 5, we use a simple formula to determine the number of k-mers we can extract from the given sequence.  At lines 7-10, we decide if we can continue based on the input from the user.  Lines 12-14 should look somewhat familiar by now.  Since we're going to use the zero-based ```substr``` to extract part of the sequence, we need to use the "..^" range operator to go *up to but not including* our value for ```$n```.  After that, we just ```put``` the extracted k-mer.

Here's a slightly shorter version that again uses the ```map``` function (https://docs.perl6.org/routine/map) to eschew a ```for``` loop:

```
$ cat -n kmer2.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	sub MAIN (Str $input!, Int :$k=10) {
     4 	    my $seq = $input.IO.f ?? $input.IO.slurp.chomp !! $input;
     5 	    my $n   = $seq.chars - $k + 1;
     6
     7 	    if $n < 1 {
     8 	        note "Cannot extract {$k}-mers from seq length {$seq.chars}";
     9 	    }
    10 	    else {
    11 	        put join "\n", map { $seq.substr($_, $k) }, 0..^$n;
    12 	    }
    13 	}
```

It's worth spending time to really learn ```map``` as it's a very safe and efficient function.  It takes two arguments:

* a code block to run
* a list of things

The code block is applied to each element in the list coming in from the right and is emitted on the left:

```
new <- map { code } <- original
```

It's important to remember that a ```map``` will always return the same number of elements it was given as opposed to a ```grep``` which looks very similar:

```
> map *.uc, <foo bar baz>
(FOO BAR BAZ)
> grep /ba/, <foo bar baz>
(bar baz)
```

So the "list" argument to ```map``` in line 11 is the range ```0..^$n```.  Each number in turn goes into the code block as the "topic" or "it" ```$_``` where it is used in the ```substr``` call which is a k-mer.  All the k-mers are returned as a new list to the ```join``` which then puts a newline between them before the call to ```put```.



