# Dictionaries

Python has a data type called a "dictionary" that allows you to associate some "key" \(a string\) to some "value" \(which can be anything such as a string, number, tuple, list, set, or another dictionary\).  The same data structure is also called a map, hash, and associative array.

You can define the define a dictionary with all the key/value pairs using the `{}` braces:

```
>>> patch = {'species': 'dog', 'age': 4}
>>> patch['species']
'dog'
>>> type(patch['species'])
<class 'str'>
>>> patch['age']
4
>>> type(patch['age'])
<class 'int'>
>>> patch['likes'] = ['walking', 'running', 'car trips', 'treats', 'pets']
>>> patch
{'species': 'dog', 'age': 4, 'likes': ['walking', 'running', 'car trips', 'treats', 'pets']}
>>> type(patch['likes'])
<class 'list'>
>>> 'Patch is {} and likes {}'.format(patch['age'], ', '.join(patch['likes']))
'Patch is 4 and likes walking, running, car trips, treats, pets'
```

Or you can use the `dict()` constructor and add key/value pairs:

```
>>> cat = dict()
>>> cat['name'] = 'Patrick the Catrick'
>>> cat
{'name': 'Patrick the Catrick'}
```

# Bridge of Death

Let's write a script to play with a dictionary:

```
$ cat -n bridge_of_death.py
     1	#!/usr/bin/env python3
     2
     3	person = {}
     4	print(person)
     5
     6	print('\n'.join(['Stop!', 'Who would cross the Bridge of Death',
     7	                 'must answer me these questions three,',
     8	                 'ere the other side he see.']))
     9
    10	for field in ['name', 'quest', 'favorite color']:
    11	    person[field] = input('What is your {}? '.format(field))
    12	    print(person)
    13
    14	if person['favorite color'].lower() == 'blue':
    15	    print('Right, off you go.')
    16	else:
    17	    print('You have been eaten by a grue.')
```

And here it is in action:

```
$ ./bridge_of_death.py
{}
Stop!
Who would cross the Bridge of Death
must answer me these questions three,
ere the other side he see.
What is your name? Sir Lancelot of Camelot
{'name': 'Sir Lancelot of Camelot'}
What is your quest? To seek the Holy Grail
{'name': 'Sir Lancelot of Camelot', 'quest': 'To seek the Holy Grail'}
What is your favorite color? Blue
{'name': 'Sir Lancelot of Camelot', 'quest': 'To seek the Holy Grail', 'favorite color': 'Blue'}
Right, off you go.
$ ./bridge_of_death.py
{}
Stop!
Who would cross the Bridge of Death
must answer me these questions three,
ere the other side he see.
What is your name? Sir Galahad of Camelot
{'name': 'Sir Galahad of Camelot'}
What is your quest? I seek the Holy Grail
{'name': 'Sir Galahad of Camelot', 'quest': 'I seek the Holy Grail'}
What is your favorite color? Blue. No yello--
{'name': 'Sir Galahad of Camelot', 'quest': 'I seek the Holy Grail', 'favorite color': 'Blue. No yello--'}
You have been eaten by a grue.
```

# Gashlycrumb

Dictionaries are perfect for looking up some bit of information by some value:

