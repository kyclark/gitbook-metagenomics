# Hashes

Hashes (https://docs.perl6.org/type/Hash) basically Arrays of Pairs (https://docs.perl6.org/type/Pair).  They are also known as dictionaries or maps.  The key in a Hash is usually a string, and the value can be pretty much anything.  The order of the Pairs is not preserved in creation order:

```
> my %dog = name => 'Patch', age => 4, color => 'white', weight => '31 lbs'
{age => 4, color => white, name => Patch, weight => 31 lbs}
```

If I ask for one key, I get one value:

```
> %dog<name>
Patch
```

I can also ask for more than one key, and I will get a list:

```
> %dog<age color>
(4 white)
```

I can ask for all the keys and values using the ```kv``` method:

```
> %dog.kv
(color white name Patch age 4 weight 31 lbs)
> %dog.pairs
(color => white name => Patch age => 4 weight => 31 lbs)
> say join ", ", %dog.map(*.kv.join(": "))
color: white, name: Patch, age: 4, weight: 31 lbs
> %dog.pairs.map(*.kv.join(": ")).join(", ")
color: white, name: Patch, age: 4, weight: 31 lbs
```

If I want the keys back in a particular order, I either have to ask for them explicitly or ```sort``` them:

```
> %dog.keys
(color name age weight)
> %dog.keys.sort
(age color name weight)
```

I can store more than one thing for a value by using lists as in this amino-acid-to-codons table:

```
$ cat -n aa.pl6
     1	#!/usr/bin/env perl6
     2
     3	my %aa = Isoleucine    => <ATT ATC ATA>,
     4	         Leucine       => <CTT CTC CTA CTG TTA TTG>,
     5	         Valine        => <GTT GTC GTA GTG>,
     6	         Phenylalanine => <TTT TTC>,
     7	         Methionine    => <ATG>,
     8	         Cysteine      => <TGT TGC>,
     9	         Alanine       => <GCT GCC GCA GCG>,
    10	         Glycine       => <GGT GGC GGA GGG>,
    11	         Proline       => <CCT CCC CCA CCG>,
    12	         Threonine     => <ACT ACC ACA ACG>,
    13	         Serine        => <TCT, TCC, TCA, TCG, AGT, AGC>,
    14	         Tyrosine      => <TAT TAC>,
    15	         Tryptophan    => <TGG>,
    16	         Glutamine     => <CAA CAG>,
    17	         Asparagine    => <AAT AAC>,
    18	         Histidine     => <CAT CAC>,
    19	         Glutamic_acid => <GAA GAG>,
    20	         Aspartic_acid => <GAT GAC>,
    21	         Lysine        => <AAA AAG>,
    22	         Arginine      => <CGT CGC CGA CGG AGA AGG>,
    23	         Stop          => <TAA TAG TGA>;
    24
    25	for %aa.keys.sort -> $key {
    26	    printf "%-15s = %s\n", $key, %aa{ $key }.join(", ");
    27	}
$ ./aa.pl6
Alanine         = GCA, GCC, GCG, GCT
Arginine        = AGA, AGG, CGA, CGC, CGG, CGT
Asparagine      = AAC, AAT
Aspartic_acid   = GAC, GAT
Cysteine        = TGC, TGT
Glutamic_acid   = GAA, GAG
Glutamine       = CAA, CAG
Glycine         = GGA, GGC, GGG, GGT
Histidine       = CAC, CAT
Isoleucine      = ATA, ATC, ATT
Leucine         = CTA, CTC, CTG, CTT, TTA, TTG
Lysine          = AAA, AAG
Methionine      = ATG
Phenylalanine   = TTC, TTT
Proline         = CCA, CCC, CCG, CCT
Serine          = AGC, AGT,, TCA,, TCC,, TCG,, TCT,
Stop            = TAA, TAG, TGA
Threonine       = ACA, ACC, ACG, ACT
Tryptophan      = TGG
Tyrosine        = TAC, TAT
Valine          = GTA, GTC, GTG, GTT
```

Hashes are great for counting things.  You don't have to check if a key/value pair exists first -- just add one to a key and it will be created and start counting from 0:

```
$ cat -n word-count.pl6
     1	#!/usr/bin/env perl6
     2
     3	sub MAIN (Str $file where *.IO.f) {
     4	    my %count;
     5	    for $file.IO.words -> $word {
     6	        %count{ $word }++;
     7	    }
     8
     9	    for %count.grep(*.value > 1) -> (:$key, :$value) {
    10	        printf "Saw '%s' %s time.\n", $key, $value;
    11	    }
    12	}
$ ./word-count.pl6 because.txt
Saw 'a' 3 times.
Saw 'of' 2 times.
Saw 'for' 2 times.
Saw 'I' 3 times.
Saw 'The' 3 times.
Saw 'but' 3 times.
Saw 'And' 2 times.
Saw 'We' 5 times.
Saw 'passed' 3 times.
Saw 'the' 6 times.
```

You can set the keys and values all-at-once like above or one-at-a-time:

```
$ cat -n bridge-of-death.pl6
     1	#!/usr/bin/env perl6
     2
     3	my %answers;
     4	for "name", "quest", "favorite color" -> $key {
     5	    %answers{ $key } = prompt "What is your $key? ";
     6	}
     7
     8	if %answers{"favorite color"} eq "Blue" {
     9	    put "Right.  Off you go.";
    10	}
    11	else {
    12	    put "AAAAAAAAAAaaaaaaaaa!"
    13	}
$ ./bridge-of-death.pl6
What is your name? My name is Sir Launcelot of Camelot
What is your quest? To seek the Holy Grail
What is your favorite color? Blue
Right.  Off you go.
$ ./bridge-of-death.pl6
What is your name? Sir Galahad of Camelot.
What is your quest? I seek the Holy Grail.
What is your favorite color? Blue.  No yel--
AAAAAAAAAAaaaaaaaaa!
```

Sometimes you need to see if a key is defined in a hash, and for that you use the adverb ```:exists```:

```
$ cat -n gashlycrumb.pl6
     1	#!/usr/bin/env perl6
     2
     3	# Text by Edward Gorey
     4
     5	my %alphabet = q:to/END/.lines.map(-> $line { $line.substr(0,1) => $line });
     6	A is for Amy who fell down the stairs.
     7	B is for Basil assaulted by bears.
     8	C is for Clara who wasted away.
     9	D is for Desmond thrown out of a sleigh.
    10	E is for Ernest who choked on a peach.
    11	F is for Fanny sucked dry by a leech.
    12	G is for George smothered under a rug.
    13	H is for Hector done in by a thug.
    14	I is for Ida who drowned in a lake.
    15	J is for James who took lye by mistake.
    16	K is for Kate who was struck with an axe.
    17	L is for Leo who choked on some tacks.
    18	M is for Maud who was swept out to sea.
    19	N is for Neville who died of ennui.
    20	O is for Olive run through with an awl.
    21	P is for Prue trampled flat in a brawl.
    22	Q is for Quentin who sank on a mire.
    23	R is for Rhoda consumed by a fire.
    24	S is for Susan who perished of fits.
    25	T is for Titus who flew into bits.
    26	U is for Una who slipped down a drain.
    27	V is for Victor squashed under a train.
    28	W is for Winnie embedded in ice.
    29	X is for Xerxes devoured by mice.
    30	Y is for Yorick whose head was bashed in.
    31	Z is for Zillah who drank too much gin.
    32	END
    33
    34	loop {
    35	    my $letter = uc prompt "Letter [0 to quit]? ";
    36
    37	    if $letter eq '0' {
    38	        put "Bye.";
    39	        exit;
    40	    }
    41	    elsif %alphabet{ $letter }:exists {
    42	        put %alphabet{ $letter };
    43	    }
    44	    else {
    45	        put "I'm sorry, I don't know that letter ($letter).";
    46	    }
    47	}
$ ./gashlycrumb.pl6
Letter [0 to quit]? j
J is for James who took lye by mistake.
Letter [0 to quit]? 9
I'm sorry, I don't know that letter (9).
Letter [0 to quit]? 0
Bye.
```