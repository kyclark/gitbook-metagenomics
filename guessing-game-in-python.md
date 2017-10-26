# Guessing Game

I firmly believe that writing your own simple games is a terrific way to learn how to program.  Let's write a simple program where the user has to guess a random number.  To start, we'll use the "new\_py.py" script to stub out the boilerplate.  I'll use the `-a` flag to indicate that I want the program to use the `argparse` module so we can accept some named arguments to our script:

```
$ new_py.py -a guess
Done, see new script "guess.py."
$ cat -n guess.py
     1	#!/usr/bin/env python3
     2	"""docstring"""
     3
     4	import argparse
     5	import sys
     6
     7	# --------------------------------------------------
     8	def get_args():
     9	    """get args"""
    10	    parser = argparse.ArgumentParser(description='Argparse Python script')
    11	    parser.add_argument('positional', metavar='str', help='A positional argument')
    12	    parser.add_argument('-a', '--arg', help='A named string argument',
    13	                        metavar='str', type=str, default='')
    14	    parser.add_argument('-i', '--int', help='A named integer argument',
    15	                        metavar='int', type=int, default=0)
    16	    parser.add_argument('-f', '--flag', help='A boolean flag',
    17	                        action='store_true')
    18	    return parser.parse_args()
    19
    20	# --------------------------------------------------
    21	def main():
    22	    """main"""
    23	    args = get_args()
    24	    str_arg = args.arg
    25	    int_arg = args.int
    26	    flag_arg = args.flag
    27	    pos_arg = args.positional
    28
    29	    print('str_arg = "{}"'.format(str_arg))
    30	    print('int_arg = "{}"'.format(int_arg))
    31	    print('flag_arg = "{}"'.format(flag_arg))
    32	    print('positional = "{}"'.format(pos_arg))
    33
    34	# --------------------------------------------------
    35	if __name__ == '__main__':
    36	    main()
```

The template shows how to create named arguments for strings, integers, Booleans, as well as positional \(unnamed\) values.  For this program, we want to know the min/max numbers to guess and the number of guesses allowed.  We will provide reasonable defaults for all of them so that they will be completely optional.  Change your program to this:

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



