# Hashes

Hashes basically Arrays of Pairs.  They are also known as dictionaries or maps.  The key in a Hash is always, so we map a string can map strings (keys) to strings or numbers (values):

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