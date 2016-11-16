# Sharing code with modules and objects

As you write more code, you will find yourself solving some problems repeatedly.  Your first instinct might be to copy and paste the needed code, but it is far better to put it into a module or object to make it easier to re-use it.  For one thing, if you find a bug, you'll want to change it just once and not have to remember all the places you pasted it.  Eventually you may find you've solved a problem that many other people have or, vice versa, others have solved your problem.  Modules are the way to package and share code.

# Modules

Here's an example of a simple module that has some code/type/functions that we've seen before:

```
$ cat -n DNA.pm6
     1	unit module DNA;
     2
     3	subset DNA of Str is export where /^ :i <[ACGTN]>+ $/;
     4
     5	sub revcom (DNA $seq) is export {
     6	    $seq.trans(<A C G T a c g t> => <T G C A t g c a>).flip;
     7	}
     8
     9	sub hamming (DNA $s1, DNA $s2) is export {
    10	    ($s1.chars - $s2.chars).abs +
    11	    ($s1.comb Z $s2.comb).grep({ $^a[0] ne $^a[1] });
    12	}
```

I created a file called "DNA.pm6" ("pm" == "Perl module") with a ```unit module DNA``` line to establish for Perl that this is the "DNA" module.  Inside it, I have defined a ```subset``` type called "DNA" that allows me to easily reuse my regular expression.  I also have two functions that are generally useful, ```revcom``` to compute the reverse complement of a string of DNA, and ```hamming``` to compute the number of point variations between two pieces of DNA.  Notice the ```is export``` statement that allows the "DNA" module to allow the code to be used in another program like so:

```
$ cat -n module1.pl6
     1	#!/usr/bin/env perl6
     2
     3	use lib '.';
     4	use DNA;
     5
     6	sub MAIN (DNA $seq) {
     7	    put "$seq revcom is {revcom($seq)}";
     8	}
$ ./module1.pl6 AACTAGAN
AACTAGAN revcom is NTCTAGTT
```

The "DNA.pm6" and this script live in the same directory.  For the same security reason as why your ```$PATH``` does not normally include your current working directory, Perl does not automatically include "." in your library path.  You either need to put your module somewhere in your ```$PERL6LIB``` path, or you need to ```use lib '.'```.  After that, then you can ```use DNA``` to bring in the "DNA" module exported code.

Here is how you might bring in the ```hamming``` function:

```
$ cat -n hamming.pl6
     1	#!/usr/bin/env perl6
     2
     3	use lib '.';
     4	use DNA;
     5
     6	sub MAIN (DNA $seq1, DNA $seq2) {
     7	    printf "Hamming distance from '%s' to '%s': %s\n",
     8	        $seq1, $seq2, hamming($seq1, $seq2);
     9	}
$ ./hamming.pl6 AACTAG CAAGAA
Hamming distance from 'AACTAG' to 'CAAGAA': 4
```

# OOPs, I did it again

Objects are another way to package up code.  The TLA (three-letter acronym) "OOP" stands for "object-oriented programming."  It's a way to couple data/state with functions that act on that data. 

To illustrate, let's model a "Person," first with a hash:

```
$ cat -n person1.pl6
     1	#!/usr/bin/env perl6
     2
     3	my %geddy = first_name => 'Geddy', last_name => 'Lee';
     4	my %alex  = first_name => 'Alex',  last_name => 'Leifson';
     5	my %neil  = first_name => 'Neil',  last_name => 'Peart';
     6
     7	for %geddy, %alex, %neil -> %person {
     8	    printf "%s %s\n", %person<first_name>, %person<last_name>;
     9	}
$ ./person1.pl6
Geddy Lee
Alex Leifson
Neil Peart
```

And now with objects:

```
$ cat -n person2.pl6
     1	#!/usr/bin/env perl6
     2
     3	class Person {
     4	    has Str $.first_name;
     5	    has Str $.last_name;
     6	}
     7
     8	my $geddy = Person.new(first_name => 'Geddy', last_name => 'Lee');
     9	my $alex  = Person.new(first_name => 'Alex',  last_name => 'Leifson');
    10	my $neil  = Person.new(first_name => 'Neil',  last_name => 'Peart');
    11
    12	for $geddy, $alex, $neil -> $person {
    13	    printf "%s %s\n", $person.first_name, $person.last_name;
    14	}
$ ./person2.pl6
Geddy Lee
Alex Leifson
Neil Peart
```

