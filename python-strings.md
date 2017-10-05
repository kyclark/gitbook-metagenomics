# Strings, Lists, and Tuples

There's some overlap among Python's strings, lists, and tuples.  In a way, you could think of strings as lists of characters \(Haskell certainly does\).  Many list operations work exactly the same over strings:

```
>>> names = ['Larry', 'Moe', 'Curly', 'Shemp']
>>> names[0]
'Larry'
>>> names[2:4]
['Curly', 'Shemp']
>>> 'Curly'[0]
'C'
>>> 'Curly'[2:4]
'rl'
>>> ', '.join(names)
'Larry, Moe, Curly, Shemp'
>>> ', '.join(names[0])
'L, a, r, r, y'
>>> 'Moe' in names
True
>>> 'r' in 'Larry'
True
>>> 'x' in 'Larry'
False
>>> 'Joe' in names
False
>>> names = ['Larry', 'Moe', 'Curly', 'Shemp']
>>> for name in names:
...   print(name)
...
Larry
Moe
Curly
Shemp
>>> for i, name in enumerate(names):
...   print('{:3} {}'.format(i, name))
...
  0 Larry
  1 Moe
  2 Curly
  3 Shemp
```

For an example, here is a simple program to determine if a given string is a palindrome

```
$ cat -n word_is_palindrome.py      1	#!/usr/bin/env python3
     2	"""Report if the given word is a palindrome"""
     3
     4	import sys
     5	import os
     6
     7	args = sys.argv
     8
     9	if len(args) != 2:
    10	    print('Usage: {} STR'.format(os.path.basename(args[0])))
    11	    sys.exit(1)
    12
    13	word = args[1]
    14	rev = ''.join(list(reversed(word)))
    15	print('"{}" is{} a palindrome.'.format(word, '' if word.lower() == rev.lower() else ' NOT'))
```

FWIW, we can extrapolate from this to a program that finds all palindromes in a file:

```
$ cat -n find_palindromes.py
     1	#!/usr/bin/env python3
     2	"""Report if the given word is a palindrome"""
     3
     4	import sys
     5	import os
     6
     7	args = sys.argv
     8
     9	if len(args) != 2:
    10	    print('Usage: {} FILE'.format(os.path.basename(args[0])))
    11	    sys.exit(1)
    12
    13	file = args[1]
    14
    15	if not os.path.isfile(file):
    16	    print('"{}" is not a file'.format(file))
    17	    sys.exit(1)
    18
    19	for line in open(file):
    20	    for word in line.lower().split():
    21	        if len(word) > 2:
    22	            rev = ''.join(list(reversed(word)))
    23	            if rev == word:
    24	                print(word)
```

You could compress lines 19-20 into this \(see "find\_palindromes2.py"\):

```
for word in open(file).read().lower().split():
```

# Tetranucleotide Composition

A common operation in bioinformatics is to determine sequence composition.  Here is a program to find the frequencies of the DNA bases \(A, C, T, G\):

```
$ cat -n dna1.py
     1	#!/usr/bin/env python3
     2	"""Tetra-nucleotide counter"""
     3
     4	import sys
     5	import os
     6
     7	args = sys.argv[1:]
     8
     9	if len(args) != 1:
    10	    print('Usage: {} DNA'.format(os.path.basename(sys.argv[0])))
    11	    sys.exit(1)
    12
    13	dna = args[0]
    14
    15	count_a, count_c, count_g, count_t = 0, 0, 0, 0
    16
    17	for letter in dna:
    18	    if letter == 'a' or letter == 'A':
    19	        count_a += 1
    20	    elif letter == 'c' or letter == 'C':
    21	        count_c += 1
    22	    elif letter == 'g' or letter == 'G':
    23	        count_g += 1
    24	    elif letter == 't' or letter == 'T':
    25	        count_t += 1
    26
    27	print(' '.join([str(count_a), str(count_c), str(count_g), str(count_t)]))
$ ./dna1.py AACCTAG
3 2 1 1
```

On line 15, we initiate four variables to count each DNA base.  Just as we can use a `for` loop to iterate through a list, we can iterate through each letter in a string on line 17.  We need to check for both upper- and lowercase strings to determine which counter to increment.  Line 27 points out that the "count\_\*" variables are numbers that must be converted to strings in order to `print` them.

