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
    13 	        my @values = $line.split($sep);
    14 	        my %record;
    15 	        for 0..^@fields.elems -> $i {
    16 	            my $key = @fields[$i];
    17 	            my $val = @values[$i];
    18 	            %record{ $key } = $val;
    19 	        }
    20 	        @data.push(%record);
    21
    22 	        last if $limit > 0 && @data.elems > $limit;
    23 	    }
    24
    25 	    say @data;
    26 	}
$ ./parser2.pl6 --limit=1 causes.csv
[(Year => 2010 Ethnicity => NON-HISPANIC BLACK Sex => MALE Cause of Death => HUMAN IMMUNODEFICIENCY VIRUS DISEASE Count => 297 Percent => 5) (Year => 2010 Ethnicity => NON-HISPANIC BLACK Sex => MALE Cause of Death => INFLUENZA AND PNEUMONIA Count => 201 Percent => 3)]
```

In this version, I ```open``` the file (line 8) to get a "filehandle" (often abbreviated "fh") which is a way to get access to the contents of the file.  I did this so that I could call the ```get``` method (https://docs.perl6.org/type/IO$COLON$COLONHandle#method_get) to retrieve just the first line which I expect to have the field names.  That returns a string which I then call ```split``` using the ```$sep``` (separator) argument (which defaults to a comma).

At lines 10-11, I initialize two variables that I will need inside the ```for``` loop.  Consider that I'm about to go picking apples.  Before I go to the orchard, I need to get a basket to carry the apples back out.  The ```@data``` is array is my basket.  

Lines 14-19 are probably a little confusing, so let's break it down.  I want to create a key-value structure that associates the headers in the first line to the fields in each record, so I declare ```my %record``` at line 14.  Then I want to step through the *numbered positions* of the ```@fields``` so that I can pair them up with their ```@values```.  List-type structures in Perl start numbering at 0, so I start my ```for``` loop with that.  Then I use the ```..^``` operator to construct a Range (https://docs.perl6.org/type/Range) that goes up to *but not including* the number of elements in ```@fields```.  It's the hat ```^``` that says to the Range constructor to stop one less than the thing it's next to.  You can also put it on the left or both (or neither).  We have to stop before the *number* of elements in ```@fields``` because of the 0-based numbering -- that is, if there are 3 ```@fields```, then they are in positions 0, 1, and 2.  So that explains the loading of the ```$i``` variable (NB: "i" is a very common "integer" variable.  If a second value is needed, then it's common to move on to "j," "k," etc.)  

Assume for a moment that our data looks like this:

```
# position =  0    1                2
my @fields = <name rank             serial_number>;
my @values = <Gene "Staff Sargeant" 1656401>;

