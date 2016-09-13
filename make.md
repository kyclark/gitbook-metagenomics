# Organizing with Makefiles

GNU "make" (https://www.gnu.org/software/make/) will help you organize your code and testing.  You only need to learn a few things about make to be productive. 

First, you create text file called "Makefile" (or "makefile").  Whenever you run something on the command line that you might want to run again, you should probably put it into your Makefile so you don't have to go searching through your history to find the magic incantation of some command that actually did what you wanted.  Don't be like me and think "It's totally OK to leave my sunglasses in the driver's seat because I definitely will not forget they are there and then sit on them when I come back to my car."  Don't trust your memory, just add it to you Makefile.

If you check out "github.com:kyclark/metagenomics-book.git", you will find a "problems/dna" directory with a finished version of the "dna.pl6" program along with sample data, a test script, and a Makefile that looks like this

```
$ cat -n Makefile
     1	run:
     2		./dna.pl6 AGCTTTTCATTCTGACTGCAACGGGCAATATGTCTCTGTGTGGATTAAAAAAAGAGTGTCTGATAGCAGC
     3
     4	file:
     5		./dna.pl6 test.txt
     6
     7	test:
     8		./test.pl6
```

If you type ```make```, you should see this:

```
$ make
./dna.pl6 AGCTTTTCATTCTGACTGCAACGGGCAATATGTCTCTGTGTGGATTAAAAAAAGAGTGTCTGATAGCAGC
20 12 17 21
```

Without any arguments, ```make``` will execute the first "target" which is basically a word-ish thing followed by a colon, which in this case is "run."  If you ```make run```, you will see the same thing.  The ```make file``` target executes the script with a file as the input rather than a string, and ```make test``` will run the "test.pl6" script:

```
$ make test
./test.pl6
1..3
ok 1 - No args gives usage
ok 2 - Correct count from command line
ok 3 - Correct count from file
$ make run
./dna.pl6 AGCTTTTCATTCTGACTGCAACGGGCAATATGTCTCTGTGTGGATTAAAAAAAGAGTGTCTGATAGCAGC
20 12 17 21
```

# Writing tests

You can easily write a test suite in Perl using the "Test" module.  The DNA test script checks that the script prints a "usage"-like statement when run with no arguments or if it prints the correct output given a known sequence from either the command line or a file.  You can look at the "test.pl6" script to understand more how it works.  You should totally write tests.

One methodology to writing software is to actually write the tests *first* (called "test-driven development" or "TDD") and then write the software passes the tests.  It's a good way to define at least the behavior of a program, e.g., the expected output for a given input.  This is another reason I would encourage you to create a ```MAIN``` entry point with named parameters as it forces you to consider what you want from the user.  