```
$ ./gashlycrumb.py c
C is for Clara who wasted away.
$ ./gashlycrumb.py t
T is for Titus who flew into bits.
$ cat -n gashlycrumb.py
     1	#!/usr/bin/env python3
     2	"""dictionary lookup"""
     3
     4	import os
     5	import sys
     6
     7	args = sys.argv[1:]
     8
     9	if len(args) != 1:
    10	    print('Usage: {} LETTER'.format(os.path.basename(sys.argv[0])))
    11	    sys.exit(1)
    12
    13	letter = args[0].upper()
    14
    15	text = """
    16	A is for Amy who fell down the stairs.
    17	B is for Basil assaulted by bears.
    18	C is for Clara who wasted away.
    19	D is for Desmond thrown out of a sleigh.
    20	E is for Ernest who choked on a peach.
    21	F is for Fanny sucked dry by a leech.
    22	G is for George smothered under a rug.
    23	H is for Hector done in by a thug.
    24	I is for Ida who drowned in a lake.
    25	J is for James who took lye by mistake.
    26	K is for Kate who was struck with an axe.
    27	L is for Leo who choked on some tacks.
    28	M is for Maud who was swept out to sea.
    29	N is for Neville who died of ennui.
    30	O is for Olive run through with an awl.
    31	P is for Prue trampled flat in a brawl.
    32	Q is for Quentin who sank on a mire.
    33	R is for Rhoda consumed by a fire.
    34	S is for Susan who perished of fits.
    35	T is for Titus who flew into bits.
    36	U is for Una who slipped down a drain.
    37	V is for Victor squashed under a train.
    38	W is for Winnie embedded in ice.
    39	X is for Xerxes devoured by mice.
    40	Y is for Yorick whose head was bashed in.
    41	Z is for Zillah who drank too much gin.
    42	"""
    43
    44	lookup = {}
    45	for line in text.splitlines():
    46	    if line:
    47	        lookup[line[0]] = line
    48
    49	if letter in lookup:
    50	    print(lookup[letter])
    51	else:
    52	    print('I do not know "{}"'.format(letter))
$ ./gashlycrumb.py
Usage: gashlycrumb.py LETTER
$ ./gashlycrumb.py a
A is for Amy who fell down the stairs.
$ ./gashlycrumb.py b
B is for Basil assaulted by bears.
$ ./gashlycrumb.py 8
I do not know "8"
```

On line 47, we create the `lookup` using the first character of the line \(`line[0]`\).  On line 49, we look to see if we have that letter in the `lookup`, printing the line of text if we do or complaining if we don't.

If we return to our previous chapter's DNA base counter, we can use dictionaries for this:

    $ cat -n dna3.py
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
        15	count = {}
        16
        17	for base in dna.lower():
        18	    if not base in count:
        19	        count[base] = 0
        20
        21	    count[base] += 1
        22
        23	counts = []
        24	for base in "acgt":
        25	    num = count[base] if base in count else 0
        26	    counts.append(str(num))
        27
        28	print(' '.join(counts))
    $ ./dna3.py `cat input.txt `
    20 12 17 21

But why?  Well, this has the great advantage of not having to declare four variables to count the four bases.  True, we're only checking \(in line 24\) for those four, but we can now count all the letters in any string.  

Notice that we create a new dict on line 15 with empty curlies `{}`.  In line 18, we have to check if the base exists in the dict; if it doesn't, we initialize it to 0, and then we increment it by one.  In line 25, we have to be careful when asking for a key that doesn't exist:

```
>>> cat['likes']
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'likes'
```

If we were counting a string of DNA like "AAAAAA," then there would be no C, G or T to report, so we have to use an `if/then` expression:

```
>>> seq = 'AAAAAA'
>>> counts = {}
>>> for base in seq:
...   if not base in counts:
...     counts[base] = 0
...   counts[base] += 1
...
>>> counts
{'A': 6}
>>> counts['G']
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'G'
>>> g = counts['G'] if 'G' in counts else 0
```

Or we can use the `get` method of a dictionary to safely get a value by a key even if the key doesn't exist:

```
>>> counts.get('G')
>>> type(counts.get('G'))
<class 'NoneType'>
```

If you look at "dna4.py," you'll see it's exactly the same as "dna3.py" with this exception:

```
    24	for base in "acgt":
    25	    num = count.get(base) or 0
    26	    counts.append(str(num))
```

The `get` method will not blow up your program:

```
>>> cat.get('likes')
>>> type(cat.get('likes'))
<class 'NoneType'>
>>> cat.get('likes') or 'Cats like nothing'
'Cats like nothing'
```

To get around the check, we could initialize the dict:

```
$ cat -n dna5.py
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
    15	count = {'a': 0, 'c': 0, 'g': 0, 't': 0}
    16
    17	for base in dna.lower():
    18	    if base in count:
    19	        count[base] += 1
    20
    21	counts = []
    22	for base in "acgt":
    23	    counts.append(str(count[base]))
    24
    25	print(' '.join(counts))
```

