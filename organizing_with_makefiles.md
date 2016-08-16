# Organizing with Makefiles

GNU "make" (https://www.gnu.org/software/make/) will help you organize your code and testing.  I'm going to introduce it here as we'll use it in the next section to write a script.  You only need to learn a few things about make to be productive. 

First, you create text file called "Makefile" (or "makefile").  Whenever you run something on the command line that you might want to run again, you should probably put it into your Makefile so you don't have to go searching through your history to find the magic incantation of some command that actually did what you wanted.  Don't be like me and think "It's totally OK to leave my sunglasses in the driver's seat because I definitely will not forget they are there and then sit on them when I come back to my car."  Don't think "I'll be able to remember that command after lunch."  Just add it to you Makefile! 

If you check out "github.com:kyclark/abe487.git", you will find a "book/problems/dna" directory with a finished version of the "dna.pl6" program along with sample data, a test script, and a Makefile that you can use to see this:

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

If you run "make" with no target name, it will run the first target which is "test."  

# Writing tests

You can easily write a test suite in Perl using the "Test" module.  The DNA test script checks that the script prints a "usage"-like statement when run with no arguments or if it prints the correct output given a known sequence from either the command line or a file.  You can look at the "test.pl6" script to understand more how it works.  You should totally write tests.  Tests are good, M'K?