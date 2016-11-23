# Sets



# Bags

Bags are like instant histograms.  They're perfect for counting the number of times each item occurs in a set.  Here's a quick way to count the lengths of each line in a file:

```
$ cat -n line-count-histogram1.pl6
     1	#!/usr/bin/env perl6
     2
     3	sub MAIN (Str $file) {
     4	    my $bag = $file.IO.lines.map(*.chars).Bag;
     5	    put $bag.pairs.sort(*.key).map(*.join("\t")).join("\n");
     6	}
```

Lots of stuff happening in just two lines!  Let's break it down.  Given this file:

```
$ cat words
a
an
the
frog
cat
dog
horse
```

Here is how we get there.  Read all the lines from the file:

```
> 'words'.IO.lines
(a an the frog cat dog horse)
```

Now ```map``` each line into a function to count how many characters are in the line.  The ```*``` stands for the current line as it passes through the function:

```
> 'words'.IO.lines.map(*.chars)
(1 2 3 4 3 3 5)
```

Now "Bag" those -- Perl will count each unique entry.  Since there are three words that have three characters, you see "3(3)":

```
> my $bag = 'words'.IO.lines.map(*.chars).Bag
bag(5, 4, 3(3), 1, 2)
```

Get the ```pairs``` from the Bag where the ```key``` represents the length of the word and the ```value``` is the number of times it was seen:

```
> $bag.pairs
(5 => 1 4 => 1 3 => 3 1 => 1 2 => 1)
```

You see these are not returned in any particular order, so let's have them sorted by the ```key```:

```
> $bag.pairs.sort(*.key)
(1 => 1 2 => 1 3 => 3 4 => 1 5 => 1)
```

To keep this on one line, I'm going to join the ```key```/```value``` with a colon and each member of the bag with a comma:

```
> $bag.pairs.sort(*.key).map(*.kv.join(":"))
(1:1 2:1 3:3 4:1 5:1)
> $bag.pairs.sort(*.key).map(*.kv.join(":")).join(", ")
1:1, 2:1, 3:3, 4:1, 5:1
```

If I would rather see the words sorted by the number of times they occur with the most frequent ones first, it would look like this:

```
>  $bag.pairs.sort(*.value).reverse.map(*.kv.join(":")).join(", ")
3:3, 2:1, 1:1, 4:1, 5:1
```

Here is the output from the script:

```
$ ./line-count-histogram1.pl6 words
1	1
2	1
3	3
4	1
5	1
$ ./line-count-histogram1.pl6 /usr/share/dict/words
1	52
2	160
3	1420
4	5272
5	10230
6	17706
7	23869
8	29989
9	32403
10	30878
11	26013
12	20462
13	14939
14	9765
15	5925
16	3377
17	1813
18	842
19	428
20	198
21	82
22	41
23	17
24	5
```

# SetHash, BagHash

Sets and Bags are immutable:

```
> my $bag = <foo bar foo baz>.Bag
bag(foo(2), baz, bar)
> $bag<foo> = 3
Cannot modify an immutable Int
  in block <unit> at <unknown file> line 1
```

If you want to change them, use the ```SetHash``` (https://docs.perl6.org/type/SetHash) and ```BagHash``` (https://docs.perl6.org/type/BagHash) variants:

```
> my $baghash = <foo bar foo baz>.BagHash
BagHash.new(foo(2), baz, bar)
> $baghash<foo>++
2
> $baghash
BagHash.new(foo(3), baz, bar)
```

These are both excellent data structures for when you have a unique group of items from which you would like to draw such that they are removed from the container.  When I was writing my Blackjack game, I found a ```Set``` to be the natural container for a deck of 52 unique cards:

```
> my @suites = <D H C S>
[D H C S]
> my @faces = <2 3 4 5 6 7 8 9 10 J Q K A>
[2 3 4 5 6 7 8 9 10 J Q K A]
> my $deck = (@faces X @suites).map(~*).SetHash
SetHash.new(3 C, 5 H, 7 S, 8 D, 10 S, 2 D, 2 H, 9 D, Q S, A C, 6 H, 6 C, 10 C, Q H, J S, 3 H, 9 H, A H, J D, 7 D, 8 H, Q C, 10 D, J C, 3 D, 4 D, 6 D, 8 C, 5 C, 5 D, 4 H, 2 S, K D, 8 S, 7 C, 4 S, K C, 7 H, K H, K S, 6 S, 5 S, J H, A D, A S, 10 H, 9 S, 4 C, 2 C, 9 C, 3 S, Q D)
> $deck.elems
52
> $deck.grab(2).join(', ')
9 C, 9 D
> $deck.elems
50
```

The ```X``` (https://docs.perl6.org/routine/X) operator crosses two lists to produce a ```Pair``` for every combination, then I ```map``` each ```Pair``` into a stringification operation ```~*``` because the keys of a Set or Bag are strings.  When I ```grab``` cards (at random) from the Set, they are removed so that I don't have to worry about seeing them again.

When I realized that casinos regularly draw from a stack of many decks of cards, I realized I'd have to turn to a ```BagHash``` so that I could keep track of how many, e.g., Jack of Diamonds I have:

```
> my $baghash = (((@faces X @suites).map(~*)) xx 2).flat.BagHash
BagHash.new(3 C(2), 5 H(2), 7 S(2), 8 D(2), 10 S(2), 2 D(2), 2 H(2), 9 D(2), Q S(2), A C(2), 6 H(2), 6 C(2), 10 C(2), Q H(2), J S(2), 3 H(2), 9 H(2), A H(2), J D(2), 7 D(2), 8 H(2), Q C(2), J C(2), 3 D(2), 4 D(2), 6 D(2), 8 C(2), 10 D(2), K D(2), 2 S(2), 4 H(2), 5 D(2), 5 C(2), K C(2), 4 S(2), 7 C(2), 8 S(2), K S(2), K H(2), 7 H(2), A S(2), A D(2), J H(2), 5 S(2), 6 S(2), 2 C(2), 4 C(2), 9 S(2), 10 H(2), Q D(2), 3 S(2), 9 C(2))
> $baghash.grab(60).elems
60
> $baghash.elems
37
```

# Mixes



