# Tests

You can easily write a test suite in Perl using the "Test" module.  The DNA test script checks that the script prints a "usage"-like statement when run with no arguments or if it prints the correct output given a known sequence from either the command line or a file.  You can look at the "test.pl6" script to understand more how it works.  You should totally write tests.

One methodology to writing software is to actually write the tests *first* (called "test-driven development" or "TDD") and then write the software passes the tests.  It's a good way to define at least the behavior of a program, e.g., the expected output for a given input.  This is another reason I would encourage you to create a ```MAIN``` entry point with named parameters as it forces you to consider what you want from the user.  

# Testing script output

As a simple example, let's look at the test for the parallel file reader:

```
$ cat test.pl6
#!/usr/bin/env perl6

use Test;

my $expected = q:to/END/;
Sequence_1
1	2	3	4
A	B	C	D
Sequence_2
5	6	7	8
E	F	G	H
END

ok my $proc =
    run("./parallel.pl6", "--phred=phred.txt", "--seq=seq.txt", :out),
    "Ran script";

is $proc.out.slurp-rest, $expected, "Got expected output";

done-testing();
```

My first test is just to see if I'm able to ```run``` the script with a couple of known input files, and so I use ```ok``` to find out if the ```Proc``` (https://docs.perl6.org/type/Proc) was created.  Next I ask ```is``` the standard out of the process the same as the expected text.  

I usually like to include a Makefile to assist me in running and testing my scripts:

```
$ cat Makefile
run:
	./parallel.pl6 --phred=phred.txt --seq=seq.txt

test:
	./test.pl6
```    
   
If I run ```make``` with no target, the first target is run:

```   
$ make
./parallel.pl6 --phred=phred.txt --seq=seq.txt
Sequence_1
1	2	3	4
A	B	C	D
Sequence_2
5	6	7	8
E	F	G	H
```

And it's so very satisfying to run ```make test``` and see all those "ok"s:

```
$ make test
./test.pl6
ok 1 - Ran script
ok 2 - Got expected output
1..2
```

# Test libraries

Testing scripts is a bit laborious and tricky as they are usually designed to complete some monolithic tasks, so you are often limited to giving some input and checking that the output was what you expect.  As you move your code into libraries and modules to aid in sharing, you'll find it's much easier to write tests for each particular method.  In my blackjack example, each object (Card, Deck, Player, Game) had complex interactions, and I quickly found I wanted to test each part individually.  By moving the ```class``` code into a module, I could ```use``` any part and test them in isolation.

# Use tests outside of test suites

Tests don't have to be limited to test suites for your scirpts or libraries.  In this example, I needed to ensure that I had completely downloaded all the files from a collaborator by using MD5 checksums.  That's a test, so I decided to use the ```is``` assertion for an elegant solution and obvious output.

Perl was created for systems administration, and Perl 6 has all the chops you've come to expect from the brand. Here I needed to use MD5 checksums from my collaborator to verify that I downloaded all their data without errors. Each data "$file" has an accompanying "$file.md5" that looks like this:

```
$ cat HOT232_1_0770m/prodigal.gff.md5
a36e4adfaa62cc4adb8cea44c4f7825f  HOT232_1_0770m/prodigal.gff
```

So I need to read the contents of this file, get just the first field, then execute my local "md5" (or "md5sum") program on the file without the ".md5" extension and determine if they are the same. All standard stuff, and I think Perl 6 gives us elegant ways to accomplish all of these, including a dead-simple testing framework. Here's my solution:

```
#!/usr/bin/env perl6

use File::Find;
use Test;

sub MAIN (Str :$dir=~$*CWD, Str :$md5="md5sum", Str :$ext='.md5') {
    my $rx = rx/$ext $/;
    for find(:dir($dir), :name($rx)) -> $md5-file {
        my $basename    = $md5-file.basename;
        my ($remote, $) = $md5-file.slurp.split(' ');
        my $local-file  = $basename.subst($rx, '');
        my $path        = $*SPEC.catfile(
                          $md5-file.dirname, $local-file);
        my ($local, $)  = run($md5, $path, :out)
                          .out.slurp-rest.split(' ');
        is $local, $remote, $md5-file;
    }

    done-testing();
}
```

