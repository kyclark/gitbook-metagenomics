# File handling

Most data in bioinformatics is exchanged an some sort of text format (GenBank, GFF, FASTA, EMBL, XML, OBO, etc.).  Reading and writing from text is bread-and-butter work, so let's do some.

# Tests

Given some input, how do you know it's a file, that you can read it, etc.?  In our bash section, we came across Unix file tests with this:

```
if [[ ! -d $OUT_DIR ]]; then
  mkdir -p $OUT_DIR
fi
```

Perl has similar tests, and to use them you simply tell Perl to treat the variable as an IO object and then execute the method for the test.  You can find all the tests https://docs.perl6.org/type/IO$COLON$COLONPath#File_test_operators.  So the above in Perl would look like this:

```
mkdir $out-dir unless $out-dir.IO.d;
```

# Say it loud...

Here's a trivial example to UPPERCASE the contents of a file:

```
$ cat -n upper1.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	use v6;
     4
     5 	sub MAIN (Str $file!) {
     6 	    die "Not a file ($file)" unless $file.IO.f;
     7
     8 	    for $file.IO.lines -> $line {
     9 	        put $line.uc;
    10 	    }
    11 	}
$ ./upper1.pl6 foo
Not a file (foo)
  in sub MAIN at ./upper1.pl6 line 6
  in block <unit> at ./upper1.pl6 line 5
$ ./upper1.pl6 jabberwocky.txt | head -6
JABBERWOCKY -- LEWIS CARROLL

'TWAS BRILLIG, AND THE SLITHY TOVES
  DID GYRE AND GIMBLE IN THE WABE:
ALL MIMSY WERE THE BOROGOVES,
  AND THE MOME RATHS OUTGRABE.
```

At line 6, we should first check that the input was actually a file.  If it is not, then we ```die``` with an error message which will halt execution (see above).  At line 8, we start a ```for``` loop using the ```$file.IO.lines``` (https://docs.perl6.org/type/IO$COLON$COLONHandle#method_lines) method to open the file and read in each of the lines into the ```$line``` variable.  Lastly we ```put``` the uppercase (```uc```) version of the line.

We can borrow a technique from functional programming with ```map``` to make this shorter:

```
$ cat -n upper2.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	use v6;
     4
     5 	sub MAIN (Str $file!) {
     6 	    die "Not a file ($file)" unless $file.IO.f;
     7
     8 	    put $file.IO.lines.map(*.uc).join("\n");
     9 	}
 ```
 
> Golfing: Sometimes you may here about programmers (often Perl hackers) who like to "golf" their programs.  It's an attempt to create a program using the fewest keystrokes as possible, similar to the strategy in golf where the player tries to strike the ball as few times as possible to put it into the cup.  It's not a necessarily admirable quality to make one's code as terse as possible, but there is some truth to the notion that more code means more bugs.  There are incredibly powerful ideas built into every language, and learning how they can save you from writing code is worth the effort of writing fewer bugs.  If you understand ```map```, then you probably also understand why ```for``` loops and mutable variables are dangerous.  If you don't, then keep studying.

