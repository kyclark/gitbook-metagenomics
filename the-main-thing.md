# The MAIN Thing

In the last chapter, we started playing with the `MAIN` subroutine to get our arguments.  Let's explore that a bit more.

If you're coming to Perl 6 from Perl 5, the global variable `@*ARGS` \([https://docs.perl6.org/language/variables\\#index-entry-%40%2AARGS\](https://docs.perl6.org/language/variables\#index-entry-%40%2AARGS\)\) will look familiar to you as the place to get the command-line arguments to your program:

```
$ cat -n main1.pl6
     1    #!/usr/bin/env perl6
     2
     3    put "ARGS = ", @*ARGS.join(', ');
$ ./main1.pl6 foo bar baz
ARGS = foo, bar, bar
```

The `@` is the sigil that denotes the variable as an array, and the `*` is the "twigle" that denotes that the variable is a global.  If you follow the above link to the `@*ARGS` documentation, you'll find a whole host of other dynamic and environmental variables like user, hostname, PID, cwd, etc.

Dealing directly with `@*ARGS` is fine your program accepts a few positional arguments where the first argument means one thing \(name\), the second argument another thing \(rank\), the third another \(serial number\), etc:

```
$ cat -n main2.pl6
     1    #!/usr/bin/env perl6
     2
     3    my ($name, $rank, $serial-num) = @*ARGS;
     4    put "name ($name) rank ($rank) serial number ($serial-num)";
$ ./main2.pl6 Patch Private 1656401
name (Patch) rank (Private) serial number (1656401)
```

Or if all the arguments are homogenous \(e.g., a list of files to process\), this works reasonably well:

```
$ cat -n main3.pl6
     1    #!/usr/bin/env perl6
     2
     3    for @*ARGS -> $file {
     4        put "Processing '$file'";
     5    }
     6
     7    put "Done."
[saguaro@~/work/perl6/main]$ ./main3.pl6 foo bar baz
Processing 'foo'
Processing 'bar'
Processing 'baz'
Done.
```