```

A lines 16-17, I get the key and value from the appropriate place in the parallel arrays.  So when ```$i``` is 0, I get ```$key="name"``` and ```$value="Gene"```.  For 1, it will be ```$key="rank"``` and ```$value="Staff Sargeant"```, and for 2 ```$key="serial_number"``` and ```$value="1656401"```.  I can then set ```%record{ $key } = $value```.  The hash grows as we add key/values pairs.  It might help to see it in the REPL:

```
> my %record;
{}
> %record<name> = 'Gene'
Gene
> %record
{name => Gene}
> %record<rank> = "Staff Sargeant"
Staff Sargeant
> %record
{name => Gene, rank => Staff Sargeant}
> %record<serial_number> = "1656401"
1656401
> %record
{name => Gene, rank => Staff Sargeant, serial_number => 1656401}
```

If we had all the information to start, we could just create the whole hash like so:

```
> my %record = name => "Gene", rank => "Staff Sargeant", serial_number => "1656401";
> %record
{name => Gene, rank => Staff Sargeant, serial_number => 1656401}
```

After we've created our ```%record```, we can add it to our basket with ```@data.push(%record)```.

At line 22, I handle the "--limit" option I added so that I could stop processing on the first record.  Here I'm using ```last``` to exit the loop if I have a positive value for ```$limit``` (the default is 0) and the number of elements in my ```@data``` are equal to the limit. 

At line 25, I've introduced the ```say``` function so you can look at the data that was collected.  The function ```dd``` (data dump) will also show you the structure:

```
Array @data = [(:Year("2010"), :Ethnicity("NON-HISPANIC BLACK"), :Sex("MALE"), "Cause of Death" => "HUMAN IMMUNODEFICIENCY VIRUS DISEASE", :Count("297"), :Percent("5")).Seq, (:Year("2010"), :Ethnicity("NON-HISPANIC BLACK"), :Sex("MALE"), "Cause of Death" => "INFLUENZA AND PNEUMONIA", :Count("201"), :Percent("3")).Seq]
```

These differ from ```print``` and ```put``` which both show:

```
Year   	2010 Ethnicity 	NON-HISPANIC BLACK Sex 	MALE Cause of Death    	HUMAN IMMUNODEFICIENCY VIRUS DISEASE Count     	297 Percent    	5 Year 	2010 Ethnicity	NON-HISPANIC BLACK Sex  	MALE Cause of Death    	INFLUENZA AND PNEUMONIA Count 	201 Percent     	3
```

This next version will use a powerful idea from functional programming called a "zip" (```Z``` https://docs.perl6.org/routine/Z) to pair up the ```@fields``` and ```@values```.  Like an actual zipper, this takes one element from each list and creates a new list of lists:

```
> @fields Z @values
((name Gene) (rank "Staff) (serial_number Sargeant"))
```

However, what we want is a new list of Pairs which are created with the ```=>``` operator (https://docs.perl6.org/routine/$EQUALS_SIGN$GREATER-THAN_SIGN), so we just mush the two operators together to create a new zip-and-pair operator ```Z=>```:

```
> my %record = @fields Z=> @values
{name => Gene, rank => "Staff, serial_number => Sargeant"}
```

Zips can work with lists of different values.  Here is how I can pair the list ("a", "b", "c") with a lazy, infinite list of integers:

```
> <a b c> Z 1..*
((a 1) (b 2) (c 3))
```

As you will see this saves many lines of code.  I've also thrown in the ability to skip lines that look like "comments" based on how the user defines a comment:

```
$ cat -n parser3.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	use v6;
     4
     5 	sub MAIN (Str $file!, Str :$sep=',', Int :$limit=0, Str :$comment) {
     6 	    die "Not a file ($file)" unless $file.IO.f;
     7
     8 	    # copy to make mutable
     9 	    (my $delim = $sep) ~~ s/\\t/\t/;
    10 	    my $fh     = open $file;
    11 	    my @fields = $fh.get.split($delim);
    12
    13 	    my @data;
    14 	    for $fh.lines -> $line {
    15 	        next if $comment.defined &&
    16 	                $line.substr(0, $comment.chars) eq $comment;
    17 	        @data.push(@fields Z=> $line.split($delim));
    18 	        last if $limit > 0 && @data.elems > $limit;
    19 	    }
    20
    21 	    say @data;
    22 	}
```

Line 9 is pretty cryptic and needs to be explained.  First off, when parsing tab-delimited files, it's difficult to pass in a non-printable character on the command line.  Passing an *actual* tab (by hitting the "Tab" key) doesn't work, and most people wouldn't know to enter:

```
./parser3.pl6 --sep=$(\t) ...
```

So we assume that the user will enter this:

```
./parser3.pl6 --sep=\t ...
```

Which will then come in to us looking like "\\t".  We need to turn that into just "\t" which is the escape character representing the "Tab."  We can do that with the ```s//``` substitution operator (https://docs.perl6.org/language/operators#Substitution_Operators), but not on the ```$sep``` variable because it is *read-only.*  So we copy it into ```$delim``` and then immediately bind (with ```~~```) to the substitution command.  Et voila, we have a "Tab" we can use.

