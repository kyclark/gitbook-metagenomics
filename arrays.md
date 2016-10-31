# Lists

Lists are ordered collections of things.  Order is important, because later we'll talk about hashes and bags that are unordered.  Lists have lots of handy functions.  Let's explore.  

# Length, Order

How long is that list?  Here's a list of the main dogs in my life from when I was a kid to now:

```
> my @dogs = <Chaps Patton Bowzer Logan Lulu Patch>
[Chaps Patton Bowzer Logan Lulu Patch]
> @dogs.elems
6
> +@dogs
6
> @dogs.Int
6
> @dogs.Numeric
6
```

The ```elems``` methods returns the number of elements and is the most obvious way to find it.  Putting a ```+``` plus sign in front coerces the list into a numerical context which returns the length of the list as does calling the ```Int``` and ```Numeric``` methods.

If I want my dogs going from current to first, I can ```reverse``` them:

```
> @dogs.reverse
[Patch Lulu Logan Bowzer Patton Chaps]
```

I can sort them either by their name:

```
> @dogs.sort
(Bowzer Chaps Logan Lulu Patch Patton)
```

Or, by passing a "unary" operator (one that takes a single argument), by the results of that operation such as the number of characters in the name (i.e., the length of the string):

```
> @dogs.sort(*.chars)
(Lulu Chaps Logan Patch Patton Bowzer)
> @dogs.sort(*.chars).reverse
(Bowzer Patton Patch Logan Chaps Lulu)
```

I can find all possible pairs of dogs:

```
> @dogs.combinations(2)
((Chaps Patton) (Chaps Bowzer) (Chaps Logan) (Chaps Lulu) (Chaps Patch) (Patton Bowzer) (Patton Logan) (Patton Lulu) (Patton Patch) (Bowzer Logan) (Bowzer Lulu) (Bowzer Patch) (Logan Lulu) (Logan Patch) (Lulu Patch))
```

I can find just the length of each dog's name by using ```map``` to apply a the ```chars``` function to each element:

```
> @dogs.map(*.chars)
(5 6 6 5 4 5)
> @dogs.map(&chars)
(5 6 6 5 4 5)
```

