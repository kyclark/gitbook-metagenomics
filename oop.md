# OOPs, I did it again

As we've learned, Perl has types like ```(Int)``` and ```(Str)```, and you can easily create your own types like a ```(File)``` with ```subset```:

```
subset File of Str where *.IO.f;
```

# DNA

Let's say we'd like to have an object to represent DNA sequences.  It's as simple as creating a ```class``` where we say that is ```has Str $.seq```:

```
$ cat -n dna1.pl6
     1	#!/usr/bin/env perl6
     2
     3	class DNA {
     4	    has Str $.seq;
     5	}
     6
     7	sub MAIN (Str $seq) {
     8	    my $dna = DNA.new(seq => $seq);
     9	    dd $dna;
    10	}
```

The ```has``` keyword will create accessor/mutator methods called ```seq``` for us to get (access) or change (mutate, if we so allow) the sequence.  By default, object attributes are read-only, so we'd have to explicitly say ```is rw``` to declare that it is read-write.

Here's what we get:

```
$ ./dna1.pl6 CAT
DNA $dna = DNA.new(seq => "CAT")
$ ./dna1.pl6 foo
DNA $dna = DNA.new(seq => "foo")
```

Well, that's a problem.  The string "foo" is clearly not a DNA sequence, so we should have some way to detect that and let the user know.  We've already used regular expressions for exactly this:

```
$ cat -n dna2.pl6
     1	#!/usr/bin/env perl6
     2
     3	class DNA {
     4	    has Str $.seq;
     5	}
     6
     7	sub MAIN (Str $seq) {
     8	    if so $seq.uc ~~ /^ <[ACGTN]>+ $/ {
     9	        my $dna = DNA.new(seq => $seq);
    10	        dd $dna;
    11	    }
    12	    else {
    13	        put "Not a DNA sequence.";
    14	    }
    15	}
$ ./dna2.pl6 CAT
DNA $dna = DNA.new(seq => "CAT")
$ ./dna2.pl6 foo
Not a DNA sequence.
```

OK, that works, but it's not a clean solution because the checking of the input really ought to happen inside the object.  Eventually we'll isolate the ```class``` part to make it reusable every place we want to represent DNA.  

# Hangman

You may find you want to create a more complex type that can encapsulate complex data and novel methods.  To demonstrate, let's create a "Puzzle" object for playing "Hangman."  Here's how a game looks when won:

```
$ ./hangman.pl6
puzzle = _ _ _ _ _ _ _ []
What is your guess? a
puzzle = _ _ _ _ _ _ _ [a]
What is your guess? i
puzzle = _ _ _ _ _ _ _ [ai]
What is your guess? e
puzzle = _ e _ _ _ e _ [aei]
What is your guess? o
puzzle = _ e _ _ _ e _ [aeio]
What is your guess? u
puzzle = _ e _ _ u e _ [aeiou]
What is your guess? s
puzzle = _ e s _ u e _ [aeiosu]
What is your guess? t
puzzle = _ e s _ u e _ [aeiostu]
What is your guess? r
puzzle = r e s _ u e r [aeiorstu]
What is your guess? c
puzzle = r e s c u e r [aceiorstu]
You won!
```

The game needs to have a random dictionary word which we can easily pluck from "/usr/share/dict/words" of some minimum (default 5) and maximum (default 9) length.  During the game, we need to know the word, what the user has guessed so far, and how many of the letters have been found.  After each guess, we need to decide if the puzzle has been solved or if the user has exceeded the allowed number of attempts (default 10).  While it's not at all necessary to use a new type/object to handle this, we can see quite a benefit to wrapping all the puzzle logic up in one place and the user interface in another.  Here's the code:

```
#!/usr/bin/env perl6

class Puzzle {
    has Str $.word;
    has Str @.state;
    has Int %.guesses;

    submethod BUILD (Str :$word) {
        $!word  = $word;
        @!state = '_' xx $word.chars;
    }

    method Str {
        "puzzle = {@.state.join(' ')} [{%.guesses.keys.sort.join}]";
    }

    method guess (Str $char) {
        unless %.guesses{ $char }++ {
            for $.word.indices($char) -> $i {
                @.state[$i] = $char;
            }
        }
    }

    method was-guessed (Str $char) {
        return %.guesses{ $char }.defined;
    }

    method number-guessed {
        return %.guesses.keys.elems;
    }

    method is-solved {
        none(@.state) eq '_';
    }
}

sub MAIN(Int :$num-guesses = 10, :$min-word-len=5, :$max-word-len=9) {
    my $words  = '/usr/share/dict/words';
    my $word   = $words.IO.lines.grep(
                 {$min-word-len <= .chars <= $max-word-len}).pick.lc;
    my $puzzle = Puzzle.new(word => $word);

    loop {
        put ~$puzzle;
        if $puzzle.is-solved {
            put "You won!";
            last;
        }

        if $puzzle.number-guessed >= $num-guesses {
            put "Too many guesses. The word was '$word\.' You lose.";
            last;
        }

        my $guess = (prompt "What is your guess? ").lc;
        if $guess !~~ m:i/^<[a..z]>$/ {
            put "Please guess just one letter";
            next;
        }

        if $puzzle.was-guessed($guess) {
            put "You guessed that before!";
            next;
        }

        $puzzle.guess($guess);
    }
}
```