Here I'll default to looking in the current directory which is accessible as the global variable ```$*CWD```.  Remember that this variable uses a ```$*``` "twigle" (two sigils) to denote a global variable which in this case is actually an IO object. I need to stringify it either by putting it in double-quotes or coercing it with the ```~``` operator. The "md5" argument is the name of my local "md5sum" binary which is often called just "md5." Lastly, I assume the file extension of the remote checksums is ".md5."

Users of Perl 5 will be pleased to note that "File::Find" is available. I was always more of a fan of "File::Find::Rule," I think the interface for the Perl 6 module is quite nice. The output is a list of IO::Path objects.

As a side note, I first wrote it like this:

```
for (find(:dir($dir), :name(rx/$ext $/))).kv -> $i, $md5-file {
    printf "%3d: %s\n", $i + 1, $md5-file;
}
```

Because I like to know as each file is being processed and how many have gone by. Take any list in Perl 6 and call ```.kv``` (key-value) on it to get the index (position) and the value:

```
> my @list = "foo", "bar", "baz"
[foo bar baz]
> @list.kv
(0 foo 1 bar 2 baz)
> for @list.kv -> $key, $val { put "$key = $val" }
0 = foo
1 = bar
2 = baz
```

Reading an entire file is done by calling slurp on it. In my case, there's only one line, and I want to immediately split it on spaces:

```
my ($remote, $) = $md5-file.slurp.split(' ');
```

Since I'm only interested in the first field (the second one, remember, is the name of the file, which I already know), I can ignore it by assigning the position to an anonymous scalar ```$```. To remove the extension, I use the ```Str.subst``` (substitute) method with the regular expression I put into the ```$rx``` variable. (The first argument to ```subst``` can also be a string.) So, if the $md5-file is "/work/03137/kyclark/ohana/HOT/HOT237_2_1000m/readpool.fastq.md5", then the basename is "readpool.fastq.md5" and the result of the ```subst``` is "readpool.fastq":

```
my $local-file = $basename.subst($rx, '');
```

To piece together the local file that needs to be tested, I can use the global ```$*SPEC``` object to get access to OS-dependent methods like ```catfile``` that will use the proper directory separators to construct the path. Again, the ```$*``` is a twigle to set the global variable/object apart from everything else. The local file should be in the same directory as the MD5 checksum file.

```
my $path = $*SPEC.catfile($md5-file.dirname, $local-file);
```

To get the local checksum, I need to run the correct "md5" binary and then read STDOUT from that program. (To capture STDERR, I could put ":err" in the call.) I put "backticks" in the title to highlight that this is now the way to call external programs and capture the output. Like the MD5 file, I'm only interested in the first field of output and can throw away the rest. I can accomplish it all in one line:

```
my ($local, $) = run($md5, $path, :out).out.slurp-rest.split(' ');
```

Finally, I brought in the "Test" module so that I can ask:

```
is $local, $remote, $md5-file;
```

If they are equal, I get output like this:

```
$ ./check-md5.pl6
ok 1 - /work/03137/kyclark/ohana/HOT/HOT224_1_0025m/prodigal.gff.md5
ok 2 - /work/03137/kyclark/ohana/HOT/HOT224_1_0025m/readpool.fastq.md5
ok 3 - /work/03137/kyclark/ohana/HOT/HOT224_1_0025m/ribosomal_rRNA.gff.md5
...
```

When they aren't, I see this:

```
not ok 193 - /work/03137/kyclark/ohana/HOT/HOT224_1_0045m/readpool.fastq.md5

# Failed test '/work/03137/kyclark/ohana/HOT/HOT224_1_0045m/readpool.fastq.md5'
# at ./check-md5.pl6 line 13
# expected: '4b49aaa2b0c60342e90cf37e30c30886'
#      got: '4cdd81767c2ffed712c0b8ffb6de8b38'
```

And so I know to get that file again!


