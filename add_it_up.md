# Add It Up

Let's pursue the ```MAIN``` signature and discussion of types a bit further with a program that will add two numbers:

```
$ cat -n adder1.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	sub MAIN (Int $a!, Int $b!) { put $a + $b }
     4
     5 	sub USAGE {
     6 	    printf "  %s <Int> <Int>\n", $*SPEC.basename($*PROGRAM-NAME);
     7 	}
$ ./adder1.pl6
  adder1.pl6 <Int> <Int>
$ ./adder1.pl6 foo bar
  adder1.pl6 <Int> <Int>
$ ./adder1.pl6 2 4
6
$ ./adder1.pl6 2 4.8
  adder1.pl6 <Int> <Int>
```

At line 3, I'm declaring two required integer variables called ```$a``` and ```$b```.  If the user provides any arguments that don't match that signature, then we've seen how Perl will generate a decent usage statement.  Here I've thrown in my own ```USAGE``` method to show how you can create your own, exactly like we created in bash.  It gets run automatically on bad input.

But now we have a problem in that we'd like to be able to add anything that looks like a number, and ```Int``` won't allow the argument "4.8."  We can move up the type hierarchy to ```Numeric``` to be more generic:

```
$ cat -n adder2.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	sub MAIN (Numeric $a!, Numeric $b!) { put $a + $b }
     4
     5 	sub USAGE {
     6 	    note sprintf "  %s <Numeric> <Numeric>", $*SPEC.basename($*PROGRAM-NAME);
     7 	}
$ ./adder2.pl6 2>err
$ cat err
  adder2.pl6 <Numeric> <Numeric>
$ ./adder2.pl6 2.71828 3.14159
5.85987
$ ./adder2.pl6 2.71828 5.4e+3
5402.71828
```

Something else I added was the ```note``` in ```USAGE``` so that the usage statement is printed to STDERR (standard error).  You can read about other useful I/O (input/output) methods at https://docs.perl6.org/type/IO.

What I like about creating descriptive signatures for ```MAIN``` is that it helps us define what inputs we require and communicates the requirements efficiently to the user.  You are encouraged to always use a ```MAIN``` entry point when possible.