OK, pretty similar, so why go with objects when hashes could work just fine.  Well, for one thing, hashes won't enforce key names, and mistakes could be made:

```
$ cat -n person3.pl6
     1	#!/usr/bin/env perl6
     2
     3	my %geddy = first_name => 'Geddy', last_name => 'Lee';
     4	my %alex  = frist_name => 'Alex',  last_name => 'Leifson';
     5	my %neil  = first_name => 'Neil',  last_neme => 'Peart';
     6
     7	for %geddy, %alex, %neil -> %person {
     8	    printf "%s %s\n", %person<first_name>, %person<last_name>;
     9	}
[saguaro@~/work/metagenomics-book/perl6/oop]$ ./person3.pl6
Geddy Lee
Use of uninitialized value of type Any in string context.
Methods .^name, .perl, .gist, or .say can be used to stringify it to something meaningful.
  in block  at ./person3.pl6 line 7
Use of uninitialized value of type Any in string context.
Methods .^name, .perl, .gist, or .say can be used to stringify it to something meaningful.
  in block  at ./person3.pl6 line 7
 Leifson
Use of uninitialized value of type Any in string context.
Methods .^name, .perl, .gist, or .say can be used to stringify it to something meaningful.
  in block  at ./person3.pl6 line 7
Use of uninitialized value of type Any in string context.
Methods .^name, .perl, .gist, or .say can be used to stringify it to something meaningful.
  in block  at ./person3.pl6 line 7
Neil
```

Ugh.  Can you spot the typos in the code -- "frist_name" and "last_neme"?  Contrast with this:

```
$ cat -n person4.pl6
     1	#!/usr/bin/env perl6
     2
     3	class Person {
     4	    has Str $.first_name is required;
     5	    has Str $.last_name  is required;
     6	}
     7
     8	my $geddy = Person.new(first_name => 'Geddy', last_name => 'Lee');
     9	my $alex  = Person.new(frist_name => 'Alex',  last_name => 'Leifson');
    10	my $neil  = Person.new(first_name => 'Neil',  last_neme => 'Peart');
    11
    12	for $geddy, $alex, $neil -> $person {
    13	    printf "%s %s\n", $person.first_name, $person.last_name;
    14	}
$ ./person4.pl6
The attribute '$!first_name' is required, but you did not provide a value for it.
  in block <unit> at ./person4.pl6 line 9
```

Because we declared that the "first_name" and "last_name" are required attributes, we can't even run our code with those typos now.  We could also try misspelling the method (notice "list_name" in the ```for``` loop):

```
$ cat -n person5.pl6
     1	#!/usr/bin/env perl6
     2
     3	class Person {
     4	    has Str $.first_name is required;
     5	    has Str $.last_name  is required;
     6	}
     7
     8	my $geddy = Person.new(first_name => 'Geddy', last_name => 'Lee');
     9	my $alex  = Person.new(first_name => 'Alex',  last_name => 'Leifson');
    10	my $neil  = Person.new(first_name => 'Neil',  last_name => 'Peart');
    11
    12	for $geddy, $alex, $neil -> $person {
    13	    printf "%s %s\n", $person.first_name, $person.list_name;
    14	}
$ ./person5.pl6
No such method 'list_name' for invocant of type 'Person'
  in block <unit> at ./person5.pl6 line 12
```

What we're doing, obviously, is printing the full name for each member of the greatest rock band in the history of all mankind.  Everytime we want to do this, we have to manually concantenate the first and last names, or we could make our users (ourselves) duplicate the names into a "full_name" of the hash (very Bad Idea), or we could write a function to do it for us:

```
$ cat -n person6.pl6
     1	#!/usr/bin/env perl6
     2
     3	my %geddy = first_name => 'Geddy', last_name => 'Lee';
     4	my %alex  = first_name => 'Alex',  last_name => 'Leifson';
     5	my %neil  = first_name => 'Neil',  last_name => 'Peart';
     6
     7	sub full_name (%person) {
     8	    join ' ', %person<first_name>, %person<last_name>;
     9	}
    10
    11	for %geddy, %alex, %neil -> %person {
    12	    printf "Full Name: %s\n", full_name(%person);
    13	}
$ ./person6.pl6
Full Name: Geddy Lee
Full Name: Alex Leifson
Full Name: Neil Peart
```

Contrast with having the method being contained within the object:

```
$ cat -n person7.pl6
     1	#!/usr/bin/env perl6
     2
     3	class Person {
     4	    has Str $.first_name is required;
     5	    has Str $.last_name  is required;
     6	    method full_name { join ' ', $!first_name, $!last_name }
     7	}
     8
     9	my $geddy = Person.new(first_name => 'Geddy', last_name => 'Lee');
    10	my $alex  = Person.new(first_name => 'Alex',  last_name => 'Leifson');
    11	my $neil  = Person.new(first_name => 'Neil',  last_name => 'Peart');
    12
    13	for $geddy, $alex, $neil { put "Full Name: ", .full_name }
$ ./person7.pl6
Full Name: Geddy Lee
Full Name: Alex Leifson
Full Name: Neil Peart
```

Objects provide us the ability to make all sorts of requirements and validation of the data we're using.  Notice we required the first and last names to be the type ```Str```, and we could easily add "birthday" as a ```Date``` or even create our own type like ```Instrument```:

```
$ cat -n person8.pl6
     1	#!/usr/bin/env perl6
     2
     3	enum Instrument <Guitar Bass Drums>;
     4
     5	class Person {
     6	    has Str $.first_name is required;
     7	    has Str $.last_name  is required;
     8	    has Date $.birthday;
     9	    has Instrument $.instrument;
    10	    method full_name { join ' ', $!first_name, $!last_name }
    11	}
    12
    13	my Person $geddy .= new(first_name => 'Geddy', last_name => 'Lee',
    14	                    birthday => Date.new(1953, 6, 29), instrument => Bass);
    15	my Person $alex  .= new(first_name => 'Alex',  last_name => 'Leifson',
    16	                    birthday => Date.new(1953, 10, 27), instrument => Guitar);
    17	my Person $neil  .= new(first_name => 'Neil',  last_name => 'Peart',
    18	                    birthday => Date.new(1952, 9, 12), instrument => Drums);
    19
    20	for $geddy, $alex, $neil {
    21	    printf "%s (%s) born %s\n", .full_name, .instrument, .birthday;
    22	}
$ ./person8.pl6
Geddy Lee (Bass) born 1953-06-29
Alex Leifson (Guitar) born 1953-10-27
Neil Peart (Drums) born 1952-09-12
```

With this ```Person``` class, it is not possible to instantiate an object with a Integer for the ```birthday``` or any ```Instrument``` we have not enumerated.  While it may not seem like that big of a gain when you are manually creating the objects,  if you are reading in a file given to you by a collaborator, such data validation can help you clean and normalize your data.  Hashes will never give you that security.

Let's say someone new joins the band like Michael J. Fox.  How are we going to handle printing his full name?  Obviously we must include his middle initial, so we can expand the class to have such a field but make it optional with a default value of "".  We can leave the implementation of "full_name" to the object and continue with the data we had before:

```
$ cat -n person9.pl6
     1	#!/usr/bin/env perl6
     2
     3	enum Instrument <Guitar Bass Drums>;
     4
     5	class Person {
     6	    has Str $.first_name is required;
     7	    has Str $.middle_initial = '';
     8	    has Str $.last_name  is required;
     9	    has Date $.birthday;
    10	    has Instrument $.instrument;
    11	    method full_name {
    12	        sprintf "%s %s%s",
    13	        $!first_name,
    14	        $!middle_initial ?? $!middle_initial ~ ' ' !! '',
    15	        $!last_name
    16	    }
    17	}
    18
    19	my Person $geddy .= new(first_name => 'Geddy', last_name => 'Lee',
    20	                    birthday => Date.new(1953, 6, 29), instrument => Bass);
    21	my Person $alex  .= new(first_name => 'Alex',  last_name => 'Leifson',
    22	                    birthday => Date.new(1953, 10, 27), instrument => Guitar);
    23	my Person $neil  .= new(first_name => 'Neil',  last_name => 'Peart',
    24	                    birthday => Date.new(1952, 9, 12), instrument => Drums);
    25	my Person $mjfox .= new(first_name => 'Michael',  last_name => 'Fox',
    26	                    middle_initial => 'J.',
    27	                    birthday => Date.new(1961, 6, 9), instrument => Guitar);
    28
    29	for $geddy, $alex, $neil, $mjfox {
    30	    printf "%s (%s) born %s\n", .full_name, .instrument, .birthday;
    31	}
$ ./person9.pl6
Geddy Lee (Bass) born 1953-06-29
Alex Leifson (Guitar) born 1953-10-27
Neil Peart (Drums) born 1952-09-12
Michael J. Fox (Guitar) born 1961-06-09
```

Now let's take our code from the "DNA" module and turn it into an object.  We'll start really simply:

```
$ cat -n dna1.pl6
     1	#!/usr/bin/env perl6
     2
     3	class DNA is Str {}
     4
     5	sub MAIN (Str $seq) {
     6	    my $dna = DNA.new(value => $seq);
     7	    dd $dna;
     8	    printf "%s is %s characters long.\n", $dna, $dna.chars;
     9	}
$ ./dna1.pl6 GAAGACT
DNA $dna = "GAAGACT"
GAAGACT is 7 characters long.
```

Remember that our ```subset DNA``` was derived from ```Str```, so here we are inheriting from the ```Str``` class for our ```DNA``` module because it really is just a string.  By subclassing ```Str```, we get all the native String methods for free!  

There's at least one big problem with this object:

```
$ ./dna1.pl6 foo
DNA $dna = "foo"
```

That's not a valid string of DNA, so we need to fix that:

```
$ cat -n dna2.pl6
     1	#!/usr/bin/env perl6
     2
     3	class DNA is Str {}
     4
     5	sub MAIN (Str $seq) {
     6	    if so $seq.uc ~~ /^ <[ACGTN]>+ $/ {
     7	        my $dna = DNA.new(value => $seq);
     8	        dd $dna;
     9	    }
    10	    else {
    11	        put "'$seq' not a DNA sequence.";
    12	    }
    13	}
$ ./dna2.pl6 GACTAG
DNA $dna = "GACTAG"
$ ./dna2.pl6 foo
'foo' not a DNA sequence.
```

OK, that works, but it's really a bad implementation because the code to check whether the string is valid DNA belongs in the object like so:

```
$ cat -n dna3.pl6
     1	#!/usr/bin/env perl6
     2
     3	class DNA is Str {
     4	    multi method ACCEPTS (Str $seq) {
     5	        return $seq ~~ /^ :i <[ACGTN]>+ $/;
     6	    }
     7	}
     8
     9	sub MAIN (Str $seq) {
    10	    if $seq ~~ DNA {
    11	        my $dna = DNA.new(value => $seq);
    12	        dd $dna;
    13	    }
    14	    else {
    15	        put "'$seq' not a DNA sequence.";
    16	    }
    17	}
$ ./dna3.pl6 GGACT
DNA $dna = "GGACT"
$ ./dna3.pl6 foo
'foo' not a DNA sequence.
```

