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

Well, that looks suspiciously close to the bash version, and it is almost a line-for-line translation.  Let's break it down.  Line 1 is our now-familiar shebang.  We don't need to know the exact path to the Perl6 binary as long as it's somewhere in our $PATH.  Next we check the number of arguments to the script by looking at @\*ARGS 