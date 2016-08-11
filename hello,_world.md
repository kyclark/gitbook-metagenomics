# Hello, World

You've probably figured out already that the first thing you're *supposed* to write in any new language is "Hello, World."  So, let's do that.

```
$ cat hello.pl6
#!/usr/bin/env perl6
put "Hello, World!"
$ ./hello.pl6
Hello, World!
```

Well, that looks almost exactly like bash and Python.  Except that is used ```put``` instead of ```print```.  Maybe this won't be so hard?  

OK, let's rewrite our "greeting" script in Perl6.  

```
$ cat -n greet1.pl6
     1	#!/usr/bin/env perl6
     2
     3	unless (1 <= @*ARGS.elems <= 2) {
     4	    printf "Usage: %s GREETING [NAME]\n", $*SPEC.basename($*PROGRAM-NAME);
     5	    exit 1;
     6	}
     7
     8	my ($greeting, $name) = @*ARGS;
     9
    10	put "$greeting, {$name // 'Stranger'}!";
$ ./greet1.pl6 Howdy "Old Joe Clark"
Howdy, Old Joe Clark!
```

Well, that looks suspiciously close to the bash version, and it is almost a line-for-line translation.  Line 1 is our now-familiar shebang.  We don't need to know the exact path to the Perl6 binary as long as it's somewhere in our $PATH.  On line 3, we check the number of arguments to the script by looking at ```@*ARGS``` which is equivalent to bash's ```$@```.  In bash we had to write two separate conditionals, but here we can write it just like in algebra class, "X <= Y <= Z" means Y is greater-than-or-equal to X and less-than-or-equal to Z.  We have ```printf``` in Perl, but we *do* separate our arguments with commas unlike bash.  Also, our ```basename``` method must now be prefaced with the "module" (or "package," a container for code).  On line 5 we see that we can "exit 1" just as in bash, too.

Line 8 is decidedly different in that we can assign to our "greeting" and "name" variables in one line, and line 10 has something very bash-like in that we can call a function *inside a string*.  In bash, the function call is denoted with ```$()``` or backticks (\`\`), and in Perl we can wrap executable code with curly braces ```{}```.  Inside those braces on line 10, we're using the ```//``` operator to say "what's in the $name variable or, if that is not defined, then the string 'Stranger'").

Now to talk about the differences in those sigils.  We've seen them in bash, and they're in awk and sed, too, but they are usually just ```$``` signs.  Now're we're seeing ```$*SPEC``` and ```@*ARGS```.  

# Data shapes

Perl has containers for different shapes of data.  You can have one thing (a sequence of DNA), lots of things (a few million sequences), a key-value pair (metadata about the sequences like species = 'H. sapiens,' age = 33, gender = 'male'), etc.  Perl would call these things scalars, lists, and hashes, and these shapes are almost universal to all programming languages.  Perl has several other incredibly useful shapes like bags and sets, and we'll get to those later. 

Languages like Python, Ruby, Haskell, Java, make no visual distinction between their reserved words and variables.  Perl's language designers feel that the variables should stand out from the language itself, and that the sigils on the variables should give the reader an indication of the shape of the data.  

## Scalars $

If you have just one of a thing like our greeting or name, then you put it into a "scalar" or singular variable.  These are prefixed with a ```$``` like ```$greeting``` and can hold only one value.  If you set it to a second value, the first value is forever lost (unless you set it to be immutable -- more on that later).

```
my $greeting = "How you doin'?";
my $pi = 3.14;

## Lists @

When you have an undetermined number of somethings, they belong in a list (or array or sequence -- we'll figure all these out as we go along).  These are plurals, and they start with the ```@``` sign.

## Hash %

When you have a key-value association, that belongs in a hash AKA "map" or "dictionary" or "associative array," if you're not into the whole brevity thing.  Yes, I know we call the ```#``` a "hash," but this "hash" is short for a "hash table" (https://en.wikipedia.org/wiki/Hash_table).  Hashes start with the ```%``` sign.