To save quite a bit of typing, let's force the input sequence to lowercase:

```
$ cat -n dna2.py
     1	#!/usr/bin/env python3
     2	"""Tetra-nucleotide counter"""
     3
     4	import sys
     5	import os
     6
     7	args = sys.argv[1:]
     8
     9	if len(args) != 1:
    10	    print('Usage: {} DNA'.format(os.path.basename(sys.argv[0])))
    11	    sys.exit(1)
    12
    13	dna = args[0]
    14
    15	count_a, count_c, count_g, count_t = 0, 0, 0, 0
    16
    17	for letter in dna.lower():
    18	    if letter == 'a':
    19	        count_a += 1
    20	    elif letter == 'c':
    21	        count_c += 1
    22	    elif letter == 'g':
    23	        count_g += 1
    24	    elif letter == 't':
    25	        count_t += 1
    26
    27	print(' '.join([str(count_a), str(count_c), str(count_g), str(count_t)]))
```

There are better ways than this to count the characters, but we'll save this until we talk about dictionaries.

# Run-length Encoding

Along the lines of counting characters in a string, we can write a very simple string compression program that encodes repetitions of characters:

```
$ ./compress.py AAACAATTTTGGGGGAC
A3CA2T4G5AC
$ cat -n compress.py
     1	#!/usr/bin/env python3
     2	"""Compress text/DNA by marking repeated letters"""
     3
     4	import os
     5	import sys
     6
     7	args = sys.argv[1:]
     8
     9	if len(args) != 1:
    10	    print('Usage: {} ARG'.format(os.path.basename(sys.argv[0])))
    11	    sys.exit(1)
    12
    13	arg = args[0]
    14	text = ''
    15	if os.path.isfile(arg):
    16	    text = ''.join(open(arg).read().split())
    17	else:
    18	    text = arg.strip()
    19
    20	if len(text) == 0:
    21	    print('No usable text')
    22	    sys.exit(1)
    23
    24	counts = []
    25	count = 0
    26	prev = None
    27	for letter in text:
    28	    if prev is None:
    29	        prev = letter
    30	        count = 1
    31	    elif letter == prev:
    32	        count += 1
    33	        prev = letter
    34	    else:
    35	        counts.append((prev, count))
    36	        count = 1
    37	        prev = letter
    38
    39	# get the last letter after we fell out of the loop
    40	counts.append((prev, count))
    41
    42	for letter, count in counts:
    43	    print('{}{}'.format(letter, '' if count == 1 else count), end='')
    44
    45	print('')
```

This program makes use of a `counts` list to keep track of each letter we saw.  We add a "tuple" to the list:

```
>>> counts = []
>>> counts.append(('A', 3))
>>> counts
[('A', 3)]
>>> counts.append(('C', 1))
>>> counts
[('A', 3), ('C', 1)]
```

Tuples are similar to lists, but they are immutable:

```
>>> tup = ('white', 'dog')
>>> tup[1]
'dog'
>>> tup[1] = 'cat'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'tuple' object does not support item assignment
```

You see they are subscripted like strings and lists, but you cannot change a value inside a tuple.  Tuples are not limited to pairs:

```
>>> tup = ('white', 'dog', 'bird')
>>> tup[-1]
'bird'
```

# tac 

We all know and love the venerable `cat` program, but do you know about `tac`?  It prints a file in reverse.  We can use lists in Python to read a file into list and `reverse` it:

```
$ cat input.txt
first line
second line
third line
fourth line
$ ./tac1.py input.txt
fourth line
third line
second line
first line
```

Here is the code:

```
$ cat -n tac1.py
     1	#!/usr/bin/env python3
     2	"""print a file in reverse"""
     3
     4	import sys
     5	import os
     6
     7	args = sys.argv[1:]
     8	if len(args) != 1:
     9	    print('Usage: {} FILE'.format(os.path.basename(sys.argv[0])))
    10	    sys.exit(1)
    11
    12	file = args[0]
    13	if not os.path.isfile(file):
    14	    print('"{}" is not a file'.format(file))
    15	    sys.exit(1)
    16
    17	lines = []
    18	for line in open(file):
    19	    lines.append(line)
    20
    21	lines.reverse()
    22
    23	for line in lines:
    24	    print(line, end='')
```

