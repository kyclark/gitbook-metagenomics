# Counting Things

Given a file like this:

```
$ cat -n names.txt
     1 	cat
     2 	dog
     3 	mouse
     4 	bird
     5 	cat
     6 	cat
     7 	dog
```

We can count the number of times each animal occurs with command-line tools:

```
$ sort names.txt | uniq -c
   1 bird
   3 cat
   2 dog
   1 mouse
```

If we wanted the output sorted numerically up or down:

```
$ sort names.txt | uniq -c | sort -n
   1 bird
   1 mouse
   2 dog
   3 cat
$ sort names.txt | uniq -c | sort -nr
   3 cat
   2 dog
   1 mouse
   1 bird
```

If we wanted them sorted instead on the animals:

```
$ sort names.txt | uniq -c | awk 'OFS="\t" { print $2,$1 }' | sort
bird   	1
cat    	3
dog    	2
mouse  	1
$ sort names.txt | uniq -c | awk 'OFS="\t" { print $2,$1 }' | sort -r
mouse  	1
dog    	2
cat    	3
bird   	1
```

Obviously this is getting a bit complicated, and it will get worse as we need to handle ugly data like "Mouse/MOUSE/mice/muose," etc.  And what if we needed to handle this file:

```
$ cat -n name-value.txt
     1 	mouse  	1
     2 	cat    	3
     3 	bird   	1
     4 	dog    	10
     5 	frog   	5
     6 	cat    	2
     7 	mouse  	3
     8 	frog   	1
```

Let's write a Perl script!

```
$ cat -n name-count1.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	sub MAIN (Str $file! where *.IO.f) {
     4 	    my %count;
     5 	    for $file.IO.lines -> $key {
     6 	        %count{ $key }++;
     7 	    }
     8
     9 	    for %count.kv -> $key, $value {
    10 	        put join "\t", $key, $value;
    11 	    }
    12 	}
$ ./name-count1.pl6 names.txt
dog    	2
cat    	3
bird   	1
mouse  	1
```

By now, this code should look pretty self-explanatory.  We set up the ```%count``` hash so that we can increment (```++```) for each line in the file (lines 4-7), and then we call the key-value pairs via ```%count.kv``` to ```put``` the keys and their counts joined on a tab character.

Now let's add an option to sort the data with an option to use descending order.

```
$ cat -n name-count2.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	sub MAIN (Str $file! where *.IO.f, :$desc=False) {
     4 	    my %count;
     5 	    for $file.IO.lines -> $key {
     6 	        %count{ $key }++;
     7 	    }
     8
     9 	    my @sorted = %count.sort;
    10 	    @sorted   .= reverse if $desc;
    11
    12 	    for @sorted -> $pair {
    13 	        put join "\t", $pair.key, $pair.value;
    14 	    }
    15 	}
$ ./name-count2.pl6 --desc names.txt
mouse  	1
dog    	2
cat    	3
bird   	1
```

I added a "flag" called ```--desc``` set by default to ```False```.  At line 9, we sort the ```%count``` hash to populate an Array of Pair objects.  Remember that hashes are stored in an unordered manner, so it's necessary to put the Pairs into a container that will respect their ordering.  The only reason to put it into a Array is for line 10 where we mutate the Array *in place* by calling ```.= reverse``` on it if the ```$desc``` flag is ```True```.

Now let's look at a version that sorts by the *count* of the keys:

```
$ cat -n name-count3.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	sub MAIN (Str $file! where *.IO.f, Bool :$desc=False) {
     4 	    my %keys;
     5 	    for $file.IO.lines -> $key {
     6 	        %keys{ $key }++;
     7 	    }
     8
     9 	    my @sorted = %keys.sort(*.value);
    10 	    @sorted   .= reverse if $desc;
    11
    12 	    for @sorted -> $pair {
    13 	        put join "\t", $pair.key, $pair.value;
    14 	    }
    15 	}
 $ ./name-count3.pl6 names.txt
bird   	1
mouse  	1
dog    	2
cat    	3
$ ./name-count3.pl6 --desc names.txt
cat    	3
dog    	2
mouse  	1
bird   	1
```

