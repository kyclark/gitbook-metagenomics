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
     4 	    if $k < 0 {
     5 	        note "k ($k) must be positive\n";
     6 	        exit 1;
     7 	    }
     8
     9 	    my $seq = $input.IO.f ?? $input.IO.slurp.chomp !! $input;
    10 	    my $n   = $seq.chars - $k + 1;
    11
    12 	    if $n < 1 {
    13 	        note "Cannot extract {$k}-mers from seq length {$seq.chars}";
    14 	        exit 1;
    15 	    }
    16
    17 	    for 0..^$n -> $i {
    18 	        put $seq.substr($i, $k);
    19 	    }
    20 	}
```

In lines 4-7, we first want to ensure that we have a positive value for *k*, so we need to explore this idea briefly.  Another way to write this would be to move that condition into the signature:

```
$ cat -n pos1.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	sub MAIN (Int :$k where * > 0 = 10) {
     4 	    put "k = $k";
     5 	}
$ ./pos1.pl6 -k=5
k = 5
$ ./pos1.pl6 -k=-1
Usage:
  ./pos1.pl6 [-k=<Int>]
```

Because that sort of thing is so useful, Perl let's us create our own types as a ```subset``` of an existing type.  I'll arbitrarily limit *k* to a positive integer less than 50:

```
$ cat -n pos2.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	subset SmallPosInt of Int where 0 < * < 50 ;
     4 	sub MAIN (SmallPosInt :$k=10) {
     5 	    put "k = $k";
     6 	}
$ ./pos2.pl6 -k=5
k = 5
$ ./pos2.pl6 -k=-1
Usage:
  ./pos2.pl6 [-k=<Int>]
$ ./pos2.pl6 -k=51
Usage:
  ./pos2.pl6 [-k=<Int>]
  ```

As it happens, there is a built-in type that will constrain us to a positive integer called ```UInt``` for "unsigned integer":

```
$ cat -n pos3.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	sub MAIN (UInt :$k=10) {
     4 	    put "k = $k";
     5 	}
$ ./pos3.pl6 -k=5
k = 5
$ ./pos3.pl6 -k=-1
Usage:
  ./pos3.pl6 [-k=<Int>]
```

Now we can shorten our code and get type checking for free:

```
$ cat -n kmer2.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	sub MAIN (Str $input!, UInt :$k=10) {
     4 	    my $seq = $input.IO.f ?? $input.IO.slurp.chomp !! $input;
     5 	    my $n   = $seq.chars - $k + 1;
     6
     7 	    if $n < 1 {
     8 	        note "Cannot extract {$k}-mers from seq length {$seq.chars}";
     9 	        exit 1;
    10 	    }
    11
    12 	    for 0..^$n -> $i {
    13 	        put $seq.substr($i, $k);
    14 	    }
    15 	}