Line 15 is the logic for skipping comments.  In the way that ```last``` will exit a loop, ```next``` will immediately jump to the next iteration of the loop.  Line 16 use a ```substr``` (substring) operation to look at the first part of the ```$line``` to see if it matches the ```$comment``` string.  I'm taking into account that the ```$comment``` may be more than one character (e.g., "//").  Line 17 doesn't bother to create a ```%record``` but just pushes the result of the zip ```Z=>```.

And with that, we have written a fairly decent delimited text parser!  This sort of thing in very standard, and so it would actually behoove us to use an existing module such as "CSV::Parser" or "Text::CSV" (http://modules.perl6.org/#q=csv).  They will both accomplish the task, so it's just a matter of which one appeals to you more.

You can install a module using the "panda" tool:

```
$ panda install CSV::Parser
==> Fetching CSV::Parser
==> Building CSV::Parser
==> Testing CSV::Parser
t/01_multiline_csv.t ... ok
t/02_escaped_csv.t ..... ok
t/03_delimiters_csv.t .. ok
t/04_binary_csv.t ...... ok
All tests successful.
Files=4, Tests=4,  2 wallclock secs ( 0.02 usr  0.01 sys +  1.75 cusr  0.22 csys =  2.00 CPU)
Result: PASS
==> Installing CSV::Parser
==> Successfully installed CSV::Parser
```

Here is how the code would look:

```
$ cat -n parser4.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	use v6;
     4 	use CSV::Parser;
     5
     6 	sub MAIN (Str $file!, Str :$sep=',', Int :$limit=0, Str :$comment) {
     7 	    die "Not a file ($file)" unless $file.IO.f;
     8 	    my $fh     = open $file;
     9 	    my $parser = CSV::Parser.new(
    10 	                 file_handle => $fh, contains_header_row => True );
    11
    12 	    my @data;
    13 	    until $fh.eof {
    14 	        @data.push(%($parser.get_line))
    15 	    }
    16
    17 	    say @data;
    18 	}
```

This module does not handle comment lines.  If you needed that, you could either go back to your own module or you could look at the author's code and add to it.  Most author's will have a Github repository that you can fork and then submit a "pull request" (PR).  Also, you might just write the author and ask very nicely to add the feature you want.

# GFF

The "general feature format" (https://en.wikipedia.org/wiki/General_feature_format) is a tab-delimited specification for describing genomic features.  Version 3 of GFF specifies nine fields:

1. sequence
2. source
3. feature
4. start
5. end
6. score
7. strand
8. frame
9. attributes

Let's download a GFF file for yeast:

```
$ wget http://downloads.yeastgenome.org/curation/chromosomal_feature/saccharomyces_cerevisiae.gff
$ head -20 saccharomyces_cerevisiae.gff
##gff-version 3
#date Sun Aug 28 19:50:04 2016
#
# Saccharomyces cerevisiae S288C genome (version=R64-2-1)
#
# Features from the 16 nuclear chromosomes labeled chrI to chrXVI,
# plus the mitochondrial genome labeled chrmt.
#
# Created by Saccharomyces Genome Database (http://www.yeastgenome.org/)
#
# Weekly updates of this file are available for download from:
# http://downloads.yeastgenome.org/curation/chromosomal_feature/saccharomyces_cerevisiae.gff
#
# Please send comments and suggestions to sgd-helpdesk@lists.stanford.edu
#
# SGD is funded as a National Human Genome Research Institute Biomedical Informatics Resource from
# the U. S. National Institutes of Health to Stanford University.
#
chrI   	SGD    	chromosome     	1      	230218 	.      	.      	.      	ID=chrI;dbxref=NCBI:;Name=chrI
chrI   	SGD    	telomere       	1      	801    	.      	-      	.      	ID=TEL01L;Name=TEL01L;Note=Telomeric%20region%20on%20the%20left%20arm%20of%20Chromosome%20I%3B%20composed%20of%20an%20X%20element%20core%20sequence%2C%20X%20element%20combinatorial%20repeats%2C%20and%20a%20short%20terminal%20stretch%20of%20telomeric%20repeats;display=Telomeric%20region%20on%20the%20left%20arm%20of%20Chromosome%20I;dbxref=SGD:S000028862
```

It's obvious that the lines beginning with "#" are comments and metadata.  After that, the data begins.  If we wanted to find out how many "chromosome" features exist and their names:

```
$ awk -F"\t" '$3 == "chromosome"' saccharomyces_cerevisiae.gff | wc -l
      17
$ awk -F"\t" '$3 == "chromosome" {print $1}' saccharomyces_cerevisiae.gff | cat -n
     1 	chrI
     2 	chrII
     3 	chrIII
     4 	chrIV
     5 	chrV
     6 	chrVI
     7 	chrVII
     8 	chrVIII
     9 	chrIX
    10 	chrX
    11 	chrXI
    12 	chrXII
    13 	chrXIII
    14 	chrXIV
    15 	chrXV
    16 	chrXVI
    17 	chrmt
```

Let's look at the top 10 most numerous features in the yeast genome:

```
$ awk -F"\t" '{print $3}' saccharomyces_cerevisiae.gff | sort | uniq -c | sort -rn | head
152010
7058 CDS
6600 mRNA
6600 gene
 484 noncoding_exon
 383 long_terminal_repeat
 377 intron
 352 ARS
 299 tRNA_gene
 196 ARS_consensus_sequence
```

There's only so far we can get with ```awk```, though, before we need to get into Perl.  For instance, let's find two overlapping genes:

```
$ cat -n gff1.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	# http://downloads.yeastgenome.org/curation/chromosomal_feature/saccharomyces_cerevisiae.gff
     4
     5 	use URI::Encode;
     6
     7 	sub MAIN (Str $gff! where *.IO.f) {
     8 	    my $fh = open $gff;
     9
    10 	    my %loc;
    11 	    for $fh.lines -> $line {
    12 	        next if $line ~~ / ^ '#' /; # comment line
    13 	        my ($sequence, $source, $feature, $start, $end, $score,
    14 	            $strand, $frame, $attributes) = $line.split(/\t/);
    15
    16 	        next unless ($feature.defined && $feature eq 'gene')
    17 	                 && ($strand.defined  && $strand ~~ / ^ <[+-]> $/);
    18
    19 	        my %attr = $attributes.split(';').map(&uri_decode)
    20 	                   .map(*.split('=')).map({ $^a.[0].lc => $^a.[1] });
    21 	        my $name = %attr<name> || 'NA';
    22
    23 	        %loc{ $sequence }{ $strand }.push(
    24 	            ($name, [+$start .. +$end]) # force numeric, important!
    25 	        );
    26 	    }
    27 	    $fh.close;
    28
    29 	    for %loc.keys -> $chr {
    30 	        my @forward-genes = %loc{ $chr }{'+'}.list;
    31 	        my @reverse-genes = %loc{ $chr }{'-'}.list;
    32 	        note sprintf "chr (%s) has %s forward genes, %s reverse genes\n",
    33 	            $chr, @forward-genes.elems, @reverse-genes.elems;
    34
    35 	        for @forward-genes -> ($gene1, $pos1) {
    36 	            for @reverse-genes -> ($gene2, $pos2) {
    37 	                if so $pos1 (&) $pos2 {
    38 	                    printf "%s [%s] (%s) => %s [%s] (%s)\n",
    39 	                        $gene1, '+', $pos1[0,*-1].join('..'),
    40 	                        $gene2, '-', $pos2[0,*-1].join('..');
    41 	                }
    42 	            }
    43 	        }
    44 	    }
    45 	}
```