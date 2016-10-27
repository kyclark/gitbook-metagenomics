# Types and Regular Expressions

In the last chapter, I introduced a "subset" where you can create a new type in Perl.  I want to extend that and introduce multi methods.  First, let's look at a simple case where you might want to detect the type of sequence you have.  To do this, I'm also going to show you regular expressions (AKA "regexes" https://docs.perl6.org/language/regexes), a sub-language inside of Perl for describing patterns:

```
$ cat -n seq-type1.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	sub MAIN (Str $input!) {
     4 	    if $input ~~ /^ :i <[ACTGN]>+ $/ {
     5 	        put "Looks like DNA";
     6 	    }
     7 	    elsif $input ~~ /^ :i <[ACUGN]>+ $/ {
     8 	        put "Looks like RNA";
     9 	    }
    10 	    elsif $input ~~ /^ :i <[A..Z]>+ $/ {
    11 	        put "Looks like protein";
    12 	    }
    13 	    else {
    14 	        put "Unknown sequence type";
    15 	    }
    16 	}
$ ./seq-type1.pl6 ACGT
Looks like DNA
$ ./seq-type1.pl6 ACGU
Looks like RNA
$ ./seq-type1.pl6 EEDS
Looks like protein
$ ./seq-type1.pl6 883k19!
Unknown sequence type
```

The ```~~``` smart-match operator <https://docs.perl6.org/language/operators#infix_~~> matches the left side to the regex on the right.  Each regex is asking if the ```$input``` matches entirely to a given alphabet.  To break it down:

```
/ ^ :i <[ACTGN]>+ $ /
1 2 3  4        5 6 7
```

1. the start of a regular expression
2. hat or caret anchors to the beginning of the string
3. an "adverb" saying that the next part matches case-insensitively
4. a character class composed of the letters inside
5. one or more of the preceding
6. dollar anchors to the end of the string
7. the end of the regular expression

We've already seen that a long if/elsif chain is better represented with ```given/when```:

```
$ cat -n seq-type2.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	sub MAIN (Str $input!) {
     4 	    given $input {
     5 	        when /^ :i <[ACTGN]>+ $/ { put "Looks like DNA" }
     6 	        when /^ :i <[ACUGN]>+ $/ { put "Looks like RNA" }
     7 	        when /^ :i <[A..Z]>+  $/ { put "Looks like protein";
     8 	        default { put "Unknown sequence type" }
     9 	   }
    10 	}
```

But here is where it really gets interesting.  Perl can do pattern matching on the signatures of functions!

```
$ cat -n seq-type3.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	subset DNA     of Str where * ~~ /^ :i <[ACTGN]>+ $/;
     4 	subset RNA     of Str where * ~~ /^ :i <[ACUGN]>+ $/;
     5 	subset Protein of Str where * ~~ /^ :i <[A..Z]>+  $/;
     6
     7 	multi MAIN (DNA     $input!) { put "Looks like DNA" }
     8 	multi MAIN (RNA     $input!) { put "Looks like RNA" }
     9 	multi MAIN (Protein $input!) { put "Looks like Protein" }
    10 	multi MAIN (Str     $input!) { put "Unknown sequence type" }
$ ./seq-type3.pl6 ACGT
Looks like DNA
$ ./seq-type3.pl6 ACGU
Looks like RNA
$ ./seq-type3.pl6 EEDS
Looks like Protein
$ ./seq-type3.pl6 2112
Unknown sequence type
```

Once you've established types like this, you can use them in your code:

```
> 'ACGT' ~~ DNA
True
> 'ACGT' ~~ RNA
False
```

This makes your code both readable and self-documenting.

# Regexes

There is quite a bit more to say about regular expressions.  Maybe let's just play in the REPL:

```
$ perl6
To exit type 'exit' or '^D'
> 1234 ~~ /\d/
｢1｣
> 1234 ~~ /\d+/
｢1234｣
> 'foobar' ~~ /oo/
｢oo｣
> 'foooobar' ~~ /o+/
｢oooo｣
```