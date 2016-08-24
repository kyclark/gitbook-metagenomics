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
```

