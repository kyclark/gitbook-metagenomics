# Hashes

Hashes basically Arrays of Pairs.  They are also known as dictionaries or maps.  The key in a Hash is always, so we map a string can map strings to strings:

```
> my %dna = A => 'C', T => 'G';
{A => C, T => G}
> %dna<A>
C
```

Or strings to numbers:

```
> my %nums = e => 2.71828, pi => 3.145927
{e => 2.71828, pi => 3.145927}
```

Although note that ```e``` and ```pi``` are built-in:

```
> e
2.71828182845905
> pi
3.14159265358979
```

We can even map strings to lists such as this amino acid to codon table:

```
my %aa =·
Isoleucine    => <ATT ATC ATA>,·
Leucine       => <CTT CTC CTA CTG TTA TTG>,·
Valine        => <GTT GTC GTA GTG>,·
Phenylalanine => <TTT TTC>,
Methionine    => <ATG>,
Cysteine      => <TGT TGC>,
Alanine       => <GCT GCC GCA GCG>,
Glycine       => <GGT GGC GGA GGG>,
Proline       => <CCT CCC CCA CCG>,
Threonine     => <ACT ACC ACA ACG>,
Serine        => <TCT, TCC, TCA, TCG, AGT, AGC>,
Tyrosine      => <TAT TAC>,
Tryptophan    => <TGG>,
Glutamine     => <CAA CAG>,
Asparagine    => <AAT AAC>,
Histidine     => <CAT CAC>,
Glutamic_acid => <GAA GAG>,
Aspartic_acid => <GAT GAC>,
Lysine        => <AAA AAG>,
Arginine      => <CGT CGC CGA CGG AGA AGG>,
Stop          => <TAA TAG TGA>;
put "Lysine = %aa<Lysine>";
```