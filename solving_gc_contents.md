# Solving GC contents

For this section, I'm going to walk through an evolution of the "DNA" problem from the Rosalind.info website (http://rosalind.info/problems/dna).  Here is a first version:

```
$ cat -n dna1.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	sub MAIN (Str $dna!) {
     4 	    my ($num-A, $num-C, $num-G, $num-T) = 0, 0, 0, 0;
     5
     6 	    for $dna.comb -> $letter {
     7 	        if    $letter eq 'a' || $letter eq 'A' { $num-A++ }
     8 	        elsif $letter eq 'c' || $letter eq 'C' { $num-C++ }
     9 	        elsif $letter eq 'g' || $letter eq 'G' { $num-G++ }
    10 	        elsif $letter eq 't' || $letter eq 'T' { $num-T++ }
    11 	    }
    12
    13 	    put "$num-A $num-C $num-G $num-T";
    14 	}
$ ./dna1.pl6 AGCTTTTCATTCTGACTGCAACGGGCAATATGTCTCTGTGTGGATTAAAAAAAGAGTGTCTGATAGCAGC
20 12 17 21
$ ./dna1.pl6 foobar
1 0 0 0
```

Not a bad start.  Our program expects a String of DNA on the command line (line 3).  We declare four variables at line 4 to hold the counts for A, C, G, and T.  Line 6 sets up a ```for``` loop (https://docs.perl6.org/syntax/for) that calls the ```comb``` method (https://docs.perl6.org/type/Str#routine_comb) on the ```$dna``` to supply ```$letter``` so that we can inspect each letter individually.  Lines 7-10 compare each letter's upper- and lowercase version to decide which counter to increment with  ```++``` (https://docs.perl6.org/routine/$PLUS_SIGN$PLUS_SIGN).  *NB: Lowercase is often used to "softmask" highly repetitive regions of DNA like in plant genomes.*   At line 13, we print out the results as described on the website.

Here's a version that makes some improvements:

```
$ cat -n dna2.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	sub MAIN (Str $dna!) {
     4 	    my ($num-A, $num-C, $num-G, $num-T) = 0, 0, 0, 0;
     5
     6 	    for $dna.lc.comb -> $letter {
     7 	        if    $letter eq 'a' { $num-A++ }
     8 	        elsif $letter eq 'c' { $num-C++ }
     9 	        elsif $letter eq 'g' { $num-G++ }
    10 	        elsif $letter eq 't' { $num-T++ }
    11 	    }
    12
    13 	    put join ' ', $num-A, $num-C, $num-G, $num-T;
    14 	}
```

I wanted to get rid of checking both cases of the letters, so at line 6 I first ```lc``` (lowercase) ```$dna``` and then get the letters.  Notice that you can chain methods.  Here is how you can reverse a string:

```
> "Because I could not stop for Death".comb.reverse.join
htaeD rof pots ton dluoc I esuaceB
```

> Protip: ```reverse``` is a list operation, so you have to convert the string into a list of letters, reverse those, then ```join``` them back together to create a new string.  If you wanted to reverse the words, then you would do this:

```
> "Because I could not stop for Death".words.reverse.join(' ')
Death for stop not could I Because
```

Most languages have a "switch" or "case" (like for our options checking in bash) statement.  Perl's is called ```given``` (https://docs.perl6.org/language/control#given), it's like a giant if/elsif statement, only much, much better:

```
$ cat -n dna3.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	sub MAIN (Str $dna!) {
     4 	    my ($num-A, $num-C, $num-G, $num-T) = 0, 0, 0, 0;
     5
     6 	    for $dna.lc.comb -> $letter {
     7 	        given $letter {
     8 	            when 'a' { $num-A++ }
     9 	            when 'c' { $num-C++ }
    10 	            when 'g' { $num-G++ }
    11 	            when 't' { $num-T++ }
    12 	        }
    13 	    }
    14
    15 	    put ($num-A, $num-C, $num-G, $num-T).join(' ');
    16 	}
```

Notice the change at line 15 where I make a temporary (anonymous) list from the four counters and then call the ```.join``` method on the list. 

What's fun is that ```for``` can act like ```given``` and use smart matching on ```$_``` AKA "the topic" (or "thing" or "it").  Here's a shorter version:

```
$ cat dna4.pl6
#!/usr/bin/env perl6

sub MAIN (Str $dna!) {
    my ($num-A, $num-C, $num-G, $num-T) = 0, 0, 0, 0;

    for $dna.lc.comb {
        when 'a' { $num-A++ }
        when 'c' { $num-C++ }
        when 'g' { $num-G++ }
        when 't' { $num-T++ }
    }

    put ($num-A, $num-C, $num-G, $num-T).join(' ');
}
```

Now I'm going to show you a version where we don't have to create variables for each of the nucleotides:

```
$ cat -n dna5.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	sub MAIN (Str $dna!) {
     4 	    my %count = A => 0, C => 0, G => 0, T => 0;;
     5
     6 	    for $dna.lc.comb {
     7 	        when 'a' { %count<A>++ }
     8 	        when 'c' { %count<C>++ }
     9 	        when 'g' { %count<G>++ }
    10 	        when 't' { %count<T>++ }
    11 	    }
    12
    13 	    put join ' ', %count<A C T G>;
    14 	}
```

At line 4, I declare the hash ```%count``` that will hold key-value pairs where the keys will be the nucleotides and the values will be the number of times we saw them.  It's important (for this version) to initialize the counts or line 13 will complain if a given base does not have a value.  To get around that, we can use a ```map``` statement:

```
$ cat -n dna6.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	sub MAIN (Str $dna!) {
     4 	    my %count;
     5
     6 	    for $dna.lc.comb {
     7 	        when 'a' { %count<A>++ }
     8 	        when 'c' { %count<C>++ }
     9 	        when 'g' { %count<G>++ }
    10 	        when 't' { %count<T>++ }
    11 	    }
    12
    13 	    put join ' ', %count<A C T G>.map({ $_ // 0 });
    14 	}
```

This version has no initialization and waits until line 13 to determine if there is something that exists for a given base.  Let's examine this in the REPL:

```
> my %count = A => 1, G => 5;
{A => 1, G => 5}
> %count<A C T G>
(1 (Any) (Any) 5)
> join ' ', %count<A C T G>
Use of uninitialized value <element> of type Any in string context
Any of .^name, .perl, .gist, or .say can stringify undefined things, if needed.  in block <unit> at <unknown file> line 1
Use of uninitialized value <element> of type Any in string context
Any of .^name, .perl, .gist, or .say can stringify undefined things, if needed.  in block <unit> at <unknown file> line 1
> say %count<A C T G>.WHAT
(List)
> %count<A C T G>.map({ $_  // 0 })
(1 0 0 5)
> join ' ', %count<A C T G>.map({ $_  // 0 })
1 0 0 5
```

What the above shows is that we get a List back when we ask for multiple keys from a hash.  That list might contain nothing for a given key, so we use the ```map``` function to apply an anonymous function (the bit in ```{}```) to each member of the list.  The function says "either *the topic* or, if it's not defined, then 0."  

Here's a version that introduces a new type called a "Bag" (https://docs.perl6.org/type/Bag):

```
$ cat -n dna7.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	sub MAIN (Str $dna!) {
     4 	    my $bag = $dna.lc.comb.Bag;
     5
     6 	    put join ' ', $bag<a c g t>
     7 	}
 ```
 
Next, I'd like to create a version that can handle input from a file as well as a string:

```
$ cat -n dna8.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	sub MAIN (Str $input!) {
     4 	    my $dna = $input.IO.e ?? $input.IO.slurp !! $input;
     5 	    my $bag = $dna.lc.comb.Bag;
     6 	    put join ' ', $bag<a c g t>
     7 	}
 $ echo AGCTTTTCATTCTGACTGCAACGGGCAATATGTCTCTGTGTGGATTAAAAAAAGAGTGTCTGATAGCAGC > dna
$ ./dna8.pl6 dna
20 12 17 21
$ ./dna8.pl6 foobar
1 0 0 0
```

At line 4, I'm going to set my ```$dna``` equal to either the contents of the ```$input``` (if it is a file) or the input itselt.  I'm using a ternary operator (https://docs.perl6.org/language/operators#index-entry-Ternary_operator) that follow the pattern:

```
result = conditional ?? true-branch !! false-branch
```

In my case, the conditional is to the the ```$input``` as an ```IO``` object and test the ```e``` (exists, https://docs.perl6.org/type/IO$COLON$COLONPath#File_test_operators).  If it is ```True```, then execute the ```slurp``` method on the IO object; otherwise, just use the ```$input```.  Notice that the Bag returns "0" for any missing elements, so no initialization was required.

The advantage to using the "bag" solution is that we can easily expand the script to also count, for instance, "N"s which are unknown base calls or the bases in RNA or proteins:

```
$ cat -n char-count.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	sub MAIN (Str $input!) {
     4 	    my $text  = $input.IO.e ?? $input.IO.slurp !! $input;
     5 	    my $bag   = $text.comb.Bag;
     6 	    for $bag.keys.grep(/<[a..zA..Z]>/).sort -> $char {
     7 	        put join "\t", $char, $bag{$char};
     8 	    }
     9 	}
$ ./char-count.pl6 AANCTANGNACTAGG
A      	5
C      	2
G      	3
N      	3
T      	2
$ ./char-count.pl6 prot.fa
A      	58
C      	47
D      	42
E      	55
F      	36
G      	60
H      	47
I      	47
K      	42
L      	44
M      	51
N      	54
P      	56
Q      	50
R      	59
S      	42
T      	45
V      	50
W      	55
Y      	60
```