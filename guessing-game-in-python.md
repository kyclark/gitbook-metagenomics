# Guessing Game

I firmly believe that writing your own simple games is a terrific way to learn how to program.  Let's write a simple program where the user has to guess a random number.  

```
$ ./guess.py
[0] Guess a number between 1 and 50 (q to quit): 25
You guessed "25"
Too high.
[1] Guess a number between 1 and 50 (q to quit): 12
You guessed "12"
Too low.
[2] Guess a number between 1 and 50 (q to quit): 20
You guessed "20"
Too high.
[3] Guess a number between 1 and 50 (q to quit): 17
You guessed "17"
Too low.
[4] Guess a number between 1 and 50 (q to quit): 18
You guessed "18"
Too many guesses! The number was "19."
```

To start, we'll use the "new\_py.py" script to stub out the boilerplate.  I'll use the `-a` flag to indicate that I want the program to use the `argparse` module so we can accept some named arguments to our script:

```
$ new_py.py -a guess
Done, see new script "guess.py."
$ cat -n guess.py
     1    #!/usr/bin/env python3
     2    """docstring"""
     3
     4    import argparse
     5    import sys
     6
     7    # --------------------------------------------------
     8    def get_args():
     9        """get args"""
    10        parser = argparse.ArgumentParser(description='Argparse Python script')
    11        parser.add_argument('positional', metavar='str', help='A positional argument')
    12        parser.add_argument('-a', '--arg', help='A named string argument',
    13                            metavar='str', type=str, default='')
    14        parser.add_argument('-i', '--int', help='A named integer argument',
    15                            metavar='int', type=int, default=0)
    16        parser.add_argument('-f', '--flag', help='A boolean flag',
    17                            action='store_true')
    18        return parser.parse_args()
    19
    20    # --------------------------------------------------
    21    def main():
    22        """main"""
    23        args = get_args()
    24        str_arg = args.arg
    25        int_arg = args.int
    26        flag_arg = args.flag
    27        pos_arg = args.positional
    28
    29        print('str_arg = "{}"'.format(str_arg))
    30        print('int_arg = "{}"'.format(int_arg))
    31        print('flag_arg = "{}"'.format(flag_arg))
    32        print('positional = "{}"'.format(pos_arg))
    33
    34    # --------------------------------------------------
    35    if __name__ == '__main__':
    36        main()
```

The template shows how to create named arguments for strings, integers, Booleans, as well as positional \(unnamed\) values.  For this program, we want to know the min/max numbers to guess and the number of guesses allowed.  We will provide reasonable defaults for all of them so that they will be completely optional.  The thing I like best about this step is that it makes me think carefully about what I expect from the user.  Change your program to this:

```
def get_args():
    """get args"""
    parser = argparse.ArgumentParser(description='Number guessing game')
    parser.add_argument('-m', '--min', help='Minimum value',
                        metavar='int', type=int, default=1)
    parser.add_argument('-x', '--max', help='Maximum value',
                        metavar='int', type=int, default=50)
    parser.add_argument('-g', '--guesses', help='Number of guesses',
                        metavar='int', type=int, default=5)
    return parser.parse_args()
```

Now in the `main` we will need to unpack the input:

```
def main():
    """main"""
    args = get_args()
    low = args.min
    high = args.max
    guesses_allowed = args.guesses
```

Alway assume you get garbage from the user, so let's check the input:

```
    if low < 1:
        print('--min cannot be lower than 1')
        sys.exit(1)

    if guesses_allowed < 1:
        print('--guesses cannot be lower than 1')
        sys.exit(1)

    if low > high:
        print('--min "{}" is higher than --max "{}"'.format(low, high))
        sys.exit(1)
```

The next thing we need is a random number between `--min` and `--max` for the user to guess.  We can `import random` to do:

```
secret = random.randint(low, high)
```

The meat of the program will be an infinite loop where we keep asking the user:

```
prompt = 'Guess a number between {} and {} (q to quit): '.format(low, high)
```

Before we enter that loop, we'll need a variable to keep track of the number of guesses the user has made:

```
num_guesses = 0
```

The beginning of the play loop looks like this:

```
    while True:
        guess = input('[{}] {}'.format(num_guesses, prompt))
        num_guesses += 1
```

Here I want the user to know how many guesses they've made so far.  We want to give them a way out, so they can enter "q" to quit:

```
        if guess == 'q':
            print('Now you will never know the answer.')
            sys.exit(0)
```

The input from the user will be a string, and we are going to need to convert it to an integer to see if it is the secret number.  Before we do that, we must check that it is a digit:

```
        if not guess.isdigit():
            print('"{}" is not a number'.format(guess))
            continue
```

If it's not a digit, we `continue` to go to the next iteration of the loop.  If we move ahead, then it's OK to convert the guess:

```
        print('You guessed "{}"'.format(guess))
        num = int(guess)
```

Now we need to determine if the user has guessed too many times, if the number if too high or low, or if they've won the game:

```

        if num_guesses >= guesses_allowed:
            print('Too many guesses! The number was "{}."'.format(secret))
            sys.exit()
        elif num < low or num > high:
            print('Number is not in the allowed range')
        elif num == secret:
            print('You win!')
            break
        elif num < secret:
            print('Too low.')
        else:
            print('Too high.')
```

The final version looks like this:

```
$ cat -n guess.py
     1	#!/usr/bin/env python3
     2	"""guess the number game"""
     3
     4	import argparse
     5	import random
     6	import sys
     7
     8	# --------------------------------------------------
     9	def get_args():
    10	    """get args"""
    11	    parser = argparse.ArgumentParser(description='Number guessing game')
    12	    parser.add_argument('-m', '--min', help='Minimum value',
    13	                        metavar='int', type=int, default=1)
    14	    parser.add_argument('-x', '--max', help='Maximum value',
    15	                        metavar='int', type=int, default=50)
    16	    parser.add_argument('-g', '--guesses', help='Number of guesses',
    17	                        metavar='int', type=int, default=5)
    18	    return parser.parse_args()
    19
    20	# --------------------------------------------------
    21	def main():
    22	    """main"""
    23	    args = get_args()
    24	    low = args.min
    25	    high = args.max
    26	    guesses_allowed = args.guesses
    27
    28	    if low < 1:
    29	        print('--min cannot be lower than 1')
    30	        sys.exit(1)
    31
    32	    if guesses_allowed < 1:
    33	        print('--guesses cannot be lower than 1')
    34	        sys.exit(1)
    35
    36	    if low > high:
    37	        print('--min "{}" is higher than --max "{}"'.format(low, high))
    38	        sys.exit(1)
    39
    40	    secret = random.randint(low, high)
    41	    num_guesses = 0
    42	    prompt = 'Guess a number between {} and {} (q to quit): '.format(low, high)
    43
    44	    while True:
    45	        guess = input('[{}] {}'.format(num_guesses, prompt))
    46	        num_guesses += 1
    47
    48	        if guess == 'q':
    49	            print('Now you will never know the answer.')
    50	            sys.exit(0)
    51
    52	        if not guess.isdigit():
    53	            print('"{}" is not a number'.format(guess))
    54	            continue
    55
    56	        print('You guessed "{}"'.format(guess))
    57	        num = int(guess)
    58
    59	        if num_guesses >= guesses_allowed:
    60	            print('Too many guesses! The number was "{}."'.format(secret))
    61	            sys.exit()
    62	        elif num < low or num > high:
    63	            print('Number is not in the allowed range')
    64	        elif num == secret:
    65	            print('You win!')
    66	            break
    67	        elif num < secret:
    68	            print('Too low.')
    69	        else:
    70	            print('Too high.')
    71
    72	# --------------------------------------------------
    73	if __name__ == '__main__':
    74	    main()
```

# Hangman

Here is an implementation of the game "Hangman" that uses dictionaries to maintain the "state" of the program -- that is, all the information needed for each round of play such as the word being guessed, how many misses the user has made, which letters have been guessed, etc.  The program uses the `argparse` module to gather options from the user while providing default values so that nothing needs to be provided.  The `main` function is used just to gather the parameters and then run the `play` function which recursively calls itself, each time passing in the new "state" of the program.  Inside `play`, we use the `get` method of `dict` to safely ask for keys that may not exist and use defaults.  When the user finishes or quits, `play` will simply call `sys.exit` to stop.  Here is the code:

```
$ cat -n hangman.py
     1    #!/usr/bin/env python3
     2    """Hangman game"""
     3
     4    import argparse
     5    import os
     6    import random
     7    import re
     8    import sys
     9
    10    # --------------------------------------------------
    11    def get_args():
    12        """parse arguments"""
    13        parser = argparse.ArgumentParser(description='Hangman')
    14        parser.add_argument('-l', '--maxlen', help='Max word length',
    15                            type=int, default=10)
    16        parser.add_argument('-n', '--minlen', help='Min word length',
    17                            type=int, default=5)
    18        parser.add_argument('-m', '--misses', help='Max number of misses',
    19                            type=int, default=10)
    20        parser.add_argument('-w', '--wordlist', help='Word list',
    21                            type=str, default='/usr/share/dict/words')
    22        return parser.parse_args()
    23
    24    # --------------------------------------------------
    25    def main():
    26        """main"""
    27        args = get_args()
    28        max_len = args.maxlen
    29        min_len = args.minlen
    30        max_misses = args.misses
    31        wordlist = args.wordlist
    32
    33        if not os.path.isfile(wordlist):
    34            print('--wordlist "{}" is not a file.'.format(wordlist))
    35            sys.exit(1)
    36
    37        if min_len < 1:
    38            print('--minlen must be positive')
    39            sys.exit(1)
    40
    41        if not 3 <= max_len <= 20:
    42            print('--maxlen should be between 3 and 20')
    43            sys.exit(1)
    44
    45        if min_len > max_len:
    46            print('--minlen ({}) is greater than --maxlen ({})'.format(min_len, max_len))
    47            sys.exit(1)
    48
    49        regex = re.compile('^[a-z]{' + str(min_len) + ',' + str(max_len) + '}$')
    50        words = [w for w in open(wordlist).read().split() if regex.match(w)]
    51        word = random.choice(words)
    52        play({'word': word, 'max_misses': max_misses})
    53
    54    # --------------------------------------------------
    55    def play(state):
    56        """Loop to play the game"""
    57        word = state.get('word') or ''
    58
    59        if not word:
    60            print('No word!')
    61            sys.exit(1)
    62
    63        guessed = state.get('guessed') or list('_' * len(word))
    64        prev_guesses = state.get('prev_guesses') or set()
    65        num_misses = state.get('num_misses') or 0
    66        max_misses = state.get('max_misses') or 0
    67
    68        if ''.join(guessed) == word:
    69            msg = 'You win. You guessed "{}" with "{}" miss{}!'
    70            print(msg.format(word, num_misses, '' if num_misses == 1 else 'es'))
    71            sys.exit(0)
    72
    73        if num_misses >= max_misses:
    74            print('You lose, loser!  The word was "{}."'.format(word))
    75            sys.exit(0)
    76
    77        print('{} (Misses: {})'.format(' '.join(guessed), num_misses))
    78        new_guess = input('Your guess? ("?" for hint, "!" to quit) ').lower()
    79
    80        if new_guess == '!':
    81            print('Better luck next time, loser.')
    82            sys.exit(0)
    83        elif new_guess == '?':
    84            new_guess = random.choice([x for x in word if x not in guessed])
    85            num_misses += 1
    86
    87        if not re.match('^[a-zA-Z]$', new_guess):
    88            print('"{}" is not a letter'.format(new_guess))
    89            num_misses += 1
    90        elif new_guess in prev_guesses:
    91            print('You already guessed that')
    92        elif new_guess in word:
    93            prev_guesses.add(new_guess)
    94            last_pos = 0
    95            while True:
    96                pos = word.find(new_guess, last_pos)
    97                if pos < 0:
    98                    break
    99                elif pos >= 0:
   100                    guessed[pos] = new_guess
   101                    last_pos = pos + 1
   102        else:
   103            num_misses += 1
   104
   105        play({'word': word, 'guessed': guessed, 'num_misses': num_misses,
   106              'prev_guesses': prev_guesses, 'max_misses': max_misses})
   107
   108    # --------------------------------------------------
   109    if __name__ == '__main__':
   110        main()
```



