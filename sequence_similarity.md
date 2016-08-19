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
     4 	    die "k ($k) must be positive)" if $k < 0;
     5 	    my $seq = $input.IO.f ?? $input.IO.slurp.chomp !! $input;
     6 	    my $n   = $seq.chars - $k + 1;
     7
     8 	    if $n < 1 {
     9 	        note "Cannot extract {$k}-mers from seq length {$seq.chars}";
    10 	        exit;
    11 	    }
    12
    13 	    for 0..^$n -> $i {
    14 	        put $seq.substr($i, $k);
    15 	    }
    16 	}
$ ./kmer1.pl6 -k=20 input.txt
AGCTTTTCATTCTGACTGCA
GCTTTTCATTCTGACTGCAA
CTTTTCATTCTGACTGCAAC
TTTTCATTCTGACTGCAACG
TTTCATTCTGACTGCAACGG
TTCATTCTGACTGCAACGGG
```

At line 4, , we use a simple formula to determine the number of k-mers we can extract from the given sequence.  At lines 7-10, we decide if we can continue based on the input from the user.  Lines 12-14 should look somewhat familiar by now.  Since we're going to use the zero-based ```substr``` to extract part of the sequence, we need to use the "..^" range operator to go *up to but not including* our value for ```$n```.  After that, we just ```put``` the extracted k-mer.

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

# Kmers from FASTA

Now let's combine our parsing of FASTA with extraction of k-mers:

```
$ cat -n fasta-kmer1.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	use Bio::SeqIO;
     4
     5 	sub MAIN (Str $file!, Int :$k=10) {
     6 	    die "Not a file ($file)" unless $file.IO.f;
     7
     8 	    my $seqIO = Bio::SeqIO.new(format => 'fasta', file => $file);
     9
    10 	    while (my $seq = $seqIO.next-Seq) {
    11 	        put join "\n", get-kmers($k, $seq.seq);
    12 	    }
    13 	}
    14
    15 	sub get-kmers(Int $k, Str $str) {
    16 	    my $n = $str.chars - $k + 1;
    17 	    map { $str.substr($_, $k) }, 0..^$n;
    18 	}
```

I copied the FASTA script from the previous chapter and introduced a function called ```get-kmers```.  You've already been writing functions, of course, with ```MAIN``` and calling functions like ```join``` and ```put```, but now we're calling a function we've written.  We define that it takes two positional arguments, an integer ```$k``` and a string ```$string```.  Right away the type checking caught an error as I tried to pass in the ```$seq``` *object* and not the ```$seq.seq``` *string of the sequence.*  

Just like with our ```MAIN```, we can change our positional arguments into named arguments by placing a ```:``` before the names (line 15):

```
$ cat -n fasta-kmer2.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	use Bio::SeqIO;
     4
     5 	sub MAIN (Str $file!, Int :$k=10) {
     6 	    die "Not a file ($file)" unless $file.IO.f;
     7
     8 	    my $seqIO = Bio::SeqIO.new(format => 'fasta', file => $file);
     9
    10 	    while (my $seq = $seqIO.next-Seq) {
    11 	        put join "\n", get-kmers(str => $seq.seq, k => $k);
    12 	    }
    13 	}
    14
    15 	sub get-kmers(Int :$k, Str :$str) {
    16 	    my $n = $str.chars - $k + 1;
    17 	    map { $str.substr($_, $k) }, 0..^$n;
    18 	}
```

This means we have to call ```get-kmers``` with Pairs for arguments (line 11), but it also means the arguments can be specified in any order.  Generally, if a function takes more than 2 or 3 arguments, I would recommend using named arguments.

Here is another version that again dips into the function programming world and uses a new string function called ```rotor``` (http://perl6.party/post/Perl-6-.rotor-The-King-of-List-Manipulation):

```
$ cat fasta-kmer3.pl6
#!/usr/bin/env perl6

use Bio::SeqIO;

sub MAIN (Str $file!, Int :$k=10) {
    die "Not a file ($file)" unless $file.IO.f;

    my $seqIO = Bio::SeqIO.new(format => 'fasta', file => $file);

    my $j = -1 * ($k - 1);
    while (my $seq = $seqIO.next-Seq) {
        put $seq.seq.comb.rotor($k => $j).map(*.join).join("\n");
    }
}
```

It's probably easiest to understand this by going into the REPL:

```
> my $s = "ACGTACGT";
ACGTACGT
> $s.comb
(A C G T A C G T)
> $s.comb.rotor: 3
((A C G) (T A C))
> $s.comb.rotor: 3 => -2
((A C G) (C G T) (G T A) (T A C) (A C G) (C G T))
> $s.comb.rotor(3 => -2).map(*.join)
(ACG CGT GTA TAC ACG CGT)
> $s.comb.rotor(3 => -2).map(*.join).join("\n")
ACG
CGT
GTA
TAC
ACG
CGT
```

We've seen before how we can use ```comb``` to explode a string into a list of characters.  Next we call ```rotor``` to break the string into sublists of 3 elements each, but for k-mers we want ```rotor``` to back up two (*k* - 1) steps before making the next list, so we pass the Pair (3 => -2).  We then pass each sublist into a ```map``` where we use the ```*``` to mean "whatever" or "the thing" and call the ```join``` method (with no argument) to create a string.  That list of strings then gets ```join```ed on newlines to print all the k-mers in the sequence.