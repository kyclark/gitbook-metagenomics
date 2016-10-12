# Palindrome

Here's a common example used to teach "strings," which in CS-talk means a list of characters.  We're going to write a program that finds all the palindromes in a file of words.  Here's our first version:

```
$ cat -n palindrome1.pl6
     1	#!/usr/bin/env perl6
     2
     3	sub MAIN (Str $file='/usr/share/dict/words') {
     4	    die "$file not a file" unless $file.IO.f;
     5
     6	    for $file.IO.lines -> $word {
     7	        my $lc  = $word.lc;
     8	        my $rev = $lc.comb.reverse.join;
     9
    10	        if $lc eq $rev {
    11	            put $word;
    12	        }
    13	    }
    14	}
```

By default, we're going to read the dictionary that's almost universally in the same place on every Unix system.  The user can provide a different word list, if they like:

```
$ ./palindrome1.pl6 -h
Usage:
  ./palindrome1.pl6 [<file>]
```