Now I can pattern-match on the ```DNA``` class because I implemented the ```ACCEPTS``` (https://docs.perl6.org/routine/ACCEPTS) method.  I used the ```multi``` keyword BECAUSE...?  

There is still a big problem with this version, though, because the check of the data is still happening outside the object.  We need to override the default ```new``` method to make sure we are getting valid data:

```
$ cat -n dna4.pl6
     1	#!/usr/bin/env perl6
     2
     3	class DNA is Str {
     4	    multi method ACCEPTS (Str $seq) {
     5	        return $seq.uc ~~ /^ :i <[ACGTN]>+ $/;
     6	    }
     7
     8	    method new (*%args) {
     9	        my $value = %args<value>.Str;
    10	        if $value !~~ DNA {
    11	            fail "'$value' not a DNA sequence.";
    12	        }
    13	        self.bless(|%args);
    14	    }
    15	}
    16
    17	sub MAIN (Str $seq) {
    18	    try {
    19	        my $dna = DNA.new(value => $seq);
    20	        dd $dna;
    21	        CATCH { default { .Str.say } }
    22	    }
    23	}
$ ./dna4.pl6 GGACTA
DNA $dna = "GGACTA"
$ ./dna4.pl6 foo
'foo' not a DNA sequence.
```

Now it is impossible to create a ```DNA``` object with invalid input because the ```new``` method will ```fail``` if the string does not match our regex.  You'll notice a new keyword ```self``` in the ```new``` function -- that references the object itself and is not preceded by a ```$``` (WHY?). Because of the ```fail```, I have introduced a ```try``` block in the calling code where the ```CATCH``` block is executed on any exceptions.

I'd like to be able to create my DNA object by just passing the string and not the Pair ```value => $seq```.  I can change the way the ```new``` constructor works:

```
$ cat -n dna5.pl6
     1	#!/usr/bin/env perl6
     2
     3	class DNA is Str {
     4	    multi method ACCEPTS (Str $seq) {
     5	        return $seq.uc ~~ /^ <[ACGTN]>+ $/;
     6	    }
     7
     8	    method new (Str $str) {
     9	        if $str.uc !~~ DNA {
    10	            fail "'$str' not a DNA sequence.";
    11	        }
    12
    13	        self.bless(value => $str);
    14	    }
    15	}
    16
    17	sub MAIN (Str $str) {
    18	    try {
    19	        my $dna = DNA.new($str);
    20	        dd $dna;
    21	        CATCH { default { .Str.say } }
    22	    }
    23	}
$ ./dna5.pl6 GAAACT
DNA $dna = "GAAACT"
$ ./dna5.pl6 foo
'foo' not a DNA sequence.
```

Now let's move our ```DNA``` class into a separate file like our earlier module, both to reduce the size of our program and to allow us to reuse it in another program:

```
$ cat -n DNA1.pm6
     1	class DNA is Str {
     2	    multi method ACCEPTS (Str $seq) {
     3	        return $seq.uc ~~ /^ <[ACGTN]>+ $/;
     4	    }
     5
     6	    method new (Str $str) {
     7	        if $str.uc !~~ DNA {
     8	            fail "'$str' not a DNA sequence.";
     9	        }
    10
    11	        self.bless(value => $str);
    12	    }
    13	}
$ cat -n dna6.pl6
     1	#!/usr/bin/env perl6
     2
     3	use lib '.';
     4	use DNA1;
     5
     6	sub MAIN (Str $str) {
     7	    try {
     8	        my $dna = DNA.new($str);
     9	        dd $dna;
    10	        CATCH { default { .Str.say } }
    11	    }
    12	}
$ ./dna6.pl6 GAACTG
DNA $dna = "GAACTG"
$ ./dna6.pl6 foo
'foo' not a DNA sequence.
```

Let's add some more functionality to our ```DNA``` class such as our ```revcom``` and ```hamming``` methods, a method to tell us the length of the DNA, and an attribute to let us know if we have the forward or reverse strand.  While we're at it, let's have both of our ```new``` methods to allow users to create a ```DNA``` object either with a single argument or a Pair.

```
$ cat -n DNA2.pm6
     1	enum Direction <Forward Reverse>;
     2
     3	class DNA is Str {
     4	    has Direction $.direction = Forward;
     5
     6	    multi method ACCEPTS (Str $seq) {
     7	        return $seq ~~ /^ :i <[ACGTN]>+ $/;
     8	    }
     9
    10	    # e.g., DNA.new($seq1);
    11	    multi method new (Str $seq) {
    12	        if $seq !~~ DNA {
    13	            fail "'$seq' not a DNA sequence.";
    14	        }
    15	        self.bless(value => $seq);
    16	    }
    17
    18	    # e.g., DNA.new(value => $seq1);
    19	    multi method new (*%args) {
    20	        my $value = %args<value>.Str;
    21	        if $value !~~ DNA {
    22	            fail "'$value' not a DNA sequence.";
    23	        }
    24	        self.bless(|%args);
    25	    }
    26
    27	    method revcom {
    28	        self.trans(<A C G T a c g t> => <T G C A t g c a>).flip;
    29	    }
    30
    31	    method length { self.chars }
    32
    33	    method hamming (DNA $other) {
    34	        return (self.chars - $other.chars).abs +
    35	               (self.comb Z $other.comb).grep({ $^a[0] ne $^a[1] });
    36	    }
    37	}
```

I created an ```enum``` type for the ```Direction``` that consists of the two values, ```Forward``` and ```Reverse```.  This is a type like we've seen before, and it allows me to constrain the new attribute ```direction``` to only those two values.  The constructor for that constraint:

```
has Direction $.direction = Forward;
```

Says that the class has a scalar value called ```direction``` that is of the type ```Direction``` and which has a default value of ```Forward```.  The ```has``` keyword will create accessor/mutator methods called ```direction``` for us to get (access) or change (mutate, if we so allow) the direction.  By default, object attributes are read-only, so we'd have to explicitly say ```is rw``` to declare that it is read-write.  Here read-only is the correct way to go because it is not something we should allow to be changed.

Here is how you can use the code:

```
$ cat -n dna7.pl6
     1	#!/usr/bin/env perl6
     2
     3	use lib '.';
     4	use DNA2;
     5
     6	sub MAIN (Str $str) {
     7	    try {
     8	        my $i    = 0;
     9	        my $temp = "%s: %s has the direction '%s' and length '%s'.\n";
    10
    11	        my $dna1 = DNA.new($str);
    12	        printf $temp, ++$i, $dna1, $dna1.direction, $dna1.length;
    13
    14	        my $dna2 = DNA.new(value => $str, direction => Forward);
    15	        printf $temp, ++$i, $dna2, $dna2.direction, $dna2.length;
    16
    17	        my $dna3 = DNA.new(value => $str, direction => Direction.pick);
    18	        printf $temp, ++$i, $dna3, $dna3.direction, $dna3.length;
    19
    20	        CATCH { default { .Str.say } }
    21	    }
    22	}
```

Notice at line 11 I create the ```DNA``` object with a single argument and no ```direction,``` but at line 14 I use two Pairs to define the ```value``` and ```direction```, while at line 17 I define the ```value``` and randomly pick a ```Direction```.  Here's what it looks like when it runs:

```
$ ./dna7.pl6 GATAGA
1: GATAGA has the direction 'Forward' and length '6'.
2: GATAGA has the direction 'Forward' and length '6'.
3: GATAGA has the direction 'Reverse' and length '6'.
$ ./dna7.pl6 foo
'foo' not a DNA sequence.
```

Here is how we could call our ```hamming``` and ```revcom``` functions:

```
$ cat -n hamming.pl6
     1	#!/usr/bin/env perl6
     2
     3	use lib '.';
     4	use DNA2;
     5
     6	sub MAIN (Str $seq1, Str $seq2) {
     7	    try {
     8	        my $dna1 = DNA.new($seq1);
     9	        my $dna2 = DNA.new($seq2);
    10	        put "Hamming distance from '$dna1' to '$dna2': ", $dna1.hamming($dna2);
    11	        CATCH { default { .Str.say } }
    12	    }
    13	}
$ ./hamming.pl6 GGATC GGCCC
Hamming distance from 'GGATC' to 'GGCCC': 2
$ cat -n revcom.pl6
     1	#!/usr/bin/env perl6
     2
     3	use lib '.';
     4	use DNA2;
     5
     6	sub MAIN (Str $str) {
     7	    try {
     8	        my $dna = DNA.new($str);
     9	        printf "Input : %s\nRevcom: %s\n", $dna, $dna.revcom;
    10	        CATCH { default { .Str.say } }
    11	    }
    12	}
[saguaro@~/work/metagenomics-book/perl6/oop]$ ./revcom.pl6 GATTAGA
Input : GATTAGA
Revcom: TCTAATC
```

And remember that, since we inherited from ```Str```, we get all those methods, too!

```
$ cat -n substr.pl6
     1	#!/usr/bin/env perl6
     2
     3	use lib '.';
     4	use DNA2;
     5
     6	sub MAIN (Str :$seq, Int :$start=0, Int :$stop=0) {
     7	    try {
     8	        my $dna = DNA.new($seq);
     9	        $stop ||= $dna.chars;
    10	        put "DNA from $start to $stop: ", $dna.substr($start, $stop);
    11	        CATCH { default { .Str.say } }
    12	    }
    13	}
$ ./substr.pl6 --seq=GGATACC --start=2 --stop=5
DNA from 2 to 5: ATACC
```

The main idea behind both modules and objects is to hide complexity from the user (even if the user is just you).  You package up everything belonging to DNA or whatever into a file or module or object, and the code that uses it doesn't have to worry about how the algorigthms or objects or whatever is implemented.  

# Hangman

I think lots of examples are the way to teach, and probably you're tired of just thinking about DNA by now.  So let's create a "Puzzle" object for playing "Hangman."  Here's how a game looks when won:

```
$ ./hangman.pl6
puzzle = _ _ _ _ _ _ _ []
What is your guess? a
puzzle = _ _ _ _ _ _ _ [a]
What is your guess? i
puzzle = _ _ _ _ _ _ _ [ai]
What is your guess? e
puzzle = _ e _ _ _ e _ [aei]
What is your guess? o
puzzle = _ e _ _ _ e _ [aeio]
What is your guess? u
puzzle = _ e _ _ u e _ [aeiou]
What is your guess? s
puzzle = _ e s _ u e _ [aeiosu]
What is your guess? t
puzzle = _ e s _ u e _ [aeiostu]
What is your guess? r
puzzle = r e s _ u e r [aeiorstu]
What is your guess? c
puzzle = r e s c u e r [aceiorstu]
You won!
```

The game needs to have a random dictionary word which we can easily pluck from "/usr/share/dict/words" of some minimum (default 5) and maximum (default 9) length.  During the game, we need to know the word, what the user has guessed so far, and how many of the letters have been found.  After each guess, we need to decide if the puzzle has been solved or if the user has exceeded the allowed number of attempts (default 10).  While it's not at all necessary to use a new type/object to handle this, we can see quite a benefit to wrapping all the puzzle logic up in one place and the user interface in another.  Here's the code:

```
#!/usr/bin/env perl6

class Puzzle {
    has Str $.word;
    has Str @.state;
    has Int %.guesses;

    submethod BUILD (Str :$word) {
        $!word  = $word;
        @!state = '_' xx $word.chars;
    }

    method Str {
        "puzzle = {@.state.join(' ')} [{%.guesses.keys.sort.join}]";
    }

    method guess (Str $char) {
        unless %.guesses{ $char }++ {
            for $.word.indices($char) -> $i {
                @.state[$i] = $char;
            }
        }
    }

    method was-guessed (Str $char) {
        return %.guesses{ $char }.defined;
    }

    method number-guessed {
        return %.guesses.keys.elems;
    }

    method is-solved {
        none(@.state) eq '_';
    }
}

sub MAIN(Int :$num-guesses = 10, :$min-word-len=5, :$max-word-len=9) {
    my $words  = '/usr/share/dict/words';
    my $word   = $words.IO.lines.grep(
                 {$min-word-len <= .chars <= $max-word-len}).pick.lc;
    my $puzzle = Puzzle.new(word => $word);

    loop {
        put ~$puzzle;
        if $puzzle.is-solved {
            put "You won!";
            last;
        }

        if $puzzle.number-guessed >= $num-guesses {
            put "Too many guesses. The word was '$word\.' You lose.";
            last;
        }

        my $guess = (prompt "What is your guess? ").lc;
        if $guess !~~ m:i/^<[a..z]>$/ {
            put "Please guess just one letter";
            next;
        }

        if $puzzle.was-guessed($guess) {
            put "You guessed that before!";
            next;
        }

        $puzzle.guess($guess);
    }
}
```

The ```Puzzle``` holds the current state of the game.  It knows the word which is being guessed, which letters have been guessed so far (and therefore how many guesses have been made), and which letters have been found.  When we ```loop``` through the game, we just need to ask the ```Puzzle``` for information like if the game is over either because too many guesses were made or because the puzzle has been solved.  The game could be written entirely without objects, but objects give us a way to wrap up state -- how things change through time -- with methods -- how to change things, how to ask about change.

# Bouncy Balls

Here's a simple game that bounces balls inside a box in your terminal.  When two balls collide, they explode with a Unicode star "★" and disappear from the board:

```
     1	#!/usr/bin/env perl6
     2	
     3	subset PosInt of Int where * > 0;
     4	
     5	class Ball {
     6	    has Int $.rows;
     7	    has Int $.cols;
     8	    has Int $.row is rw = (2..^$!rows).pick;
     9	    has Int $.col is rw = (2..^$!cols).pick;
    10	    has Int $.horz-dir is rw = (1, -1).pick;
    11	    has Int $.vert-dir is rw = (1, -1).pick;
    12	
    13	    method Str { join ',', $!row, $!col }
    14	
    15	    method pos { ($.row, $.col) }
    16	
    17	    method move {
    18	        $!col += $!horz-dir;
    19	        $!row += $!vert-dir;
    20	        $!horz-dir *= -1 if $!col <= 1 || $!col >= $!cols;
    21	        $!vert-dir *= -1 if $!row <= 1 || $!row >= $.rows;
    22	    }
    23	}
    24	
    25	my $DOT           = "\x25A0"; # ■
    26	my $STAR          = "\x2605"; # ★
    27	my $SMILEY-FACE   = "\x263A"; # ☺
    28	my ($ROWS, $COLS) = qx/stty size/.words;
    29	
    30	sub MAIN (
    31	    PosInt :$rows=$ROWS - 4,
    32	    PosInt :$cols=$COLS - 2,
    33	    PosInt :$balls=10,
    34	    Numeric :$refresh=.075,
    35	    Bool :$smiley=False,
    36	) {
    37	    print "\e[2J";
    38	    my Str $bar    = '+' ~ '-' x $cols ~ '+';
    39	    my $icon       = $smiley ?? $SMILEY-FACE !! $DOT;
    40	    my Ball @balls = Ball.new(:$rows, :$cols) xx $balls;
    41	
    42	    loop {
    43	        .move for @balls;
    44	
    45	        my $positions = (@balls».Str).Bag;
    46	
    47	        my %row;
    48	        for $positions.list -> (:$key, :$value) {
    49	            my ($row, $col) = $key.split(',');
    50	            %row{ $row }.append: $col => $value;
    51	        }
    52	
    53	        print "\e[H";
    54	        my $screen = "$bar\n";
    55	
    56	        for 1..$rows -> $this-row {
    57	            my $line = '|' ~ " " x $cols;
    58	            if %row{ $this-row }:exists {
    59	                for %row{ $this-row }.list -> (:$key, :$value) {
    60	                    $line.substr-rw($key, 1) = $value == 1 ?? $icon !! $STAR;
    61	                }
    62	            }
    63	
    64	            $screen ~= "$line|\n";
    65	        }
    66	
    67	        $screen ~= $bar;
    68	        put $screen;
    69	        sleep $refresh;
    70	        my @collisions = $positions.grep(*.value > 1).map(*.key);
    71	        @balls = @balls.grep(none(@collisions) eq *.Str);
    72	    }
    73	}
```

While it's not a requirement to use objects for this game, they do allow me to wrap up the state of each ball into a little package that I can interact with by saying ```move``` to get the ball to figure: 

1. know its current location
2. know its current direction (up/down, left/right)
3. reverse its course if it encounters a boundary

Once I've ```move```d all the balls, I can get all their positions and draw them to the screen.  I need to know if more than one ball occupies any location, so I ```Bag``` them up to get counts.  If a position has more than one ball, I use the star icon to indicate the collision and remove the offenders from the ```@balls``` array.