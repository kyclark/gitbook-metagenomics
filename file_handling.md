# File handling

Most data in bioinformatics is exchanged an some sort of text format.  Reading and writing from text is bread-and-butter work, so let's do some.

# File tests

Given some input, how do you know it's a file, that you can read it, etc.?  In our bash section, we came across Unix file tests with this where we wanted to check if `$OUT_DIR` was a directory using `-d`:

```
if [[ ! -d $OUT_DIR ]]; then
  mkdir -p $OUT_DIR
fi
```

Perl has similar tests, and to use them you simply tell Perl to treat the variable as an IO object and then execute the method for the test.  You can find all the tests [https://docs.perl6.org/type/IO$COLON$COLONPath\#File\_test\_operators](https://docs.perl6.org/type/IO$COLON$COLONPath#File_test_operators).  So the above in Perl would look like this:

```
mkdir $out-dir unless $out-dir.IO.d;
```

# Say it loud...

Here's a trivial example to UPPERCASE the contents of a file:

```
$ cat -n upper1.pl6
     1     #!/usr/bin/env perl6
     2
     3     sub MAIN (Str $file!) {
     4         die "Not a file ($file)" unless $file.IO.f;
     5
     6         for $file.IO.lines -> $line {
     7             put $line.uc;
     8         }
     9     }
$ ./upper1.pl6 foo
Not a file (foo)
  in sub MAIN at ./upper1.pl6 line 6
  in block <unit> at ./upper1.pl6 line 5
$ ./upper1.pl6 jabberwocky.txt | head -6
JABBERWOCKY -- LEWIS CARROLL

'TWAS BRILLIG, AND THE SLITHY TOVES
  DID GYRE AND GIMBLE IN THE WABE:
ALL MIMSY WERE THE BOROGOVES,
  AND THE MOME RATHS OUTGRABE.
```

At line 4, we should first check that the input was actually a file.  If it is not, then we `die` with an error message which will halt execution \(see above\).  At line 6, we start a `for` loop using the `$file.IO.lines` \([https://docs.perl6.org/type/IO$COLON$COLONHandle\#method\_lines](https://docs.perl6.org/type/IO$COLON$COLONHandle#method_lines)\) method to open the file and read in each of the lines into the `$line` variable.  Lastly we `put` the uppercase \(`uc`\) version of the line.

We can borrow a technique from functional programming with `map` \([https://docs.perl6.org/routine/map](https://docs.perl6.org/routine/map)\) to make this shorter:

```
$ cat -n upper2.pl6
     1     #!/usr/bin/env perl6
     2
     3     sub MAIN (Str $file!) {
     4         die "Not a file ($file)" unless $file.IO.e;
     5
     6         put $file.IO.lines.map(*.uc).join("\n");
     7     }
```

This technique pushes the lines of the file directly into a the `uc` transformation and then into the join to make a new string.

> Golfing: Sometimes you may here about programmers \(often Perl or c hackers\) who like to "golf" their programs.  It's an attempt to create a program using the fewest keystrokes as possible, similar to the strategy in golf where the player tries to strike the ball as few times as possible to put it into the cup.  It's not a necessarily admirable quality to make one's code as terse as possible, but there is some truth to the notion that more code means more bugs.  There are incredibly powerful ideas built into every language, and learning how they can save you from writing code is worth the effort of writing fewer bugs.  If you understand `map`, then you probably also understand why `for` loops and mutable variables are dangerous.  If you don't, then keep studying.

# Reading two files in parallel

I came across a StackOverflow question where the user had sequence scores in Phred format in one file and the sequences in another file like so:

```
$ cat phred.txt
"#$%
&'()
$ cat seq.txt
ABCD
EFGH
```

The user needed to merge the files together, line-by-line, and assign "Sequence\_\#" headers like so:

```
Sequence_1
1    2    3    4
A    B    C    D
Sequence_2
5    6    7    8
E    F    G    H
```

In the following script, I use the `Z` zip operator \([https://docs.perl6.org/routine/Z](https://docs.perl6.org/routine/Z)\) to combine three lists together -- an increasing integer value \(sequence number\), a line from the Phred file, and a line from the sequence file.  To understand that, look at zipping an infinite list of integers `1..*` with the first four letters of the alphabet `"a".."d"` and the lines from the local dictionary file.  Remember that zipping stops on the shortest list:

```
> 1..* Z "a".."d" Z '/usr/share/dict/words'.IO.lines
((1 a A) (2 b a) (3 c aa) (4 d aal))
```

Here it is in action:

```
$ cat -n parallel.pl6
     1    #!/usr/bin/env perl6
     2
     3    subset File of Str where *.IO.f;
     4
     5    sub MAIN (File :$phred='phred.txt', File :$seq='seq.txt') {
     6        my $phred-fh = open $phred;
     7        my $seq-fh   = open $seq;
     8        my %xlate    = map { chr($_ + 33) => $_ }, 1..8;
     9
    10        for 1..* Z $phred-fh.IO.lines Z $seq-fh.IO.lines -> ($i, $score, $bases) {
    11            put join "\n",
    12                "Sequence_$i",
    13                (map { %xlate{$_} }, $score.comb).join("\t"),
    14                $bases.comb.join("\t");
    15        }
    16    }
```

# Movie file reader

It seems to me that, even in movies set in the future, filmmakers still like to have computers spit out messages one-character-at-a-time as if they were arriving like telegrams. If you would like to read a file like this, I present the "movie file reader." First, an explicit, long-hand version:

```
$ cat -n reader1.pl6
     1    #!/usr/bin/env perl6
     2
     3    sub MAIN (Str $file) {
     4        for $file.IO.lines(:chomp(False)) -> $line {
     5            for $line.comb -> $letter {
     6                print $letter;
     7                my $pause = do given $letter {
     8                    when /<[.!?]>/ { .50 }
     9                    when /<[,;]>/  { .20 }
    10                    default        { .05 }
    11                }
    12                sleep $pause;
    13            }
    14        }
    15    }
```

So I'm reading the given $file line-by-line, telling Perl not to "chomp" each line \(remove the newline for whatever value constitutes that, which, BTW, you can set with `nl-in`\).  Another way to write that is `:!chomp`.  I `print` the letter, not `put` because I don't want a newline after it. Then I need to pause with the `sleep` because computers move way faster than the human eye. To figure out how long to sleep, I want to inspect the character for punctuation that either ends a sentence or introduces a pause. I use `<[]>` to create a character class that includes a period, exclamation point, and question mark or one that includes a comma or semi-colon. The `do given` lets me return the value of the `given` statement, effectively turning it into a `given` operator.

I always bounce my ideas off `#perl6` on IRC, and Zoffix suggested this much shorter version:

```
$ cat -n reader2.pl6
     1    #!/usr/bin/env perl6
     2
     3    sub MAIN (Str $file) {
     4        for $file.IO.comb {
     5            .print;
     6            sleep  /<[.!?]>/ ?? .30
     7                !! /<[,;]>/  ?? .10
     8                !!              .05;
     9        }
    10    }
```

Here we're reading the file character-by-character into the default `$_` topic variable upon which we can call the `.print` method. Then we `sleep` \(perchance to dream\) using a stacked ternary operator to find how long. Much shorter, but more cryptic to the uninitiated. I like both versions because they both work and they allow the programmer varying levels of expressiveness and efficiency.

# Reading compressed files

Often it makes sense to keep data in a compressed format to save disk space.  If you need to read a gzipped file, here's a way to use a pipe \(Proc [https://docs.perl6.org/language/ipc\#index-entry-Proc\_object-proc](https://docs.perl6.org/language/ipc#index-entry-Proc_object-proc)\):

```
$ cat -n read-gzip.pl6
     1     #!/usr/bin/env perl6
     2
     3     sub MAIN (Str $file! where *.IO.f) {
     4         my $fh = $file ~~ /\.gz$/
     5                ?? run(«gunzip -c $file |», :out).out
     6                !! open $file;
     7
     8         for $fh.lines -> $line {
     9             put $line;
    10         }
    11     }
```



