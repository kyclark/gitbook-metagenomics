# Data Shapes

Perl has various containers for different shapes of data.  You can have one thing (a string of DNA), lots of things (a list of transcription factors), a key-value pair (metadata about a sample like species = 'H. sapiens,' age = 33, gender = 'male'), etc.  Perl would call these things "scalars," "lists," and "hashes," and these shapes are almost universal to all programming languages.  Perl has several other useful shapes like bags and sets, and we'll get to those later. 

Perl's language designers feel that the variables should stand out from the language itself, and that the sigils (decorations on the front, cf. "https://en.wikipedia.org/wiki/Sigil_(computer_programming)") on the variables should give the reader an indication of the shape of the data.  Languages like Python, Ruby, Haskell, Java, make no visual distinction between their reserved words and variables.   

Time to fire up ```perl6``` and start typing.  The following show the result of typing the examples into the REPL.

## Scalar $

If you have just one of a thing like our greeting or name, then you put it into a "scalar" or singular variable.  These are prefixed with a ```$``` like ```$greeting``` and can hold only one value.  If you set it to a second value, the first value is forever lost unless you set it to be immutable using ```:=```.

```
> my $greeting = "How you doin'?";
How you doin'?
> $greeting.chars
14
> my $money := 1e+6;
1000000
> $money = 0
Cannot assign to an immutable value
  in block <unit> at <unknown file> line 1
```

## Array @

When you have an undetermined number of somethings, they belong in an Array (mutable) or a List (immutable).  These are plurals, and they start with the ```@``` sign.  The items in a series are separated with commas or you can use the ```<>``` operator (<https://docs.perl6.org/language/quoting#index-entry-%3C_%3E_word_quote>).  Perl also supports infinite lists -- just don't try to print them.

```
> my @nums = 8, 1, 43;
[8 1 43]
> put @nums.join(', ');
8, 1, 43
> my @sizes = <small medium large>
[small medium large]
> my @x = 1..10
[1 2 3 4 5 6 7 8 9 10]
> my @positive = 1..*
[...]
```

## Hash %

When you have a key-value association, that belongs in a hash AKA "map" or "dictionary" or "associative array," if you're not into the whole brevity thing.  Yes, I know we call the ```#``` a "hash," but this "hash" is short for a "hash table" (https://en.wikipedia.org/wiki/Hash_table).  Hashes start with the ```%``` sign.

```
> my %genome = species => "H_sapiens", taxid => 9606;
{species => H_sapiens, taxid => 9606}
> put %genome<species>;
H_sapiens
> %genome.keys
(species taxid)
```

In the case of each variable, something that should be very striking is that we can ask the variables to do things for us.  We can ask a scalar how many "chars" (characters) it has, we can have a list join its elements together using a comma to create a string that we can print, and we can ask the hash to give us the value for some given key or even all the keys it has.  We can even go meta (literally) and ask the variables what they can do for us.  I will elide the output here, but you should try this in your own REPL:

```
> $greeting.^methods
(BUILD Int Num chomp pred succ simplematch match ...)
> @nums.^methods
(iterator from-iterator new STORE reification-target shape pop shift splice ...)
> %genome.^methods
(clone BIND-KEY name keyof of default dynamic push append ...)
```

This only begins to scratch the surface.  You can read more at https://docs.perl6.org/type.html.