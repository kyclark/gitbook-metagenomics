# I, Palindrome, I

Here's a common example used to teach "strings," which in CS-talk means a list of characters.  We're going to write a program that finds all the palindromes (words spelled the same forwards and backwards) in a list of words.  By default, we're going to read the dictionary that's almost universally in the same place on every Unix system.  The user can provide a different word list, if they like.  Here's our first version:

```
$ ./palindrome1.pl6 -h
Usage:
  ./palindrome1.pl6 [<file>]
$ cat -n palindrome1.pl6
     1	#!/usr/bin/env perl6
     2
     3	sub MAIN (Str $file='/usr/share/dict/words') {
     4	    die "$file not a file" unless $file.IO.f;
     5
     6	    for $file.IO.lines -> $word {
     7	        my $lc  = $word.lc;
     8	        my $rev = $lc.flip;
     9
    10	        if $lc eq $rev {
    11	            put $word;
    12	        }
    13	    }
    14	}
```

We will want to lowercase the words because "Mom" is not equal to "moM," so line 7 copies the ```$word``` into ```$lc```.  To find the reverse of the word, we  ```flip``` (https://docs.perl6.org/routine/flip) the word.  If the forward and reverse are the same, print the word.  (Be sure to use ```eq``` for string equivalence and not ```==``` for numerical.  Try ```===``` to understand the error you get.)

```
if $lc eq $rev {
    put $word;
}
```

Ask yourself, what do you get if you make this mistake?  If you don't know, try it and see.

```
if $lc = $rev { # very bad code
    put $word;
}
```

Now let's run our script, and see what we get:

```
$ ./palindrome1.pl6 | head -5
A
a
aa
aba
Abba
```

Right away, I don't like the fact that we have single letters showing up, so let's skip those.  While we're at it, I'm going to shorten up the comparison:

```
$ cat -n palindrome2.pl6
     1	#!/usr/bin/env perl6
     2
     3	sub MAIN (Str $file='/usr/share/dict/words') {
     4	    die "$file not a file" unless $file.IO.f;
     5
     6	    for $file.IO.lines -> $word {
     7	        next if $word.chars == 1;
     8	        if $word.lc eq $word.lc.flip {
     9	            put $word;
    10	        }
    11	    }
    12	}
```

Line 7 is the new bit of logic I added; otherwise, I just got rid of the ```$lc``` and ```$rev``` variables and directly test the conditions.

Ask yourself what is the result if you change the ```==``` (numerical comparison) on line 7 to ```=``` (assignment)?  In this case, you'd get this error:

```
Cannot modify an immutable Int
  in sub MAIN at ./palindrome2.pl6 line 7
  in block <unit> at ./palindrome2.pl6 line 3
```

You're not allowed to change the number of characters in the ```$word``` by assigning a value to it, which is as it should be!  Let's test our script:

```
$ ./palindrome2.pl6 | head -5
aa
aba
Abba
acca
Ada
```

It works and no longer emits single-letter words, but there are couple of things that I don't like about this version.  First off, that "1" (the minimum length) qualifies as something often called a "magic number" in that it's not documented anywhere what it means.  It's almost always better to give it a name like ```$min```, so let's do that in the next version.  While we're at it, why not make it an optional parameter to the script so that the user can define it?

Another thing that bothers me about the above code is that I'm breaking the DRY (Don't Repeat Yourself) code by executing ```lc``` twice.  By using ```map```, I can apply the ```lc``` function to the words as they come from ```lines```?  We saw this a bit in the previous chapter with uppercasing a file using a ```map``` statement to apply a function (```lc```) to every member of a list (the words from ```lines```):

```
$ cat -n palindrome3.pl6
     1	#!/usr/bin/env perl6
     2
     3	sub MAIN (Str $file='/usr/share/dict/words', :$min=2) {
     4	    die "$file not a file" unless $file.IO.f;
     5
     6	    for $file.IO.lines.map(*.lc) -> $word {
     7	        next unless $word.chars >= $min;
     8	        if $word eq $word.flip {
     9	            put $word;
    10	        }
    11	    }
    12	}
```

To better understand ```map```, let's look at some more examples:

```
$ perl6
> <We will, we will, rock you>.map(*.tc)
(We Will, We Will, Rock You)
> (1..10).map(* / 2)
(0.5 1 1.5 2 2.5 3 3.5 4 4.5 5)
> <My cat's breath smells like cat food>.map(*.chars)
(2 5 6 6 4 3 4)
> (1..10).map(*.is-prime)
(False True True False True False True False False False)
```

The ```*``` is the list element at the time and can be called "whatever" or "the thing" or "the topic."  The ```*``` creates block automatically, but you could also write the block explicitly and use the regular ```$_``` for the topic variable.

```
> (1.4, 3.6, 8.1, 2.3).map({$_.floor})
(1 3 8 2)
> (1..10).map({$_**2})
(1 4 9 16 25 36 49 64 81 100)
```

Just because, I want to shorten up the code a bit more by using ```grep``` to filter the words to those of a minimum length:

```
$ cat -n palindrome4.pl6
     1	#!/usr/bin/env perl6
     2
     3	sub MAIN (Str $file='/usr/share/dict/words', :$min=2) {
     4	    die "$file not a file" unless $file.IO.f;
     5
     6	    for $file.IO.lines.grep(*.chars >= $min).map(*.lc) -> $word {
     7	        if $word eq $word.flip {
     8	            put $word;
     9	        }
    10	    }
    11	}
```