We initialize a new list on line 17, then read through the file line-by-line and call the `append` method to add the line to the end of our list.  Then we call `reverse` to mutate the list **IN PLACE**:

```
>>> names
['Larry', 'Moe', 'Curly', 'Shemp']
>>> names.reverse()
>>> names
['Shemp', 'Curly', 'Moe', 'Larry']
```

After `reverse` we see that the `names` are permanently changed.  We can put them back with another call:

```
>>> names.reverse()
>>> names
['Larry', 'Moe', 'Curly', 'Shemp']
```

If we had simply wanted to use them in a reversed order **WITHOUT ALTERING THE ACTUAL LIST**, we could call the `reversed` function:

```
>>> list(reversed(names))
['Shemp', 'Curly', 'Moe', 'Larry']
>>> names
['Larry', 'Moe', 'Curly', 'Shemp']
```

It's really easy to read an entire file directly into a list with `readline()`, but you should be sure that you have at least as much memory on your machine as the file is big:

```
$ cat -n tac2.py
     1	#!/usr/bin/env python3
     2	"""print a file in reverse"""
     3
     4	import sys
     5	import os
     6
     7	args = sys.argv[1:]
     8	if len(args) != 1:
     9	    print('Usage: {} FILE'.format(os.path.basename(sys.argv[0])))
    10	    sys.exit(1)
    11
    12	file = args[0]
    13	if not os.path.isfile(file):
    14	    print('"{}" is not a file'.format(file))
    15	    sys.exit(1)
    16
    17	lines = open(file).readlines()
    18	lines.reverse()
    19
    20	for line in lines:
    21	    print(line, end='')
```

This version shows using the `reversed` function:

```
$ cat -n tac3.py
     1	#!/usr/bin/env python3
     2	"""print a file in reverse"""
     3
     4	import sys
     5	import os
     6
     7	args = sys.argv[1:]
     8	if len(args) != 1:
     9	    print('Usage: {} FILE'.format(os.path.basename(sys.argv[0])))
    10	    sys.exit(1)
    11
    12	file = args[0]
    13	if not os.path.isfile(file):
    14	    print('"{}" is not a file'.format(file))
    15	    sys.exit(1)
    16
    17	lines = open(file).readlines()
    18
    19	for line in reversed(lines):
    20	    print(line, end='')
```

And finally I will introduce the `with/open` convention that you will see in Python:

```
$ cat -n tac4.py
     1	#!/usr/bin/env python3
     2	"""print a file in reverse"""
     3
     4	import sys
     5	import os
     6
     7	args = sys.argv[1:]
     8	if len(args) != 1:
     9	    print('Usage: {} FILE'.format(os.path.basename(sys.argv[0])))
    10	    sys.exit(1)
    11
    12	file = args[0]
    13	if not os.path.isfile(file):
    14	    print('"{}" is not a file'.format(file))
    15	    sys.exit(1)
    16
    17	with open(file) as fh:
    18	    lines = fh.readlines()
    19	    for line in reversed(lines):
    20	        print(line, end='')
```

# Picnic

Here is a little memory game you might have played with your bored siblings on family car trips:

```
$ ./picnic.py
What are you bringing? [q to quit] chips
We'll have chips.
What else are you bringing? [q to quit] ham sammich
We'll have chips and ham sammich.
What else are you bringing? [q to quit] Coke
We'll have chips, ham sammich, and Coke.
What else are you bringing? [q to quit] cupcakes
We'll have chips, ham sammich, Coke, and cupcakes.
What else are you bringing? [q to quit] apples
We'll have chips, ham sammich, Coke, cupcakes, and apples.
What else are you bringing? [q to quit] q
Bye.
```

Each person introduces a new item, and the other person has to remember all the previous items and add a new one.  This is a classic "stack" that can be implemented with lists:

```
$ cat -n picnic.py
     1	#!/usr/bin/env python3
     2	"""What are you bringing to the picnic?"""
     3
     4	# --------------------------------------------------
     5	def joiner(items):
     6	    """properly conjuct items"""
     7	    num_items = len(items)
     8	    if num_items == 0:
     9	        return ''
    10	    elif num_items == 1:
    11	        return items[0]
    12	    elif num_items == 2:
    13	        return ' and '.join(items)
    14	    else:
    15	        items[-1] = 'and ' + items[-1]
    16	        return ', '.join(items)
    17
    18	# --------------------------------------------------
    19	def main():
    20	    """start here"""
    21	    items = []
    22
    23	    while True:
    24	        item = input('What {}are you bringing? [q to quit] '.format('else ' if items else ''))
    25	        if item == 'q':
    26	            break
    27	        elif len(item.strip()) > 0:
    28	            if item in items:
    29	                print('You said "{}" already.'.format(item))
    30	            else:
    31	                items.append(item)
    32	                print("We'll have {}.".format(joiner(items.copy())))
    33
    34	    print('Bye.')
    35
    36	# --------------------------------------------------
    37	if __name__ == '__main__':
    38	    main()
```

One bug that got me in writing this program was line 32.  Because I mutate the last item in the list in my `joiner` function, I was actually mutating the original list!  I had to learn to pass `items.copy()` so as to work on a copy of the data and not the actual list.

Sometimes \(esp when writing games\) you may want a random selection from a list of items.  Here is an insult generator that draws from the fabulous vocabulary of Shakespeare:

```
$ cat -n insult.py
     1	#!/usr/bin/env python3
     2	"""Shakespearean insult generator"""
     3
     4	import sys
     5	import random
     6
     7	ADJECTIVES = """
     8	scurvy old filthy scurilous lascivious foolish rascaly gross rotten corrupt
     9	foul loathsome irksome heedless unmannered whoreson cullionly false filthsome
    10	toad-spotted caterwauling wall-eyed insatiate vile peevish infected
    11	sodden-witted lecherous ruinous indistinguishable dishonest thin-faced
    12	slanderous bankrupt base detestable rotten dishonest lubbery
    13	""".split()
    14
    15	NOUNS = """
    16	knave coward liar swine villain beggar slave scold jolthead whore barbermonger
    17	fishmonger carbuncle fiend traitor block ape braggart jack milksop boy harpy
    18	recreant degenerate Judas butt cur Satan ass coxcomb dandy gull minion
    19	ratcatcher maw fool rogue lunatic varlet worm
    20	""".split()
    21
    22	args = sys.argv[1:]
    23	num = 5
    24	if len(args) > 0 and args[0].isdigit():
    25	    num = int(args[0])
    26
    27	for i in range(0, num):
    28	    adjs = []
    29	    for j in range(0, 3):
    30	        adjs.append(random.choice(ADJECTIVES))
    31
    32	    print('You {} {}!'.format(', '.join(adjs), random.choice(NOUNS)))
$ ./insult.py foo
You bankrupt, cullionly, detestable milksop!
You foul, indistinguishable, false Satan!
You lascivious, scurilous, bankrupt villain!
You lascivious, lecherous, rotten jack!
You toad-spotted, base, foolish Satan!
$ ./insult.py 3
You detestable, cullionly, wall-eyed scold!
You peevish, caterwauling, caterwauling traitor!
You thin-faced, foul, dishonest Judas!
```

Notice how the program takes an optional argument that I expect to be an integer.  On line 24, I test both that there is an argument present and that it `isdigit()` before attempting to use it as a number.  The real work is done by the `random.choice` function to grab my adjectives and noun.

Lists could represent biological entities such as promotor, coding, and terminator regions.  Let's say we wanted to design synthetic microbes where we tested all possible permutations of these regions with each other to see if we were able to increase production of a desired enzyme.  Since the operation is N^3, I will only show the output for 2 genes:

```
$ ./recomb.py 2
N = "2"
  1: ('P1', 'C1', 'T1')
  2: ('P1', 'C1', 'T2')
  3: ('P1', 'C2', 'T1')
  4: ('P1', 'C2', 'T2')
  5: ('P2', 'C1', 'T1')
  6: ('P2', 'C1', 'T2')
  7: ('P2', 'C2', 'T1')
  8: ('P2', 'C2', 'T2')
```

Here is the Python code:

```
$ cat -n recomb.py
     1	#!/usr/bin/env python3
     2	"""Show recominations"""
     3
     4	import os
     5	import sys
     6	from itertools import product, chain
     7
     8	args = sys.argv[1:]
     9
    10	if len(args) != 1:
    11	    print('Usage: {} NUM_GENES'.format(os.path.basename(sys.argv[0])))
    12	    sys.exit(1)
    13
    14	if not args[0].isdigit():
    15	    print('"{}" does not look like an integer'.format(args[0]))
    16	    sys.exit(1)
    17
    18	num_genes = int(args[0])
    19	if not 2 <= num_genes <= 10:
    20	    print('NUM_GENES must be greater than 1, less than 10')
    21	    sys.exit(1)
    22
    23	promotors = []
    24	coding = []
    25	terminators = []
    26	for i in range(0, num_genes):
    27	    n = str(i + 1)
    28	    promotors.append('P' + n)
    29	    coding.append('C' + n)
    30	    terminators.append('T' + n)
    31
    32	print('N = "{}"'.format(num_genes))
    33	for i, combo in enumerate(chain(product(promotors, coding, terminators))):
    34	    print('{:3}: {}'.format(i + 1, combo))
```

The heavy lifting is being done on line 33 by the `product` function we get from the `itertools` module.  Because this function is given three lists to cross, it returns a list of three sub-lists which I want to combine into one list with `chain`.  Then I call the `enumerate` function \(shown in the first section\) to get the list number and the list in one loop so I don't have to keep up with a counter variable.

I don't like lines 26-30, so I tried rewriting using a list comprehension \(one of the most useful things you can do with lists\):

```
$ cat -n recomb2.py
     1	#!/usr/bin/env python3
     2	"""Show recominations"""
     3
     4	import os
     5	import sys
     6	from itertools import product, chain
     7
     8	args = sys.argv[1:]
     9
    10	if len(args) != 1:
    11	    print('Usage: {} NUM_GENES'.format(os.path.basename(sys.argv[0])))
    12	    sys.exit(1)
    13
    14	if not args[0].isdigit():
    15	    print('"{}" does not look like an integer'.format(args[0]))
    16	    sys.exit(1)
    17
    18	num_genes = int(args[0])
    19	if not 2 <= num_genes <= 10:
    20	    print('NUM_GENES must be greater than 1, less than 10')
    21	    sys.exit(1)
    22
    23	promotors = ['P' + str(n + 1) for n in range(0, num_genes)]
    24	coding = ['C' + str(n + 1) for n in range(0, num_genes)]
    25	terminators = ['T' + str(n + 1) for n in range(0, num_genes)]
    26
    27	print('N = "{}"'.format(num_genes))
    28	for i, combo in enumerate(chain(product(promotors, coding, terminators))):
    29	    print('{:3}: {}'.format(i + 1, combo))
```

But still, lines 23-25 are identical with the exception of the character I'm using, so I can put that code into a little function:

```
$ cat -n recomb3.py
     1	#!/usr/bin/env python3
     2	"""Show recominations"""
     3
     4	import os
     5	import sys
     6	from itertools import product, chain
     7
     8	args = sys.argv[1:]
     9
    10	if len(args) != 1:
    11	    print('Usage: {} NUM_GENES'.format(os.path.basename(sys.argv[0])))
    12	    sys.exit(1)
    13
    14	if not args[0].isdigit():
    15	    print('"{}" does not look like an integer'.format(args[0]))
    16	    sys.exit(1)
    17
    18	num_genes = int(args[0])
    19	if not 2 <= num_genes <= 10:
    20	    print('NUM_GENES must be greater than 1, less than 10')
    21	    sys.exit(1)
    22
    23	def gen(prefix):
    24	    return [prefix + str(n + 1) for n in range(0, num_genes)]
    25
    26	promotors = gen('P')
    27	coding = gen('C')
    28	terminators = gen('T')
    29
    30	print('N = "{}"'.format(num_genes))
    31	for i, combo in enumerate(chain(product(promotors, coding, terminators))):
    32	    print('{:3}: {}'.format(i + 1, combo))
```

Now all the repeated code is in the `gen` function \(line 23-24\), and I simply call that for each character I want. 



