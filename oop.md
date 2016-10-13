# OOPs, I did it again

As we've learned, Perl has types like ```(Int)``` and ```(Str)```, and you can easily create your own types like a ```(File)``` with ```subset```:

```
subset File of Str where *.IO.f;
```

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

Let's 