I would like to keep track of the number of found words, so I'll add a counter and a ```printf``` statement to make pretty output:

```
$ cat -n palindrome5.pl6
     1	#!/usr/bin/env perl6
     2
     3	sub MAIN (Str $file='/usr/share/dict/words', :$min=2) {
     4	    die "$file not a file" unless $file.IO.f;
     5
     6	    my $i = 0;
     7	    for $file.IO.lines.grep(*.chars >= $min).map(*.lc) -> $word {
     8	        if $word eq $word.flip {
     9	            printf "%3d: %s\n", ++$i, $word;
    10	        }
    11	    }
    12	}
```

The only tricky thing to point out here is the ```++$i``` which increments the variable *before* taking the value.  If I did ```$i++```, then it would first print "0" (the initial value) and after would bump it to "1."

If we look closely, we find we can combine our two comparisons (minimum length, match forward/backward) into a single ```grep```:

```
$ cat -n palindrome6.pl6
     1	#!/usr/bin/env perl6
     2
     3	sub MAIN (Str $file='/usr/share/dict/words', :$min=2) {
     4	    die "$file not a file" unless $file.IO.f;
     5
     6	    my $i = 0;
     7	    for $file.IO.lines -> $line {
     8	        for $line.words.map(*.lc).grep({$_.chars >= $min && $_ eq $_.flip})
     9	        -> $word {
    10	            printf "%3d: %s\n", ++$i, $word;
    11	        }
    12	    }
    13	}
```

But that gets a little long, so we could move that into a function:

```
$ cat -n palindrome7.pl6
     1	#!/usr/bin/env perl6
     2
     3	sub MAIN (Str $file='/usr/share/dict/words', :$min=2) {
     4	    die "$file not a file" unless $file.IO.f;
     5
     6	    sub is-palindrome ($s) { $s.chars >= $min && $s eq $s.flip }
     7
     8	    my $i = 0;
     9	    for $file.IO.lines -> $line {
    10	        for $line.words.map(*.lc).grep({is-palindrome($_)}) -> $word {
    11	            printf "%3d: %s\n", ++$i, $word;
    12	        }
    13	    }
    14	}
```

What if we wanted to run this on any text file, not just a simple list of words?  We simply need to add another ```for``` loop to go through the words on each line.  In this last example, I'm also going to move the ```is-palindrome``` from a function to extending the ```(Str)``` class with this method.  It requires the use of the ```MONKEY-TYPING``` pragma and is probably not the greatest idea, but it's kind of fun to try:

```
$ cat -n palindrome8.pl6
     1	#!/usr/bin/env perl6
     2
     3	use MONKEY-TYPING;
     4
     5	sub MAIN (Str $file='/usr/share/dict/words', :$min=2) {
     6	    die "$file not a file" unless $file.IO.f;
     7
     8	    augment class Str {
     9	        method is-palindrome {
    10	            self.chars >= $min && self.lc eq self.lc.flip
    11	        }
    12	    }
    13
    14	    my $i = 0;
    15	    for $file.IO.lines -> $line {
    16	        for $line.words.grep(*.is-palindrome) -> $word {
    17	            printf "%3d: %s\n", ++$i, $word;
    18	        }
    19	    }
    20	}}
    22	}
$ wget http://www.usconstitution.net/const.txt
$ ./palindrome6.pl6 const.txt
  1: ------------------------------
  2: 11
  3: noon
  4: noon
  5: noon
  6: 22
```

Running that on the text of the US Constitution, we find only a few palindromes. 

In this last version, we'll check if a phrase provided on the command line is a palindrome.

```
$ ./palindrome9.pl6
A man, a plan, a canal. Panama
Yes
$ ./palindrome9.pl6 "This is not a palindrome"
This is not a palindrome
No
```

To check phrases, we need to remove anything that's not a letter, and the easiest way to do that is with a regular expression:

```
$ cat -n palindrome9.pl6
     1	#!/usr/bin/env perl6
     2
     3	sub MAIN (Str $phrase="A man, a plan, a canal. Panama") {
     4	    my $forward = $phrase.lc.subst(/<-[a..z]>/, '', :g);
     5	    printf "%s\n%s\n", $phrase, $forward eq $forward.flip ?? 'Yes' !! 'No';
     6	}
```

On line 4, we take the lowercase version (```lc```) of the ```$phrase``` and substitute (```subst```) anything that fits the regular expression (https://docs.perl6.org/language/regexes) with the empty string (```''```).  The third argument ```:g``` is the "global" flag; without this, the substitution would only happen one time.  Remember that the ```:g``` is a shorthand for making a Pair (https://docs.perl6.org/type/Pair) of the key ```g``` (for "global") and the value "True."  We could have written it like this, too (but we don't because it's more typing):

```
my $forward = $phrase.lc.subst(/<-[a..z]>/, '', g => True);
```

On line 5, we're using ```printf``` to establish a template for printing the output to the user.  First we want to reiterate the given phrase, then we want to print "Yes" or "No" depending on the outcome of the comparison.  On line 7, we're using the ternary operator (<https://docs.perl6.org/routine/$QUESTION_MARK$QUESTION_MARK%20!!#(Operators)_infix_??_!!>) which is a condensed ```if-then-else``` statement.  Here is another way to write it:

```
my $answer  = do if $forward eq $forward.comb.reverse.join 
                 {'Yes'} else {'No'};
```