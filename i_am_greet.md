# I Am Greet

Let's revisit our greeting script and see how we can do much better than bash-Perl.  

```
$ cat -n greet2.pl6
     1	#!/usr/bin/env perl6
     2
     3	sub MAIN ($greeting, $name='Stranger') {
     4	    put "$greeting, $name";
     5	}
$ ./greet2.pl6
Usage:
  ./greet2.pl6 <greeting> [<name>]
$ ./greet2.pl6 --help
Usage:
  ./greet2.pl6 <greeting> [<name>]
$ ./greet2.pl6 Hello
Hello, Stranger
$ ./greet2.pl6 Hello Kenny
Hello, Kenny
```

Just look at that script.  Just look at it!  Do you see how much disappeared, and yet we still have all the functionality that we wrote before?!  We have a usage statement, we have required and optional arguments, we have variable assignment, we have sane defaults, we have clean syntax. 

Now, let's add named arguments:

```
$ cat -n greet3.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	sub MAIN (:$greeting!, :$name='Stranger') {
     4 	    put "$greeting, $name";
     5 	}
$ ./greet3.pl6
Usage:
  ./greet3.pl6 --greeting=<Any> [--name=<Any>]
$ ./greet3.pl6 --greeting=Salutations
Salutations, Stranger
$ ./greet3.pl6 --greeting=Salutations --name=Wilbur
Salutations, Wilbur
```

By putting a colon ```:``` in front of the argument names, we used a shorthand to create a ```Pair``` type (https://docs.perl6.org/type/Pair).  This is what the long-hand version would look like:

```
$ cat -n greet4.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	sub MAIN (:$greeting!, :$name='Stranger') {
     4 	    put "$greeting, $name";
     5 	}
```