Now when we check on line 18, we're only going to count bases that we initialized; further, we can then just use the `keys` method to get the bases:

```
$ cat -n dna5.py
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
    15	count = {'a': 0, 'c': 0, 'g': 0, 't': 0}
    16
    17	for base in dna.lower():
    18	    if base in count:
    19	        count[base] += 1
    20
    21	counts = []
    22	for base in sorted(count.keys()):
    23	    counts.append(str(count[base]))
    24
    25	print(' '.join(counts))
```

This kind of checking and initializing is so common that there is a standard module to define a dictionary with a default value.  Unsurprisingly, it is called "defaultdict":

```
$ cat -n dna6.py
     1	#!/usr/bin/env python3
     2	"""Tetra-nucleotide counter"""
     3
     4	import sys
     5	import os
     6	from collections import defaultdict
     7
     8	args = sys.argv[1:]
     9
    10	if len(args) != 1:
    11	    print('Usage: {} DNA'.format(os.path.basename(sys.argv[0])))
    12	    sys.exit(1)
    13
    14	dna = args[0]
    15
    16	count = defaultdict(int)
    17
    18	for base in dna.lower():
    19	    count[base] += 1
    20
    21	counts = []
    22	for base in "acgt":
    23	    counts.append(str(count[base]))
    24
    25	print(' '.join(counts))
```

On line 16, we create a `defaultdict` with the `int` type \(not in quotes\) for which the default value will be zero:

```
>>> from collections import defaultdict
>>> counts = defaultdict(int)
>>> counts['a']
0
```

Finally, I will show you the `Counter` that will do all the base-counting for you, returning a `defaultdict`:

```
>>> from collections import Counter
>>> c = Counter('AACTAC')
>>> c['A']
3
>>> c['G']
0
```

And here is it in the script:

```
 $ cat -n dna7.py
     1	#!/usr/bin/env python3
     2	"""Tetra-nucleotide counter"""
     3
     4	import sys
     5	import os
     6	from collections import Counter
     7
     8	args = sys.argv[1:]
     9
    10	if len(args) != 1:
    11	    print('Usage: {} DNA'.format(os.path.basename(sys.argv[0])))
    12	    sys.exit(1)
    13
    14	dna = args[0]
    15
    16	count = Counter(dna.lower())
    17
    18	counts = []
    19	for base in "acgt":
    20	    counts.append(str(count[base]))
    21
    22	print(' '.join(counts))
```

So we can take that and create a program that counts all characters either from the command line or a file:

```
$ cat -n char_count1.py
     1	#!/usr/bin/env python3
     2	"""Character counter"""
     3
     4	import sys
     5	import os
     6	from collections import Counter
     7
     8	args = sys.argv
     9
    10	if len(args) != 2:
    11	    print('Usage: {} INPUT'.format(os.path.basename(args[0])))
    12	    sys.exit(1)
    13
    14	arg = args[1]
    15	text = ''
    16	if os.path.isfile(arg):
    17	    text = ''.join(open(arg).read().splitlines())
    18	else:
    19	    text = arg
    20
    21	count = Counter(text.lower())
    22
    23	for letter, num in count.items():
    24	    print('{} {:5}'.format(letter, num))
$ ./char_count1.py input.txt
a    20
g    17
c    12
t    21
```

# Methods

The `keys` from a dict are in no particular order:

```
>>> c = Counter('AAACTAGGGACTGA')
>>> c
Counter({'A': 6, 'G': 4, 'C': 2, 'T': 2})
>>> c.keys()
dict_keys(['A', 'C', 'T', 'G'])
```

If you want them sorted, you must be explicit:

```
>>> sorted(c.keys())
['A', 'C', 'G', 'T']
```

Note that, unlike a list, you cannot call `sort` which makes sense as that will try to sort a list in-place:

```
>>> c.keys().sort()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'dict_keys' object has no attribute 'sort'
```

You can also just call `values` to get those:

```
>>> c.values()
dict_values([6, 2, 2, 4])
```

Often you will want to go through the `items` in a dict and do something with the key and value:

```
>>> for base, count in c.items():
...   print('{} = {}'.format(base, count))
...
A = 6
C = 2
T = 2
G = 4
```

But if you want to have the `keys` in a particular order, you can do this:

```
>>> for base in sorted(c.keys()):
...   print('{} = {}'.format(base, c[base]))
...
A = 6
C = 2
G = 4
T = 2
```

Or you can notice that `items` returns a list of tuples:

```
>>> c.items()
dict_items([('A', 6), ('C', 2), ('T', 2), ('G', 4)])
```

And you can call `sorted` on that:

```
>>> sorted(c.items())
[('A', 6), ('C', 2), ('G', 4), ('T', 2)]
```

Which means this will work:

```
>>> for base, count in sorted(c.items()):
...   print('{} = {}'.format(base, count))
...
A = 6
C = 2
G = 4
T = 2
```

Note that `sorted` will sort by the first elements of all the tuples, then by the second, and so forth:

```
>>> genes = [('Indy', 4), ('Boss', 2), ('Lush', 10), ('Boss', 4), ('Lush', 1)]
>>> sorted(genes)
[('Boss', 2), ('Boss', 4), ('Indy', 4), ('Lush', 1), ('Lush', 10)]
```

If we want to sort the bases instead by their frequency, we have to use some trickery like a list comprehension to first reverse the tuples:

```
>>> [(x[1], x[0]) for x in c.items()]
[(6, 'A'), (2, 'C'), (2, 'T'), (4, 'G')]
>>> sorted([(x[1], x[0]) for x in c.items()])
[(2, 'C'), (2, 'T'), (4, 'G'), (6, 'A')]
```

But what is particularly nifty about Counters is that they have built-in methods to help you with such actions:

```
>>> c.most_common(2)
[('A', 6), ('G', 4)]
>>> c.most_common()
[('A', 6), ('G', 4), ('C', 2), ('T', 2)]
```