The first version is using the ```chars``` _method_ called on each list object while the second version is applying the ```chars``` _function_ to each element (https://docs.perl6.org/routine/chars).  The leading ampersand ```&``` is passing the ```chars``` function as a reference.  Note that you cannot call it like so:

```
> @dogs.map(chars)
===SORRY!=== Error while compiling:
Calling chars() will never work with proto signature ($)
------> @dogs.map(⏏chars)
```

The ```map``` function is something I'd really like you to understand, so let's break it down a bit.  I can get the same answer (sort of) with a ```for``` loop:

```
> for @dogs -> $dog { say $dog.chars }
5
6
6
5
4
5
```

And I can capture the numbers by using ```do```:

```
> my @chars = do for @dogs -> $dog { $dog.chars }
[5 6 6 5 4 5]
```

Or, more briefly:

```
> my @chars = do for @dogs { .chars }
[5 6 6 5 4 5]
```

So the ```map``` _function_ just turns that around a bit:

```
> my @chars = map { .chars }, @dogs
[5 6 6 5 4 5]
```

And the ```map``` _method_ (of a List) turns that around:

```
> my @chars = @dogs.map({ .chars })
[5 6 6 5 4 5]
> my @chars = @dogs.map(*.chars)
[5 6 6 5 4 5]
> my @chars = @dogs.map: *.chars
[5 6 6 5 4 5]
```

You see there is more than one way to write a map.  We'll break this down later.

The elements in a List are not limited to scalars.  Using ```map```, I create a List of other Lists that combines each dog's name and the length of the name using the ```Z``` "zip" operator (https://docs.perl6.org/routine/Z):

```
> @dogs Z @dogs.map(*.chars)
((Chaps 5) (Patton 6) (Bowzer 6) (Logan 5) (Lulu 4) (Patch 5))
```

Does that makes sense?  Zip takes two lists and combines them element-by-element, stopping on the shorter list:

```
> 1..10 Z 'a'..'z'
((1 a) (2 b) (3 c) (4 d) (5 e) (6 f) (7 g) (8 h) (9 i) (10 j))
> 1..* Z 'Bowzer'.comb
((1 B) (2 o) (3 w) (4 z) (5 e) (6 r))
```

Lastly, I'll show you how ```sum``` (https://docs.perl6.org/routine/sum) total number of characters in all the dog names:

```
> @dogs.map(*.chars).sum
31
```

# Iterating

One of the most common list operations is to iterate over the members while keeping track of the position.  Here's a script that breaks a string (here maybe some DNA) into a list using the ```comb``` method and prints the position and the letter:

```
$ cat -n iterate1.pl6
     1	#!/usr/bin/env perl6
     2
     3	sub MAIN (Str $dna) {
     4	    my $i = 0;
     5	    for $dna.comb -> $letter {
     6	        $i++;
     7	        say "$i: $letter";
     8	    }
     9	}
 $ ./iterate1.pl6 AACTAG
1: A
2: A
3: C
4: T
5: A
6: G
```

This is so common that Lists have shorter ways to do this:

```
$ cat -n iterate2.pl6
     1	#!/usr/bin/env perl6
     2
     3	sub MAIN (Str $dna) {
     4	    for $dna.comb.kv -> $k, $v {
     5	        say "{$k+1}: $v";
     6	    }
     7	}
$ ./iterate2.pl6 AACTAG
1: A
2: A
3: C
4: T
5: A
6: G
```

Positions in Perl Lists and Strings start at 0, so I have to add 1 to ```$k```.  Notice that I can run code _inside a string_ by putting ```{}``` curly braces around it.  

I don't have to give the pointy block signature (```-> $k, $v```) bit.  I can use ```$^k``` and ```$^v``` (or ```$^a``` and ```$^b``` or whatever) to refer to the first and second arguments (in sorted Unicode order) to the block (https://docs.perl6.org/language/variables#index-entry-%24%5E):

```
$ cat -n iterate3.pl6
     1	#!/usr/bin/env perl6
     2
     3	sub MAIN (Str $dna) {
     4	    for $dna.comb.kv { say join ": ", $^k, $^v }
     5	}
$ ./iterate3.pl6 AACTAG
0: A
1: A
2: C
3: T
4: A
5: G
```

Here's a version using ```pairs``` to get a Pair (https://docs.perl6.org/type/Pair) with the index (position) as the "key" and the letter as the "value":

```
$ cat -n iterate4.pl6
     1	#!/usr/bin/env perl6
     2
     3	sub MAIN (Str $dna) {
     4	    for $dna.comb.pairs -> $pair {
     5	        printf "%s: %s\n", $pair.key + 1, $pair.value;
     6	    }
     7	}
 $ ./iterate3.pl6 AACTAG
1: A
2: A
3: C
4: T
5: A
6: G
```

Again, I don't have to have the ```-> $pair``` bit if I use the ```^``` twigil.  I can just refer to the one positional argument as ```$^pair``` and call the ```key``` and ```value``` methods on that:

```
$ cat -n iterate5.pl6
     1	#!/usr/bin/env perl6
     2
     3	sub MAIN (Str $dna) {
     4	    for $dna.comb.pairs { say join ': ', $^pair.key + 1, $^pair.value }
     5	}
[saguaro@~/work/metagenomics-book/perl6/lists]$ ./iterate5.pl6 AACTAG
1: A
2: A
3: C
4: T
5: A
6: G
```

Or I can use the ```:``` twigil (https://docs.perl6.org/language/variables#The_:_Twigil) a way to declare named parameters:

```
$ cat -n iterate6.pl6
     1	#!/usr/bin/env perl6
     2
     3	sub MAIN (Str $dna) {
     4	    for $dna.comb.pairs -> (:$key, :$value) {
     5	        say join ': ', $key + 1, $value;
     6	    }
     7	}
$ ./iterate6.pl6 AACTAG
1: A
2: A
3: C
4: T
5: A
6: G
```

# Filtering

Often you want to choose or remove certain members of a list.  For that, ```grep``` is probably the way to go.  Let's find only the Gs and Cs in a string (https://en.wikipedia.org/wiki/GC-content).  Note that I uppercase (```uc```) the ```$dna``` first so that I only have to check for one case of letters:

```
$ cat -n gc1.pl6
     1	#!/usr/bin/env perl6
     2
     3	sub MAIN (Str $dna) {
     4	    my @gc = $dna.uc.comb.grep({$_ eq 'G' || $_ eq 'C'});
     5	    say "$dna has {@gc.elems}";
     6	}
$ ./gc1.pl6 AACTAG
AACTAG has 2
```

Like ```map```, ```grep``` takes a block of code that will be executed for each member of the list.  Any elements for which the block evaluates to "True-ish" are allowed through.  The ```$_``` (topic, thing, "it") variable has the current element, so the code is asking "if the thing is a 'G' or if the thing is a 'C'".  One can use the ```*``` to represent "it" and eschew the curly brackets.  Here I'll also use a Junction (https://docs.perl6.org/type/Junction) to compare to "G or C" in one go:

```
$ cat -n gc2.pl6
     1	#!/usr/bin/env perl6
     2
     3	sub MAIN (Str $dna) {
     4	    my @gc = $dna.uc.comb.grep(* eq 'G' | 'C');
     5	    say "$dna has {@gc.elems}";
     6	}
$ ./gc2.pl6 AACTAG
AACTAG has 2
```

Another way to write the ```|``` Junction is with ```any```.  The ```so``` routine (https://docs.perl6.org/routine/so) collapses the various Booleans down to a single value.

```
> so 'G' eq 'G' | 'C'
True
> so 'G' eq any(<G C>)
True
```

It's extremely common to use regular expressions (https://docs.perl6.org/type/Regex) to filter lists.  We'll cover these more later, but here I'm using a character class to represent either "G or C":

```
$ cat -n gc3.pl6
     1	#!/usr/bin/env perl6
     2
     3	sub MAIN (Str $dna) {
     4	    my @gc = $dna.uc.comb.grep(/<[GC]>/);
     5	    say "$dna has {@gc.elems}";
     6	}
$ ./gc3.pl6 AACTAG
AACTAG has 2
```

Here's how you can find the prime numbers between 1 and 10:

```
> (1..10).grep(*.is-prime)
(2 3 5 7)
```

# Classification

We can group lists elements based on predicates we supply.  Here is how we can split up the numbers 1 through 10 based on whether they are or are not even divisible by 2:

```
> 2 %% 2
True
> 3 %% 2
False
> (1..10).classify(* %% 2)
{False => [1 3 5 7 9], True => [2 4 6 8 10]}
```

Going back to our G-C counter, we can group each base into whether it is or isn't a "G" or a "C":

```
$ cat -n gc4.pl6
     1	#!/usr/bin/env perl6
     2
     3	sub MAIN (Str $dna) {
     4	    my %hash = $dna.uc.comb.categorize({?/<[GC]>/});
     5	    say "$dna has {%hash<True>.elems}";
     6	}
$ ./gc4.pl6 AACTAG
AACTAG has 2
```

Although it's not really intuitive to use "True" or "False," so let's provide our own String value for the name of the bucket we want:

```
$ cat -n gc5.pl6
     1	#!/usr/bin/env perl6
     2
     3	sub MAIN (Str $dna) {
     4	    my %hash = $dna.uc.comb.categorize({ /<[GC]>/ ?? 'GC' !! 'Other' });
     5	    say "$dna has {%hash<GC>.elems}";
     6	}
$ ./gc4.pl6 AACTAG
AACTAG has 2
```

It might help to see that one in the REPL:

```
> 'AACTAG'.uc.comb.classify({?/<[GC]>/})
{False => [A A T A], True => [C G]}
> 'AACTAG'.uc.comb.classify({/<[GC]>/ ?? 'GC' !! 'Other'})
{GC => [C G], Other => [A A T A]}
```

```classify``` takes a code block and uses the resulting string to put the element into a bucket.  Here I've used the same regular expression ```/<[GC]>/``` to return the string "GC" if it's a match or "Other" if it's not.  The combination of the ```?? !!``` is the "ternary" operator that we'll talk about more later.  The resulting Hash has a key called "GC" and its value is a list containing the "G" and "C" found in the string.

So you're seeing that Lists can be inside of other Lists as well as inside of Hashes and other data structures.

I can classify my ```@dogs``` based on the length of their names using that same syntax variations we saw for ```map```:

```
> @dogs.classify({.chars})
{4 => [Lulu], 5 => [Chaps Logan Patch], 6 => [Patton Bowzer]}
> @dogs.classify(*.chars)
{4 => [Lulu], 5 => [Chaps Logan Patch], 6 => [Patton Bowzer]}
> @dogs.classify(&chars)
{4 => [Lulu], 5 => [Chaps Logan Patch], 6 => [Patton Bowzer]}
```

Lists can also be composed of Pairs (https://docs.perl6.org/type/Pair).  Here I'll redeclare my ```@dogs``` with their names as the "key" and thier sex as the "value."  Then I can ```classify``` them on their ```value```:

```
> my @dogs = Chaps => 'male', Patton => 'male', Bowzer => 'male', Logan => 'male', Lulu => 'female', Patch => 'male'
[Chaps => male Patton => male Bowzer => male Logan => male Lulu => female Patch => male]
> @dogs.classify(-> $dog {$dog.value})
{female => [Lulu => female], male => [Chaps => male Patton => male Bowzer => male Logan => male Patch => male]}
> @dogs.classify({$^dog.value})
{female => [Lulu => female], male => [Chaps => male Patton => male Bowzer => male Logan => male Patch => male]}
> @dogs.classify({.value})
{female => [Lulu => female], male => [Chaps => male Patton => male Bowzer => male Logan => male Patch => male]}
> @dogs.classify(*.value)
{female => [Lulu => female], male => [Chaps => male Patton => male Bowzer => male Logan => male Patch => male]}
```

# Picking random elements

To finish off, let's write a Shakespearean insult generator that uses the ```pick``` method to randomly choose some perjoratives:

```
$ cat -n insult.pl6
     1	#!/usr/bin/env perl6
     2
     3	sub MAIN (Int :$n=1) {
     4	    my @adjectives = qw{scurvy old filthy scurilous lascivious
     5	        foolish rascaly gross rotten corrupt foul loathsome irksome
     6	        heedless unmannered whoreson cullionly false filthsome
     7	        toad-spotted caterwauling wall-eyed insatiate vile peevish
     8	        infected sodden-witted lecherous ruinous indistinguishable
     9	        dishonest thin-faced slanderous bankrupt base detestable
    10	        rotten dishonest lubbery};
    11	    my @nouns = qw{knave coward liar swine villain beggar
    12	        slave scold jolthead whore barbermonger fishmonger carbuncle
    13	        fiend traitor block ape braggart jack milksop boy harpy
    14	        recreant degenerate Judas butt cur Satan ass coxcomb dandy
    15	        gull minion ratcatcher maw fool rogue lunatic varlet worm};
    16
    17	    printf "You %s, %s, %s %s!\n",
    18	        @adjectives.pick(3), @nouns.pick for ^$n;
    19	}
$ ./insult.pl6 -n=5
You foul, dishonest, old whore!
You irksome, gross, false degenerate!
You old, dishonest, toad-spotted jack!
You ruinous, unmannered, foolish slave!
You scurry, slanderous, peevish harpy!
```

You can also use ```roll``` so that each selection is made independently.  Above I'm using the ```qw{}``` "quote-word" operator (https://docs.perl6.org/language/quoting#index-entry-qw_word_quote) to create a list of words rather than writing:

```
my @adjectives = "scurvy", "old", "filthy";
```

You should spend a few minutes reading about all the different quoting options available as they will come in handy.