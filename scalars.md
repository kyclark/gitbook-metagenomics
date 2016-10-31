# Scalars

Let's look at some of the things that fit in a scalar.  This list isn't exhaustive, just an overture to what we'll see as we move through the text.

# Cool

"Cool" (https://docs.perl6.org/type/Cool) is short for "Convenient OO Loop" and can hold values that look either like numbers or strings, converting back and forth ("coercing," in the parlance) depending on how you use them.

# Numbers

Perl has many numeric types including Int (integer), UInt (unsigned integer), Num (floating point), Rat/FatRat/Rational, Real, and Complex.

# Strings

A "string" (https://docs.perl6.org/type/Str) is a series of characters like "GATTAGA."  If I put "2112" in quotes it's a string, but if I don't it's a number.  Here's how to create the reverse complement of a strand of DNA:

```
my $dna = "GATTAGA";
> $dna.trans(<A C G T> => <T G C A>).flip
TCTAATC
```

# Date, DateTime

The ```Date``` (https://docs.perl6.org/type/Date) and ```DateTime``` (https://docs.perl6.org/type/DateTime) types are pretty much what you'd expect:

```
> my $birthday = Date.new(1972, 5, 4);
1972-05-04
> $birthday.day-of-week
4 # Thursday
> Date.new(now).year - $birthday.year
44
> DateTime.new(now)
2016-10-31T03:27:46.544350Z
```

# File handles

If "foo.txt" is a file that lives on your system, then you can ```open``` the file to get a file handle that will allow you to read the file.

```
my $fh = open "foo.txt";
for $fh.lines -> $line { say $line }
# or
for $fh.lines { .say }
```

# Bool

Boolean types (https://docs.perl6.org/type/Bool, named for mathematician George Boole) are your typical ```True``` and ```False``` values, but be warned that they are a type of Enum (<https://docs.perl6.org/language/typesystem#index-entry-Enumeration-_Enums-_enum>)`, not a truely separate type.

```
> if True { put "true" } else { put "false" }
true
> True == 1
True
> 1 + True == 2
True
```

# Pair

A Pair (https://docs.perl6.org/type/Pair) is a combination of a key and a value.

```
> my $bp = A => 'C';
A => C
> $bp.key
A
> $bp.value
C
> $bp.kv
(A C)
```

# Proc

A "Proc" (process, https://docs.perl6.org/type/Proc) is the result of running a command outside of Perl such as "pwd":

```
> my $proc = run('pwd', :out)
Proc.new(in => IO::Pipe, out => IO::Pipe.new(:path(""),:chomp), err => IO::Pipe, exitcode => 0, pid => Any, signal => 0, command => ["pwd"])
>
> $proc.out.get
/Users/kyclark
```

# Sub

We'll talk about creating subroutines much later, but know that you can store them into a scalar, pass them around like data, and call them when you like.

```
> my $pony = sub { put "I dig a pony" }
sub () { #`(Sub|140377779867312) ... }
> $pony()
I dig a pony
```