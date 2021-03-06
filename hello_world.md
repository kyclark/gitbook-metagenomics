# Hello, World

You've probably figured out already that the first thing you're supposed to write in any new language is "Hello, World."  So, let's do that.

```
$ cat hello.pl6
#!/usr/bin/env perl6
put "Hello, World!"
$ ./hello.pl6
Hello, World!
```

Well, that looks almost exactly like bash and Python except that it uses ```put``` (like ```puts``` in Ruby) instead of ```echo``` or ```print```.  Maybe this won't be too hard?  

OK, let's rewrite our "greeting" script in Perl6.  

```
$ cat -n greet1.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	unless (1 <= @*ARGS.elems <= 2) {
     4 	    note "Usage:\n\t{$*PROGRAM-NAME.IO.basename} GREETING [NAME]";
     5 	    exit 1;
     6 	}
     7
     8 	my ($greeting, $name) = @*ARGS;
     9
    10 	put "$greeting, {$name // 'Stranger'}";
$ ./greet1.pl6 Howdy "Old Joe Clark"
Howdy, Old Joe Clark
```

Well, that looks suspiciously close to the bash version.  Line 1 is our now-familiar shebang.  We don't need to know the exact path to the Perl6 binary as long as it's somewhere in our $PATH.  On line 3, we check the number of arguments to the script by looking at ```@*ARGS``` which is equivalent to bash's ```$@```.  In bash we had to write two separate conditionals to check if the number of arguments was with a range, but in Perl we can write it just like in algebra class, "X <= Y <= Z" means Y is greater-than-or-equal to X and less-than-or-equal to Z.  The call to ```note``` is a way to print to STDERR, and we're formatting a standard "Usage" statement.  On line 5 we see that we can "exit 1" just as in bash.

One very bash-like thing we see on lines 4 and 10 Line 10 is that we can call a function *inside a string*.  In bash, the function call is denoted with ```$()``` or backticks (\`\`).  In Perl curly braces ```{}``` create what is called a "block" of code that can be executed or used as an argument to another function.  Inside the block on line 4, we're using the global/magic variable/object ```$*PROGRAM-NAME``` to find the ```basename``` of the program, and on line 10, we're using the ```//``` operator to say "what's in the $name variable or, if that is not defined, then the string 'Stranger'").  You can read about Perl 6's operators at https://docs.perl6.org/routine.html.

At line 8 is there's a ```my```, which is new.  Remember in bash how we had to ```set -u``` to tell bash to complain when we try to use a variable that was never initialized?  That kind of checking is built into Perl so that if we were to write ```put $greting``` it would complain:

```
===SORRY!=== Error while compiling /Users/kyclark/work/abe487/book/perl6/./greet1.pl6
Variable '$greting' is not declared. Did you mean '$greeting'?
at /Users/kyclark/work/abe487/book/perl6/./greet1.pl6:10
```

Another difference at line 8 is that we can assign two variables, "greeting" and "name," in one line by using a list (the parentheses) on the left-hand side ("LHS" in CS parlance).

Now to talk about the differences in those sigils.  We've seen them in bash, and they're in awk and sed, too, but they are usually just ```$``` signs.  Now're we're seeing ```$*SPEC``` and ```@*ARGS```.  We need to talk about data shapes.