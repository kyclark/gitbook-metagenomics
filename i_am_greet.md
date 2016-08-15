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

So what is this ```MAIN``` thing?  It's a subroutine that gets run automatically with the arguments to the script (https://docs.perl6.org/language/functions#index-entry-MAIN).  The great masters of old passed down the wisdom that a good starting point for programs should be a function called "main," and so we do.

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
$ ./greet3.pl6 --name=Wilbur --greeting=Salutations
Salutations, Wilbur
```

By putting a colon ```:``` in front of the argument names, we used a shorthand to create a ```Pair``` type (https://docs.perl6.org/type/Pair).  This was much easier than all that ```getopt``` business in bash, and naming the arguments lets the user specify them in any order desired.  The ```!``` at the end of ```:$greeting!``` indicates that this is a required argument, while the ```=``` after ```$name``` indicates a default value for an optional argument.

Did you notice also that the generated help included a hint as the type of data wanted by each argument?  The ```(Any)``` is a data type (https://docs.perl6.org/type.html) in Perl 6 at the top of the type hierarchy (https://docs.perl6.org/type/Any).  We could indicate that we want strings by adding a type to the signature:

```
$ cat -n greet4.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	sub MAIN (Str :$greeting!, Str :$name='Stranger') {
     4 	    put "$greeting, $name";
     5 	}
$ ./greet4.pl6
Usage:
  ./greet4.pl6 --greeting=<Str> [--name=<Str>]
$ ./greet4.pl6 --greeting="Top o' the morning"Top o' the morning, Stranger
$ ./greet4.pl6 --greeting="Top o' the morning" --name=11
Top o' the morning, 11
```

We got off to a nice start there with the USAGE telling us to provide strings, so why didn't it complain about "11"?  The problem is that everything coming from the command line looks like a string.  There are ways to handle this, but they can't be so easily fixed.  We'll come back to this later.