You should read the documentation to learn more \(https://docs.python.org/3/library/collections.html\).

# Character Counter with the works

Finally, I'll show you a version of the character counter that takes some other arguments to control how to show the results:

```
$ cat -n char_count2.py
     1	#!/usr/bin/env python3
     2	"""Character counter"""
     3
     4	import argparse
     5	import os
     6	import sys
     7	from collections import Counter
     8
     9	# --------------------------------------------------
    10	def get_args():
    11	    """get args"""
    12	    parser = argparse.ArgumentParser(description='Argparse Python script')
    13	    parser.add_argument('arg', help='File/string to count', type=str)
    14	    parser.add_argument('-c', '--charsort', help='Sort by character',
    15	                        dest='charsort', action='store_true')
    16	    parser.add_argument('-n', '--numsort', help='Sort by number',
    17	                        dest='numsort', action='store_true')
    18	    parser.add_argument('-r', '--reverse', help='Sort in reverse order',
    19	                        dest='reverse', action='store_true')
    20	    return parser.parse_args()
    21
    22	# --------------------------------------------------
    23	def main():
    24	    """main"""
    25	    args = get_args()
    26	    arg = args.arg
    27	    charsort = args.charsort
    28	    numsort = args.numsort
    29	    revsort = args.reverse
    30
    31	    if charsort and numsort:
    32	        print('Please choose one of --charsort or --numsort')
    33	        sys.exit(1)
    34
    35	    if not charsort and not numsort:
    36	        charsort = True
    37
    38	    text = ''
    39	    if os.path.isfile(arg):
    40	        text = ''.join(open(arg).read().splitlines())
    41	    else:
    42	        text = arg
    43
    44	    count = Counter(text.lower())
    45
    46	    if charsort:
    47	        letters = sorted(count.keys())
    48	        if revsort:
    49	            letters.reverse()
    50
    51	        for letter in letters:
    52	            print('{} {:5}'.format(letter, count[letter]))
    53	    else:
    54	        pairs = sorted([(x[1], x[0]) for x in count.items()])
    55	        if revsort:
    56	            pairs.reverse()
    57
    58	        for n, char in pairs:
    59	            print('{} {:5}'.format(char, n))
    60
    61	# --------------------------------------------------
    62	if __name__ == '__main__':
    63	    main()
```

# Acronym Finder

Similar to the `gashlycrumb.py` program that looked up a line of text for a given letter, we could randomly create meanings for a given acronym:

```
$ ./bacryonym.py NSF
NSF =
 - Nonrepresentationalism Staunchness Forever
 - Naturing Significantly Fontal
 - Nonclinical Solecistical Folkmoter
 - Nonhumanist Scaledrake Fellani
 - Naumk Sulpha Fause
$ ./bacryonym.py FBI
FBI =
 - Folksiness Boxmaker Interviewer
 - Flavorless Bumbler Incorruption
 - Flusterate Bakuninism Isopilocarpine
 - Freshen Bondsman Indigene
 - Fluotantalate Bornyl Interligamentous
```

That is just using the standard dictionary to look up words, so we could make it more interesting by using the works of Shakespeare:

```
$ ./bacryonym.py -w shakespeare.txt FBI
FBI =
 - Furthermore Burnet Instigation
 - Favor Bursting Insisting
 - Flower Beart Immanity
 - Fearfully Borne Itmy
 - Fooleries Blunts Intoxicates
```

Here is the Python for that:

```
$ cat -n bacryonym.py
     1	#!/usr/bin/env python3
     2	"""Make guesses about acronyms"""
     3
     4	import argparse
     5	import sys
     6	import os
     7	import random
     8	import re
     9	from collections import defaultdict
    10
    11	# --------------------------------------------------
    12	def main():
    13	    """main"""
    14	    args = get_args()
    15	    acronym = args.acronym
    16	    wordlist = args.wordlist
    17	    limit = args.num
    18	    goodword = r'^[a-z]{2,}$'
    19	    badwords = set(re.split(r'\s*,\s*', args.exclude.lower()))
    20
    21	    if not re.match(goodword, acronym.lower()):
    22	        print('"{}" must be >1 in length, only use letters'.format(acronym))
    23	        sys.exit(1)
    24
    25	    if not os.path.isfile(wordlist):
    26	        print('"{}" is not a file.'.format(wordlist))
    27	        sys.exit(1)
    28
    29	    seen = {}
    30	    words_by_letter = defaultdict(list)
    31	    for word in open(wordlist).read().lower().split():
    32	        clean = re.sub('[^a-z]', '', word)
    33	        if re.match(goodword, clean) and clean not in seen and clean not in badwords:
    34	            seen[clean] = 1
    35	            words_by_letter[clean[0]].append(clean)
    36
    37	    len_acronym = len(acronym)
    38	    definitions = []
    39	    for i in range(0, limit):
    40	        definition = []
    41	        for letter in acronym.lower():
    42	            possible = words_by_letter[letter]
    43	            if len(possible) > 0:
    44	                definition.append(random.choice(possible).title())
    45
    46	        if len(definition) == len_acronym:
    47	            definitions.append(' '.join(definition))
    48
    49	    if len(definitions) > 0:
    50	        print(acronym.upper() + ' =')
    51	        for definition in definitions:
    52	            print(' - ' + definition)
    53	    else:
    54	        print('Sorry I could not find any good definitions')
    55
    56	# --------------------------------------------------
    57	def get_args():
    58	    """get arguments"""
    59	    parser = argparse.ArgumentParser(description='Explain acronyms')
    60	    parser.add_argument('acronym', help='Acronym', type=str, metavar='STR')
    61	    parser.add_argument('-n', '--num', help='Maximum number of definitions',
    62	                        type=int, metavar='NUM', default=5)
    63	    parser.add_argument('-w', '--wordlist', help='Dictionary/word file',
    64	                        type=str, metavar='STR',
    65	                        default='/usr/share/dict/words')
    66	    parser.add_argument('-x', '--exclude', help='List of words to exclude',
    67	                        type=str, metavar='STR', default='a,an,the')
    68	    return parser.parse_args()
    69
    70	# --------------------------------------------------
    71	if __name__ == '__main__':
    72	    main()
```