If we write the special `MAIN` \([https://docs.perl6.org/language/functions\\#index-entry-MAIN](https://docs.perl6.org/language/functions\#index-entry-MAIN\)\) subroutine, we can automatically assign the variables instead of taking them from `@*ARGS`:

```
$ cat -n main4.pl6
     1    #!/usr/bin/env perl6
     2
     3    sub MAIN ($name, $rank, $serial-num) {
     4        put "name ($name) rank ($rank) serial number ($serial-num)";
     5    }
```

Perl will enforce the number of arguments and generate help when supplied the wrong number or when requested:

```
$ ./main4.pl6 Patch Private
Usage:
  ./main4.pl6 <name> <rank> <serial-num>
$ ./main4.pl6 Patch Private 1656401
name (Patch) rank (Private) serial number (1656401)
$ ./main4.pl6 -h
Usage:
  ./main4.pl6 <name> <rank> <serial-num>
$ ./main4.pl6 --help
Usage:
  ./main4.pl6 <name> <rank> <serial-num>
```

You can also add type \([https://docs.perl6.org/type.html](https://docs.perl6.org/type.html\)\) constraints:

```
$ cat -n main5.pl6
     1    #!/usr/bin/env perl6
     2
     3    sub MAIN (Str $name, Str $rank, Int $serial-num) {
     4        put "name ($name) rank ($rank) serial number ($serial-num)";
     5    }
$ ./main5.pl6 foo bar baz
Usage:
  ./main5.pl6 <name> <rank> <serial-num>
$ ./main5.pl6 foo bar 123
name (foo) rank (bar) serial number (123)
```

To make them named arguments, simply prefix with a `:` and then you can supply the arguments in any order:

```
$ cat -n main6.pl6
     1    #!/usr/bin/env perl6
     2
     3    sub MAIN (Str :$name, Str :$rank, Int :$serial-num) {
     4        put "name ($name) rank ($rank) serial number ($serial-num)";
     5    }
$ ./main6.pl6 -h
Usage:
  ./main6.pl6 [--name=<Str>] [--rank=<Str>] [--serial-num=<Int>]
$ ./main6.pl6 --name=Patch --serial-num=1656401 --rank=Private
name (Patch) rank (Private) serial number (1656401)
```

There's a problem if I don't supply any arguments to this last version:

```
$ ./main6.pl6
Use of uninitialized value of type Str in string context.
Methods .^name, .perl, .gist, or .say can be used to stringify it to something meaningful.
  in sub MAIN at ./main6.pl6 line 3
Use of uninitialized value of type Str in string context.
Methods .^name, .perl, .gist, or .say can be used to stringify it to something meaningful.
  in sub MAIN at ./main6.pl6 line 3
Use of uninitialized value of type Int in string context.
Methods .^name, .perl, .gist, or .say can be used to stringify it to something meaningful.
  in sub MAIN at ./main6.pl6 line 3
name () rank () serial number ()
```

Perl is complaining that it can't print uninitialized values. I didn't declare that any of the arguments are required -- which I can do by post-fixing the variables with `!` -- nor did I provide defaults for those that are not -- which I can do with `=`:

```
$ cat -n main7.pl6
     1    #!/usr/bin/env perl6
     2
     3    sub MAIN (Str :$name!, Str :$rank='NA', Int :$serial-num=0) {
     4        put "name ($name) rank ($rank) serial number ($serial-num)";
     5    }
$ ./main7.pl6
Usage:
  ./main7.pl6 --name=<Str> [--rank=<Str>] [--serial-num=<Int>]
```

The absence of `[]` around the `--name` argument indicates to the user that the argument is required, while `--rank` and `--serial-num` are optional:

```
$ ./main7.pl6 --name=Patch
name (Patch) rank (NA) serial number (0)
$ ./main7.pl6 --name=Patch --serial-num=1656401 --rank=Private
name (Patch) rank (Private) serial number (1656401)
```

Even though Perl is checking the types \(string, integer\) for our arguments, it's not enforcing any validation on the value. We can add a where clause \([https://docs.perl6.org/type/Signature\\#index-entry-where\_clause\_%28Signature%29\](https://docs.perl6.org/type/Signature\#index-entry-where\_clause\_%28Signature%29\)\) to ensure we only get a positive value for `--serial-num`:

```
$ cat -n main8.pl6
     1    #!/usr/bin/env perl6
     2
     3    sub MAIN (Str :$name!, Str :$rank='NA', Int :$serial-num where * > 0 = 1) {
     4        put "name ($name) rank ($rank) serial number ($serial-num)";
     5    }
$ ./main8.pl6 --name=Patch --serial-num=-10
Usage:
  ./main8.pl6 --name=<Str> [--rank=<Str>] [--serial-num=<Int>]
$ ./main8.pl6 --name=Patch
name (Patch) rank (NA) serial number (1)
```

As it turns out, the built-in `UInt` \([https://docs.perl6.org/type/UInt\](https://docs.perl6.org/type/UInt\)\) type will work for ensuring positive, so maybe we would be better off ensuring that the serial number is of the correct length:

```
$ cat -n main9.pl6
     1    #!/usr/bin/env perl6
     2
     3    sub MAIN (
     4        Str :$name!,
     5        Str :$rank='NA',
     6        UInt :$serial-num where *.Str.chars == 7 = 1111111
     7    ) {
     8        put "name ($name) rank ($rank) serial number ($serial-num)";
     9    }
$ ./main9.pl6 --name=Patch --serial-num=12345
Usage:
  ./main9.pl6 --name=<Str> [--rank=<Str>] [--serial-num=<Int>]
$ ./main9.pl6 --name=Patch --serial-num=1234567
name (Patch) rank (NA) serial number (1234567)
```

Unfortunately the usage doesn't include information about the length, so we can override the default `USAGE` \([https://docs.perl6.org/language/functions\\#index-entry-USAGE\](https://docs.perl6.org/language/functions\#index-entry-USAGE\)\):

```
$ cat -n main10.pl6
     1    #!/usr/bin/env perl6
     2
     3    sub MAIN (
     4        Str :$name!,
     5        Str :$rank='NA',
     6        UInt :$serial-num where *.Str.chars == 7 = 1111111
     7    ) {
     8        put "name ($name) rank ($rank) serial number ($serial-num)";
     9    }
    10
    11    sub USAGE {
    12        printf "Usage:\n  %s --name=<Str> [--rank=<Str>] [--serial-num=<Int>]\n",
    13            $*PROGRAM.basename;
    14
    15        put "Note: --serial-num must be 7 digits in length."
    16    }
$ ./main10.pl6
Usage:
  main10.pl6 --name=<Str> [--rank=<Str>] [--serial-num=<Int>]
Note: --serial-num must be 7 digits in length.
```

I can move the `--serial-num` code into a `subset` \([https://docs.perl6.org/language/typesystem\\#index-entry-subset-subset\](https://docs.perl6.org/language/typesystem\#index-entry-subset-subset\)\) and further constrain `--rank` to a list of acceptable values:

```
$ cat -n main11.pl6
     1    #!/usr/bin/env perl6
     2
     3    subset SerialNum of UInt where *.Str.chars == 7;
     4    my @ranks = <NA Recruit Private PSC PFC Specialist>;
     5    subset Rank of Str where * eq any(@ranks);
     6
     7    sub MAIN (
     8        Str :$name!,
     9        Rank :$rank='NA',
    10        SerialNum :$serial-num=1111111
    11    ) {
    12        put "name ($name) rank ($rank) serial number ($serial-num)";
    13    }
    14
    15    sub USAGE {
    16        printf "Usage:\n  %s --name=<Str> [--rank=<Str>] [--serial-num=<Int>]\n",
    17            $*PROGRAM.basename;
    18
    19        put "--rank must be one of: {@ranks.join(', ')}";
    20        put "--serial-num must be 7 digits in length."
    21    }
$ ./main11.pl6 --name=Patch --serial-num=1234567 --rank=XXX
Usage:
  main11.pl6 --name=<Str> [--rank=<Str>] [--serial-num=<Int>]
--rank must be one of: NA, Recruit, Private, PSC, PFC, Specialist
--serial-num must be 7 digits in length.
[saguaro@~/work/perl6/main]$ ./main11.pl6 --name=Patch --serial-num=1234567 --rank=PFC
name (Patch) rank (PFC) serial number (1234567)
```

We can declare the `MAIN` with the keyword"`multi` instead of `sub` to indicate multiple signatures that the program can respond to.  In this example, we're writing a script that will print either the reverse complement or the RNA translation of a string of DNA.  This is a common pattern in command-line programs, for example, the first argument to "git" is a command like "checkout" or "branch."  We'll match `MAIN` to a literal string \(either "revcom" or "rna"\) and use a custom `subset` to define what a string of DNA looks like:

```
$ cat -n main12.pl6
     1    #!/usr/bin/env perl6
     2
     3    subset DNA of Str where /^ :i <[ACTGN]>+ $/;
     4
     5    multi MAIN ('revcom', DNA $dna) {
     6        put $dna.trans(<A C G T a c g t> => <T G C A t g c a>).flip;
     7    }
     8
     9    multi MAIN ('rna', DNA $dna) {
    10        put $dna.subst('T', 'U', :g);
    11    }
$ ./main12.pl6
Usage:
  ./main12.pl6 revcom <dna>
  ./main12.pl6 rna <dna>
[saguaro@~/work/metagenomics-book/perl6/main]$ ./main12.pl6 revcom TTACG
CGTAA
[saguaro@~/work/metagenomics-book/perl6/main]$ ./main12.pl6 rna TTACG
UUACG
```

If the user provides arguments that don't match any of our signatures, they get a `USAGE`:

```
$ ./main12.pl6 foo TTACG
Usage:
  ./main12.pl6 revcom <dna>
  ./main12.pl6 rna <dna>
$ ./main12.pl6 revcom foo
Usage:
  ./main12.pl6 revcom <dna>
  ./main12.pl6 rna <dna>
```

# Add It Up

Here are a few more examples of using types in  `MAIN` signature and discussion of types a bit further with a program that will add two numbers:

```
$ cat -n adder1.pl6
     1     #!/usr/bin/env perl6
     2
     3     sub MAIN (Int $a!, Int $b!) { put $a + $b }
     4
     5     sub USAGE {
     6         printf "  %s <Int> <Int>\n", $*SPEC.basename($*PROGRAM-NAME);
     7     }
$ ./adder1.pl6
  adder1.pl6 <Int> <Int>
$ ./adder1.pl6 foo bar
  adder1.pl6 <Int> <Int>
$ ./adder1.pl6 2 4
6
$ ./adder1.pl6 2 4.8
  adder1.pl6 <Int> <Int>
```

At line 3, I'm declaring two required integer variables called `$a` and `$b`.  If the user provides any arguments that don't match that signature, then we've seen how Perl will generate a decent usage statement.  Here I've thrown in my own `USAGE` method to indicate specifically that I want integer values.

We have a problem in that we'd like to be able to add anything that looks like a number, and `Int` won't allow the argument "4.8."  We can move up the type hierarchy to `Numeric` to be more generic:

```
$ cat -n adder2.pl6
     1     #!/usr/bin/env perl6
     2
     3     sub MAIN (Numeric $a!, Numeric $b!) { put $a + $b }
     4
     5     sub USAGE {
     6         note sprintf "  %s <Numeric> <Numeric>", $*SPEC.basename($*PROGRAM-NAME);
     7     }
$ ./adder2.pl6 2>err
$ cat err
  adder2.pl6 <Numeric> <Numeric>
$ ./adder2.pl6 2.71828 3.14159
5.85987
$ ./adder2.pl6 2.71828 5.4e+3
5402.71828
```

Something else I added was the `note` in `USAGE` so that the usage statement is printed to STDERR \(standard error\).  You can read about other useful I/O \(input/output\) methods at [https://docs.perl6.org/type/IO](https://docs.perl6.org/type/IO).

What I like about creating descriptive signatures for `MAIN` is that it helps us define what inputs we require and communicates the requirements efficiently to the user.  You are encouraged to always use a `MAIN` entry point when possible.

# But wait, there's more!

Here's a magical little secret: Everything that applies to `MAIN` also applies to every subroutine in Perl! I'll declare a `multi foo` subroutine that will work just like the above `MAIN` where I match on a literal string and some other sort of argument like an `Int` or a `Str`:

```
$ cat -n sub.pl6
     1	#!/usr/bin/env perl6
     2
     3	foo('bar', 'baz');
     4	foo('bar', 123);
     5	foo('quux', 3.14);
     6
     7	multi foo ('bar', Str $str) {
     8	    put "Bar got a string '$str'";
     9	}
    10
    11	multi foo ('bar', Int $n) {
    12	    put "Bar got an integer '$n'";
    13	}
    14
    15	multi foo ('quux', Any $x) {
    16	    put "Bar got an argument '$x'";
    17	}
$ ./sub.pl6
Bar got a string 'baz'
Bar got an integer '123'
Bar got an argument '3.14'
```

The combination of pattern matching of Perl subroutines and its type system give you a simple yet powerful way to write explicit and safe code.

