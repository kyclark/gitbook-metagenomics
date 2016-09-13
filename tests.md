# Tests

You can easily write a test suite in Perl using the "Test" module.  The DNA test script checks that the script prints a "usage"-like statement when run with no arguments or if it prints the correct output given a known sequence from either the command line or a file.  You can look at the "test.pl6" script to understand more how it works.  You should totally write tests.

One methodology to writing software is to actually write the tests *first* (called "test-driven development" or "TDD") and then write the software passes the tests.  It's a good way to define at least the behavior of a program, e.g., the expected output for a given input.  This is another reason I would encourage you to create a ```MAIN``` entry point with named parameters as it forces you to consider what you want from the user.  