There are just two changes to this script.  First, I added a ```Bool``` constraint on the ```$desc``` to indicate that I want to treat this as a binary, on/off flag.  The other change is on line 9 where I pass to the ```sort``` call the *unary* operation ```*.value```.  Take this example:

```
> my @names = <fred barney Wilma Betty>
[fred barney Wilma Betty]
> @names.sort
(Betty Wilma barney fred)
> @names.sort(*.lc)
(barney Betty fred Wilma)
> @names.sort(*.chars)
(fred Wilma Betty barney)
```

A call to ```sort``` use an lexicographic sort where uppercase characters occur before lowercase (cf. the ASCII table).  Passing the code ```*.lc``` tells ```sort``` to call the lowercase method on each of the arguments before sorting, so we get a case-insensitive ordering.  Likewise, passing ```*.chars``` says to order by the number of characters in the strings.

Now let's merge these scripts so that the user can choose whether to sort on the keys (default) or the counts either in ascending (default) or descending order:

```
$ cat -n name-count4.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	subset SortBy of Str where * ~~ /^keys?|values?$/;
     4 	sub MAIN (Str $file! where *.IO.f, SortBy :$sort-by='key', Bool :$desc=False) {
     5 	    my %keys;
     6 	    for $file.IO.lines -> $key {
     7 	        %keys{ $key }++;
     8 	    }
     9
    10 	    my @sorted = $sort-by ~~ /key/ ?? %keys.sort !! %keys.sort(*.value);
    11 	    @sorted   .= reverse if $desc;
    12
    13 	    for @sorted -> $pair {
    14 	        put join "\t", $pair.key, $pair.value;
    15 	    }
    16 	}
$ ./name-count4.pl6 --sort-by=value --desc names.txt
cat    	3
dog    	2
mouse  	1
bird   	1
```

I've create a ```subset``` on line 3 to describe the allowed values for the ```$sort-by``` argument.  I've described it as a String that must match the regular expression:

```
/ ^ keys? | values? $ /
1 2 3   4 5 6     7 8 9
```

1. beginning of regular expression
2. start of the string
3. the string "key"
4. optional string "s"
5. OR
6. the string "value"
7. optional string "s"
8. the end of the string
9. end of regular expression

This allows both "key" or "keys," "value" or "values."  Line 10 is now using the ternary operator (```predicate ?? true-branch !! false-branch```) with a test on whether the ```$sort-by``` matches the string "key" (which will also match "keys").  If it does match, it sort as normal, otherwise it sorts on the values.  The rest is identical to before.

As you might have guessed, I'm now going to show you a significantly shorter version:

```
$ cat -n name-count5.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	sub MAIN (Str $file! where *.IO.f) {
     4 	    put $file.IO.lines.Bag.map(*.kv.join("\t")).join("\n");
     5 	}
```

As great as hashes are, a Bag (immutable) or a BagHash (mutable) is the better option for this task.  We saw a Bag in the DNA problem, so that much should not be new.  The processing of it with ```map``` and ```join``` might be confusing, so let's look more closely at that in the RELP:

```
> my $bag = 'names.txt'.IO.lines.Bag
bag(mouse, cat(3), bird, dog(2))
> $bag.map(*.kv.join('::'))
(mouse::1 cat::3 bird::1 dog::2)
> $bag.map(*.kv.join('::')).join(' -- ')
mouse::1 -- cat::3 -- bird::1 -- dog::2
```

I'm using commas and dashes as delimiters as tabs and newlines won't show well in the REPL.  The Bag has a ```map``` method that allows us to apply a block of code to each element.  The Bag is hold Pairs like "mouse => 1," "cat => 3," so we call ```*.kv``` to get a List:

```
> (mouse => 1).kv
(mouse 1)
> say (mouse => 1).kv.WHAT
(List)
```

Then we ask that the List be joined:

```
> say (mouse => 1).kv.join('::')
mouse::1
```

The result of the ```map``` operation is a new List of Strings that we then want to ```join``` on newlines.

Obviously this is a big of golf on my part.  It's not a script I would actually release or maintain, in part because it's not as functional as the previous one.  Here's is a more realistic program:

```
$ cat -n name-count6.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	subset SortBy of Str where * (elem) <key value keys values>.Bag;
     4 	sub MAIN (Str $file! where *.IO.f, SortBy :$sort-by='key', Bool :$desc=False) {
     5 	    my $bag    = $file.IO.lines.Bag;
     6 	    my @sorted = $sort-by ~~ /key/ ?? $bag.sort !! $bag.sort(*.value);
     7 	    @sorted   .= reverse if $desc;
     8 	    put join "\n", map *.join("\t"), @sorted;
     9 	}
 ```
 
On line 3, I'm showing you another way to define a set of acceptable strings using the test ```(elem)``` to see if *the thing* (```*```) is in the Bag.  Otherwise, this script is pretty close to the earlier versions, only shorter and less error prone.  

Here is another version with some interesting variations:

```
$ cat -n name-count7.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	subset SortBy of Str where * ∈ <key value keys values>.Bag;
     4 	sub MAIN (Str $file! where *.IO.f, SortBy :$sort-by='key', Bool :$desc=False) {
     5 	    my $bag    = $file.IO.lines.Bag;
     6 	    my @sorted = $sort-by ~~ /key/ ?? $bag.sort !! $bag.sort(*.value);
     7 	    @sorted   .= reverse if $desc;
     8 	    put @sorted.map(*.join("\t")).join("\n");
     9 	} 
```

Line 3 uses the Unicode version ```∈``` of the ```(elem)``` operator.  Perl handles Unicode natively for both code and data!  The other change is at line 8 where I use chained method calls instead of functions to generate the output.

Now let's move on to a version that sums the counts for various keys:

```
$ cat -n name-value-count1.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	subset SortBy of Str where * ∈ <key value keys values>.Bag;
     4 	sub MAIN (Str $file! where *.IO.f, SortBy :$sort-by='key', Bool :$desc=False) {
     5 	    my %counts;
     6 	    for $file.IO.lines -> $line {
     7 	        my ($key, $value) = $line.split("\t");
     8 	        %counts{ $key } += $value;
     9 	    }
    10
    11 	    my @sorted = $sort-by ~~ /key/ ?? %counts.sort !! %counts.sort(*.value);
    12 	    @sorted   .= reverse if $desc;
    13
    14 	    for @sorted -> $pair {
    15 	        put join "\t", $pair.key, $pair.value;
    16 	    }
    17 	}
```

This is almost identical to one of our first versions, but we are calling ```$line.split``` to break the line on the tab character into the ```$key``` and ```$value```.  Since we're assiging them as a list, we need the parentheses around the ```my ()```.  At line 8, we're using the ```+=``` operator to add the ```$value``` to whatever was in ```$count{ $key }```.  If there was nothing there, the key is created and set to 0.  The rest of the script continues as before.

Here is another version to consider:

```
$ cat -n name-value-count2.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	subset SortBy of Str where * ∈ <key value keys values>.Bag;
     4 	sub MAIN (Str $file! where *.IO.f, SortBy :$sort-by='key', Bool :$desc=False) {
     5 	    my %counts;
     6 	    for $file.IO.lines.map(*.split(/\s+/)) -> [$key, $value] {
     7 	        %counts{ $key } += $value;
     8 	    }
     9
    10 	    my @sorted = $sort-by ~~ /key/ ?? %counts.sort !! %counts.sort(*.value);
    11 	    @sorted   .= reverse if $desc;
    12
    13 	    put @sorted.map(*.join("\t")).join("\n");
    14 	}
$ ./name-value-count2.pl6 --desc name-value.txt
mouse  	4
frog   	6
dog    	10
cat    	5
bird   	1
$ ./name-value-count2.pl6 --desc --sort-by=value name-value.txt
dog    	10
frog   	6
cat    	5
mouse  	4
bird   	1
 ```
 
 On line 6, I'm using a regular expression in the ```split``` call to say I want to split on any number of whitespace characters.  This would break if we had a key like "grey wolf," so it's important to know your data.  I'm just showing you another way to specify the ```split```.  The other big change here is that the ```for``` has a signature after the ```-> [$key, $value]```.  Remember that curlies ```{}``` create code blocks, and code blocks can have signatures.  Here we're having Perl match the pattern ```[$key, $value]``` to say we're expecting something that looks like a two-element list and to put those values into the variables we named.