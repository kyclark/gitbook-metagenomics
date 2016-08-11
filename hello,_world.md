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
    10	put "$greeting, {$name // 'Stranger'}";
[saguaro@~/work/abe487/book/perl6]$ vi greet1.pl6
[saguaro@~/work/abe487/book/perl6]$ cat -n greet1.pl6
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

Well, that looks suspiciously close to the bash version, and it is almost a line-for-line translation.  Let's 