$ ./kmer2.pl6 -k=20 input.txt
AGCTTTTCATTCTGACTGCA
GCTTTTCATTCTGACTGCAA
CTTTTCATTCTGACTGCAAC
TTTTCATTCTGACTGCAACG
TTTCATTCTGACTGCAACGG
TTCATTCTGACTGCAACGGG
```

At line 5, use a simple formula to determine the number of k-mers we can extract from the given sequence.  At lines 7-10, we decide if we can continue based on the input from the user.  Lines 12-14 should look somewhat familiar by now.  Since we're going to use the zero-based ```substr``` to extract part of the sequence, we need to use the "..^" range operator to go *up to but not including* our value for ```$n```.  After that, we just ```put``` the extracted k-mer.

Here's a slightly shorter version that again uses the ```map``` function (https://docs.perl6.org/routine/map) to eschew a ```for``` loop:

```
$ cat -n kmer3.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	sub MAIN (Str $input!, UInt :$k=10) {
     4 	    my $seq = $input.IO.f ?? $input.IO.slurp.chomp !! $input;
     5 	    my $n   = $seq.chars - $k + 1;
     6
     7 	    if $n < 1 {
     8 	        note "Cannot extract {$k}-mers from seq length {$seq.chars}";
     9 	        exit 1;
    10 	    }
    11
    12 	    put join "\n", map { $seq.substr($_, $k) }, 0..^$n;
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

So the "list" argument to ```map``` in line 12 is the range ```0..^$n```.  Each number in turn goes into the code block as the "topic" ```$_``` where it is used in the ```substr``` call to extract the k-mer which are then returned as a new list to the ```join``` which then puts a newline between them before the call to ```put```.

I'll present one more version that puts the kmers into an array using ```gather/take``` (https://docs.perl6.org/language/control#gather/take):

```
$ cat -n kmer4.pl6
     1	#!/usr/bin/env perl6
     2
     3	sub MAIN (Str $input!, Int :$k=10) {
     4	    my $seq   = $input.IO.f ?? $input.IO.slurp.chomp !! $input;
     5	    my $n     = $seq.chars - $k + 1;
     6	    my @kmers = gather for 0..^$n -> $i {
     7	        take $seq.substr($i, $k);
     8	    }
     9
    10	    dd @kmers;
    11	}
```

The ```dd``` is the built-in "data dumper" that shows you a textual representation of a data structure:

```
$ ./kmer4.pl6 input.txt
Array @kmers = ["AGCTTTTCAT", "GCTTTTCATT",
"CTTTTCATTC", "TTTTCATTCT", "TTTCATTCTG",
"TTCATTCTGA", "TCATTCTGAC", "CATTCTGACT",
"ATTCTGACTG", "TTCTGACTGC", "TCTGACTGCA",
"CTGACTGCAA", "TGACTGCAAC", "GACTGCAACG",
"ACTGCAACGG", "CTGCAACGGG"]
```

# Kmers from FASTA

Now let's combine our parsing of FASTA with extraction of k-mers:

```
$ cat -n fasta-kmer1.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	use Bio::SeqIO;
     4
     5 	sub MAIN (Str $file!, UInt :$k=10) {
     6 	    die "Not a file ($file)" unless $file.IO.f;
     7 	    my $seqIO = Bio::SeqIO.new(format => 'fasta', file => $file);
     8
     9 	    while (my $seq = $seqIO.next-Seq) {
    10 	        put join "\n", get-kmers($k, $seq.seq);
    11 	    }
    12 	}
    13
    14 	sub get-kmers(Int $k, Str $str) {
    15 	    my $n = $str.chars - $k + 1;
    16 	    map { $str.substr($_, $k) }, 0..^$n;
    17 	}
```

I copied the FASTA script from the previous chapter and introduced a function called ```get-kmers```.  You've already been writing functions, of course, like ```MAIN``` and calling functions like ```join``` and ```put```, but now we're calling a function we've written.  We define that it takes two positional arguments, an integer ```$k``` and a string ```$string```.  Right away the type checking caught an error as I tried to pass in the ```$seq``` *object* and not the ```$seq.seq``` *string of the sequence.*  

Just like with our ```MAIN```, we can change our positional arguments into named arguments by placing a ```:``` before the names (line 14 below):

```
$ cat -n fasta-kmer2.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	use Bio::SeqIO;
     4
     5 	sub MAIN (Str $file! where *.IO.f, UInt :$k=10) {
     6 	    my $seqIO = Bio::SeqIO.new(format => 'fasta', file => $file);
     7
     8 	    while (my $seq = $seqIO.next-Seq) {
     9 	        put join "\n", get-kmers(str => $seq.seq, k => $k);
    10 	    }
    11 	}
    12
    13 	sub get-kmers(Int :$k, Str :$str) {
    14 	    my $n = $str.chars - $k + 1;
    15 	    map { $str.substr($_, $k) }, 0..^$n;
    16 	}
```

This means we have to call ```get-kmers``` with Pairs for arguments (line 10), but it also means the arguments can be specified in any order.  Generally, if a function takes more than 2 or 3 arguments, I would recommend using named arguments.  

Something else you might have noticed in the above version is that I moved the check of the file-ness of ```$input``` into a constraint in the ```MAIN``` signature.  If you needed this contraint more than once, it would behoove you to create a new ```subset```.

Here is another version that again dips into the function programming world and uses a new string function called ```rotor``` (http://perl6.party/post/Perl-6-.rotor-The-King-of-List-Manipulation):

```
$ cat -n fasta-kmer3.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	use Bio::SeqIO;
     4
     5 	sub MAIN (Str $file!, UInt :$k=10) {
     6 	    die "Not a file ($file)" unless $file.IO.f;
     7 	    my $seqIO = Bio::SeqIO.new(format => 'fasta', file => $file);
     8
     9 	    my $j = -1 * ($k - 1);
    10 	    while (my $seq = $seqIO.next-Seq) {
    11 	        put $seq.seq.comb.rotor($k => $j).map(*.join).join("\n");
    12 	    }
    13 	}
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

# Non-sequence data

You can use k-mers to determine the similarity of non-sequence data.  This program will process all the given files in a pair-wise, all-versus-all fashion to determine if any two files are more similar than the overall similarity of all the files as figured with two-sided Student's t-test.  Notice that I can use Unicode characters like "μ" (mu) and "x̄" (bar-X) as variable names:

```
     1	#!/usr/bin/env perl6
     2	
     3	subset PosInt of Int where * > 0;
     4	
     5	sub MAIN (PosInt :$k=5, PosInt :$max-sd=5, *@files) {
     6	    die "No files" unless @files;
     7	    my @bags = map { find-kmers(+$k, $_) }, @files;
     8	    my %counts;
     9	    for (1..@bags.elems).combinations(2) -> ($i, $j) {
    10	        my $bag1  = @bags[$i-1];
    11	        my $bag2  = @bags[$j-1];
    12	        my $s1    = $bag1.Set;
    13	        my $s2    = $bag2.Set;
    14	        my @union = ($s1 (&) $s2).keys;
    15	        my $count = (map { $bag1{ $_ } }, @union)
    16	                  + (map { $bag2{ $_ } }, @union);
    17	        %counts{"$i-$j"} = $count;
    18	    }
    19	
    20	    my @n  = %counts.values;
    21	    my $μ  = mean @n;
    22	    my $sd = std-dev @n;
    23	    my $d  = $sd/(@n.elems).sqrt;
    24	    for %counts.kv -> $pair, $x̄ {
    25	        # https://en.wikipedia.org/wiki/Student%27s_t-test
    26	        my $t = ($x̄ - $μ) / $d;
    27	        if $t.abs > $max-sd {
    28	            my ($i, $j) = $pair.split('-');
    29	            my $f1 = @files[$i-1].IO.basename;
    30	            my $f2 = @files[$j-1].IO.basename;
    31	            put "$pair ($x̄) = $t [$f1, $f2]";
    32	        }
    33	    }
    34	}
    35	
    36	sub find-kmers (Int $k, Str $file) {
    37	    my $text = $file.IO.lines.lc.join(' ')
    38	               .subst(/:i <-[a..z\s]>/, '', :g).subst(/\s+/, ' ');
    39	    $text.comb.rotor($k => -1 * ($k - 1)).map(*.join).Bag;
    40	}
    41	
    42	sub mean (*@n) { @n.sum / @n.elems }
    43	
    44	sub std-dev (*@n) {
    45	    # https://en.wikipedia.org/wiki/Standard_deviation
    46	    my $mean = mean(@n);
    47	    my @dev  = map { ($_ - $mean)² }, @n;
    48	    my $var  = @dev.sum / @dev.elems;
    49	    return $var.sqrt;
    50	}
```

Here is the result of running this program on the homework submissions for a class of mine:

```
1-9 (74) = -6.36237878762838 [s01.pl6, s09.pl6]
1-4 (60) = -8.97194821224158 [s01.pl6, s04.pl6]
5-6 (136) = 5.19428580708723 [s05.pl6, s06.pl6]
3-7 (208) = 18.6149285622408 [s03.pl6, s07.pl6]
2-4 (76) = -5.98958315554078 [s02.pl6, s04.pl6]
6-8 (142) = 6.31267270335003 [s06.pl6, s08.pl6]
4-10 (74) = -6.36237878762838 [s04.pl6, s10.pl6]
7-8 (178) = 13.0229940809268 [s07.pl6, s08.pl6]
3-10 (208) = 18.6149285622408 [s03.pl6, s10.pl6]
3-8 (142) = 6.31267270335003 [s03.pl6, s08.pl6]
6-9 (78) = -5.61678752345318 [s06.pl6, s09.pl6]
4-5 (66) = -7.85356131597878 [s04.pl6, s05.pl6]
7-10 (198) = 16.7509504018028 [s07.pl6, s10.pl6]
4-6 (68) = -7.48076568389118 [s04.pl6, s06.pl6]
4-9 (72) = -6.73517441971598 [s04.pl6, s09.pl6]
5-8 (148) = 7.43105959961283 [s05.pl6, s08.pl6]
2-9 (80) = -5.24399189136558 [s02.pl6, s09.pl6]
```

The amount of similarity is fairly high, which is not unexpected from running this on Perl code; however, a few of these are really too high to be explained by chance.  The extremely high ones can be isolated:

```
$ ./similarity.pl6 --max-sd=16 ~/homework/*.pl6
3-7 (208) = 18.6149285622408 [s03.pl6, s07.pl6]
3-10 (208) = 18.6149285622408 [s03.pl6, s10.pl6]
7-10 (198) = 16.7509504018028 [s07.pl6, s10.pl6]
```

And, indeed, I found that "s03.pl6" was submitted with minor changes by two other students as "s07.pl6" and "s10.pl6" (names changed to protect the innocent, of course).