The ```Puzzle``` holds the current state of the game.  It knows the word which is being guessed, which letters have been guessed so far (and therefore how many guesses have been made), and which letters have been found.  When we ```loop``` through the game, we just need to ask the ```Puzzle``` for information like if the game is over either because too many guesses were made or because the puzzle has been solved.  The game could be written entirely without objects, but objects give us a way to wrap up state -- how things change through time -- with methods -- how to change things, how to ask about change.

# Bouncy Balls

Here's a simple game that bounces balls inside a box in your terminal.  When two balls collide, they explode with a Unicode star "★" and disappear from the board:

```
     1	#!/usr/bin/env perl6
     2	
     3	subset PosInt of Int where * > 0;
     4	
     5	class Ball {
     6	    has Int $.rows;
     7	    has Int $.cols;
     8	    has Int $.row is rw = (2..^$!rows).pick;
     9	    has Int $.col is rw = (2..^$!cols).pick;
    10	    has Int $.horz-dir is rw = (1, -1).pick;
    11	    has Int $.vert-dir is rw = (1, -1).pick;
    12	
    13	    method Str { join ',', $!row, $!col }
    14	
    15	    method pos { ($.row, $.col) }
    16	
    17	    method move {
    18	        $!col += $!horz-dir;
    19	        $!row += $!vert-dir;
    20	        $!horz-dir *= -1 if $!col <= 1 || $!col >= $!cols;
    21	        $!vert-dir *= -1 if $!row <= 1 || $!row >= $.rows;
    22	    }
    23	}
    24	
    25	my $DOT           = "\x25A0"; # ■
    26	my $STAR          = "\x2605"; # ★
    27	my $SMILEY-FACE   = "\x263A"; # ☺
    28	my ($ROWS, $COLS) = qx/stty size/.words;
    29	
    30	sub MAIN (
    31	    PosInt :$rows=$ROWS - 4,
    32	    PosInt :$cols=$COLS - 2,
    33	    PosInt :$balls=10,
    34	    Numeric :$refresh=.075,
    35	    Bool :$smiley=False,
    36	) {
    37	    print "\e[2J";
    38	    my Str $bar    = '+' ~ '-' x $cols ~ '+';
    39	    my $icon       = $smiley ?? $SMILEY-FACE !! $DOT;
    40	    my Ball @balls = Ball.new(:$rows, :$cols) xx $balls;
    41	
    42	    loop {
    43	        .move for @balls;
    44	
    45	        my $positions = (@balls».Str).Bag;
    46	
    47	        my %row;
    48	        for $positions.list -> (:$key, :$value) {
    49	            my ($row, $col) = $key.split(',');
    50	            %row{ $row }.append: $col => $value;
    51	        }
    52	
    53	        print "\e[H";
    54	        my $screen = "$bar\n";
    55	
    56	        for 1..$rows -> $this-row {
    57	            my $line = '|' ~ " " x $cols;
    58	            if %row{ $this-row }:exists {
    59	                for %row{ $this-row }.list -> (:$key, :$value) {
    60	                    $line.substr-rw($key, 1) = $value == 1 ?? $icon !! $STAR;
    61	                }
    62	            }
    63	
    64	            $screen ~= "$line|\n";
    65	        }
    66	
    67	        $screen ~= $bar;
    68	        put $screen;
    69	        sleep $refresh;
    70	        my @collisions = $positions.grep(*.value > 1).map(*.key);
    71	        @balls = @balls.grep(none(@collisions) eq *.Str);
    72	    }
    73	}
```

While it's not a requirement to use objects for this game, they do allow me to wrap up the state of each ball into a little package that I can interact with by saying ```move``` to get the ball to figure: 

1. know its current location
2. know its current direction (up/down, left/right)
3. reverse its course if it encounters a boundary

Once I've ```move```d all the balls, I can get all their positions and draw them to the screen.  I need to know if more than one ball occupies any location, so I ```Bag``` them up to get counts.  If a position has more than one ball, I use the star icon to indicate the collision and remove the offenders from the ```@balls``` array.