# Add It Up

Let's persue the ```MAIN``` signature and discussion of types a bit further with a program that will add two numbers:

```
$ cat -n adder1.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	use v6;
     4
     5 	sub MAIN (Int $a!, Int $b!) { put $a + $b }
     6
     7 	sub USAGE {
     8 	    printf "  %s <Int> <Int>\n", $*SPEC.basename($*PROGRAM-NAME);
     9 	}
$ cat -n adder1.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	use v6;
     4
     5 	sub MAIN (Int $a!, Int $b!) { put $a + $b }
     6
     7 	sub USAGE {
     8 	    printf "  %s <Int> <Int>\n", $*SPEC.basename($*PROGRAM-NAME);
     9 	}
$ ./adder1.pl6
  adder1.pl6 <Int> <Int>
$ ./adder1.pl6 foo bar
  adder1.pl6 <Int> <Int>
$ ./adder1.pl6 2 4
6
$ ./adder1.pl6 2 4.8
  adder1.pl6 <Int> <Int>
```

At line 5, I'm declaring two required integer variables called ```$a``` and ```$b```.  If the user provides any arguments that don't match that signature, then we've seen how Perl 6 will generate a decent usage statement.  Here I've thrown in my own ```USAGE``` method to show how you can create your own, much like the ```HELP``` that we created in bash.