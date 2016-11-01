# Scalars

Let's look at some of the things that fit in a scalar.  This list isn't exhaustive, just an overture to what we'll see as we move through the text.

# Numbers

Perl has many numeric types including Int (integer), UInt (unsigned integer), Num (floating point), Rat/FatRat/Rational, Real, and Complex.

# Strings

A "string" (https://docs.perl6.org/type/Str) is a series of characters like "GATTAGA."  If I put "2112" in quotes it's a string, but if I don't it's a number.  Here's how to create the reverse complement of a strand of DNA:

```
my $dna = "GATTAGA";
> $dna.trans(<A C G T> => <T G C A>).flip
TCTAATC
```

# Cool

"Cool" (https://docs.perl6.org/type/Cool) is short for "Convenient OO Loop" and can hold values that look either like numbers or strings, converting back and forth ("coercing," in the parlance) depending on how you use them.  

Let's look at a situation where I have a string that I bounce back and forth from being a string or a number:

```
> my $x = "42"
42
> put "$x + 1 = " ~ $x + 1
42 + 1 = 43
```

Lots going on here.  First ```$x``` is a string "42," and in ```put "$x + 1 = "``` it's treated like a string because it's being interpolated inside quotes.  In ```$x + 1```, ```$x``` is coerced to a number because I'm adding it (```+```):

```
> "42" + 1
43
> say ("42" + 1).WHAT
(Int)
```

but then to concantenate it (via ```~```) to the other string, the sum of ```42 + 1``` is coerced to a string:

```
> say ("$x + 1 = " ~ "42" + 1).WHAT
(Str)
```

You can declare that your variables can only contain a certain Type (https://docs.perl6.org/type.html):

```
> my Int $x = "42"
Type check failed in assignment to $x; expected Int but got Str ("42")
  in block <unit> at <unknown file> line 1
> my Str $x = 42;
Type check failed in assignment to $x; expected Str but got Int (42)
  in block <unit> at <unknown file> line 1
```

But Perl will still coerce values according to how you use them:

```
> my Str $x = "42";
42
> say $x.WHAT
(Str)
> say $x.Int.WHAT
(Int)
> say (+$x).WHAT
(Int)
> put "$x + 1 = " ~ $x + 1
42 + 1 = 43
```
# Date, DateTime

The ```Date``` (https://docs.perl6.org/type/Date) and ```DateTime``` (https://docs.perl6.org/type/DateTime) types are pretty much what you'd expect:

```
> my $birthday = Date.new(1972, 5, 4);
1972-05-04
> $birthday.day-of-week
4 # Thursday
> DateTime.new(now)
2016-10-31T16:25:09.335788Z
> Date.new(now).year - $birthday.year
44
> my $days = Date.new(now) - $birthday
16251
> $days div 365, $days % 365
(44 191) # (years, days)
```

# File handles (IO)

If "foo.txt" is a file that lives on your system, then you can ```open``` the file to get a file handle that will allow you to read the file.  This is a function of I/O (input/output), and lots of interesting types inhabit IO (https://docs.perl6.org/type/IO):

```
my $fh = open "foo.txt";
for $fh.lines -> $line { say $line }
# or
for $fh.lines { .say }
```

# Bool

Boolean types (https://docs.perl6.org/type/Bool, named for mathematician George Boole) are your typical ```True``` and ```False``` values, but be warned that they are a type of Enum (<https://docs.perl6.org/language/typesystem#index-entry-Enumeration-_Enums-_enum>), not a truely separate type.

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