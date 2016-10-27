# Types and Regular Expressions

Regular expressions (https://docs.perl6.org/language/regexes) constitute a sub-language within Perl.  There have been many changes from Perl 5's regex syntax, but the principles remain the same.  

Let's play in the REPL:

```
$ perl6
To exit type 'exit' or '^D'
> 1234 ~~ /\d/
｢1｣
> 1234 ~~ /\d+/
｢1234｣
> 1234 ~~ Int
True
> 'foobar' ~~ /oo/
｢oo｣
> 'foooobar' ~~ /o+/
｢oooo｣
```

Here's a more complicated regex for detecting US-style phone numbers:

```
> my $re = /'('? \d ** 3 ')'? <[\s*-]>? \d ** 3 <[\s*-]>? \d ** 4 /
> /'('? \d ** 3 ')'? <[\s*-]>? \d ** 3 <[\s*-]>? \d ** 4 /
> my @phones = "(520) 321 9087", "520 321 9087", "520-321-9087", "5203219087", "(520)321-9087"
[(520) 321 9087 520 321 9087 520-321-9087 5203219087 (520)321-9087]
> for @phones -> $phone { say $phone ~~ $re }
True
True
True
True
True
> all(@phones) ~~ $re
True
```

The last method uses a Junction (https://docs.perl6.org/type/Junction) to compare all the phone numbers at once.

The subject of regexes is far too deep to cover here.  For now, just know that, if you can describe the pattern of text you are searching for, then you can probably write a regex to match it.  I want to use regexes as a bridge to talking about creating your own types and something called "multiple dispatch."

In the last chapter, I introduced a "subset" where you can create a new type in Perl.  First, let's look at a simple case where you might want to detect the type of sequence you have.

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

Now let's move our regular expressions into types.  There are many reasons for doing this, not the least of which is that they are then documented and represented in just one place in your code.  Suppose that you initially defined the DNA pattern as just "ACGT" and used it several places in a script.  Later you realize you want to add "N" or maybe even the full IUPAC table (http://www.bioinformatics.org/sms/iupac.html).  If you fail to update the regex in every place in your code, you've introduced a very subtle bug that you may spend hours fixing -- or worse, you may never find!  See the difference with this code:

```
$ cat -n seq-type3.pl6
     1	#!/usr/bin/env perl6
     2
     3	subset DNA     of Str where * ~~ /^ :i <[ACTGN]>+ $/;
     4	subset RNA     of Str where * ~~ /^ :i <[ACUGN]>+ $/;
     5	subset Protein of Str where * ~~ /^ :i <[A..Z]>+  $/;
     6
     7	sub MAIN (Str $input!) {
     8	    given $input {
     9	        when DNA     { put "Looks like DNA" }
    10	        when RNA     { put "Looks like RNA" }
    11	        when Protein { put "Looks like protein" }
    12	        default      { put "Unknown sequence type" }
    13	    }
    14	}
```

Now whenever you need the DNA pattern, you just reach for your ```DNA``` type.  You could even define common regexes/patterns in a single module (file) that you import into your scripts so that you only ever define (and fix) them in one place.

Now for that "multiple dispatch" I mentioned earlier.  Perl can do pattern matching on the signatures of functions.  We can use the ```multi``` keyword instead of ```sub``` to indicate to Perl that we intend to define ```MAIN``` in different ways:

```
$ cat -n seq-type4.pl6
     1	#!/usr/bin/env perl6
     2
     3	subset DNA     of Str where * ~~ /^ :i <[ACTGN]>+ $/;
     4	subset RNA     of Str where * ~~ /^ :i <[ACUGN]>+ $/;
     5	subset Protein of Str where * ~~ /^ :i <[A..Z]>+  $/;
     6
     7	multi MAIN (DNA     $input!) { put "Looks like DNA" }
     8	multi MAIN (RNA     $input!) { put "Looks like RNA" }
     9	multi MAIN (Protein $input!) { put "Looks like Protein" }
    10	multi MAIN (Str     $input!) { put "Unknown sequence type" }
$ ./seq-type4.pl6 ACGT
Looks like DNA
$ ./seq-type4.pl6 ACGU
Looks like RNA
$ ./seq-type4.pl6 EEDS
Looks like Protein
$ ./seq-type4.pl6 2112
Unknown sequence type
```
