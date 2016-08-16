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

# CSV & tab-delimited files

One of the most common text file formats is records separated by newlines ("\n" or "\r\n" on Windows) where the fields of the records are delimited by commas ("comma-separated values" or "CSV") or tabs ("tab-delimited").  Here is an example that will extract a field from a delimited text file showing "[t]he leading causes of death by sex and ethnicity in New York City in since 2007" (https://data.cityofnewyork.us/api/views/jb7j-dtam/rows.csv?accessType=DOWNLOAD).

```
$ cat -n simple1.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	use v6;
     4
     5 	sub MAIN (Str $file!, Int :$n=0, Str :$sep=',') {
     6 	    die "Not a file ($file)" unless $file.IO.f;
     7
     8 	    for $file.IO.lines -> $line {
     9 	        my @fields = $line.split($sep);
    10 	        put @fields[$n];
    11 	    }
    12 	}
$ ./simple1.pl6 causes.csv | head -3
Year
2010
2010
```

The problem with relying on the position of data is that someone may change the number of columns, and that will break your code.  It's better, when possible, to use the *name* of the fields, and this particular file does have named fields:

```
$ head -1 causes.csv
Year,Ethnicity,Sex,Cause of Death,Count,Percent
```

Here's a version that merges the headers on the first line with each data record to create a hash:

```
$ cat -n parser2.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	use v6;
     4
     5 	sub MAIN (Str $file!, Str :$sep=',', Int :$limit=0) {
     6 	    die "Not a file ($file)" unless $file.IO.f;
     7
     8 	    my $fh = open $file;
     9 	    my @fields = $fh.get.split($sep);
    10
    11 	    my @data;
    12 	    for $fh.lines -> $line {
    13 	        @data.push(@fields Z=> $line.split($sep));
    14 	        last if $limit > 0 && @data.elems > $limit;
    15 	    }
    16
    17 	    say @data;
    18 	}
$ ./parser2.pl6 --limit=1 causes.csv
[(Year => 2010 Ethnicity => NON-HISPANIC BLACK Sex => MALE Cause of Death => HUMAN IMMUNODEFICIENCY VIRUS DISEASE Count => 297 Percent => 5) (Year => 2010 Ethnicity => NON-HISPANIC BLACK Sex => MALE Cause of Death => INFLUENZA AND PNEUMONIA Count => 201 Percent => 3)]
```

In this version, I ```open``` the file (line 8) to get a "filehandle" (often abbreviated "fh") which is a way to get access to the contents of the file.  I did this so that I could call the ```get``` method (https://docs.perl6.org/type/IO$COLON$COLONHandle#method_get) to retrieve just the first line which I expect to have the field names.  That returns a string which I then call ```split``` using the ```$sep``` (separator) argument (which defaults to a comma).

At lines 10-11, I initialize two variables that I will need inside the ```for``` loop.  Consider that I'm about to go picking apples.  Before I go to the orchard, I need to get a basket to carry the apples back out.  The ```@data``` is array is my basket.  At line 13, I call the ```push``` method to add the result of a zip operation.

I added a "--limit" option so I could stop it on the first record.  I also introduce the ```say``` function so you can look at the data that was collected.  The function ```dd``` (data dump) will also show you the structure:

```
Array @data = [(:Year("2010"), :Ethnicity("NON-HISPANIC BLACK"), :Sex("MALE"), "Cause of Death" => "HUMAN IMMUNODEFICIENCY VIRUS DISEASE", :Count("297"), :Percent("5")).Seq, (:Year("2010"), :Ethnicity("NON-HISPANIC BLACK"), :Sex("MALE"), "Cause of Death" => "INFLUENZA AND PNEUMONIA", :Count("201"), :Percent("3")).Seq]
```

These differ from ```print``` and ```put``` which both show:

```
Year   	2010 Ethnicity 	NON-HISPANIC BLACK Sex 	MALE Cause of Death    	HUMAN IMMUNODEFICIENCY VIRUS DISEASE Count     	297 Percent    	5 Year 	2010 Ethnicity	NON-HISPANIC BLACK Sex  	MALE Cause of Death    	INFLUENZA AND PNEUMONIA Count 	201 Percent     	3
```

