# Minimally competent bash scripting

Bash is the worst shell scripting language except for all the others.  Much of the time, all you need is a simple bash script, so let's figure out how to write a decent one.  Though there are plenty of entire books you can read, I'll share with you what I've found to be the minimal amount I use.

# Comments
Every language has a way to indicate non-executable text in the source code.  Many Unix/c-style languages use the "#" (hash) sign to indicate that any text to the right should be ignored by the language, but some languages use other characters or character combinations.  Programmers may actually "comment" on their code to explain what some particularly hairy bit is doing, or they may use the characters to temporarily disable some section of code.  Here is what you might see:

```
# cf. https://en.wikipedia.org/wiki/Factorial
sub fac(n) { 
  # first check terminal condition
  if (n <= 1) {
    return 1
  }
  # no? let's recurse!
  else {
    n * fac(n - 1) # the number times one less the number
  }
}
```

It's worth investing time in an editor that can easily comment/uncomment whole sections of code.  For instance, in vim, I have the "#" sign mapped to a function that will add or removed the appropriate comment character(s) from the beginning of the selected section.  If your editor can't do that (e.g., nano), then I suggest you find something more powerful.

# Shebang

Scripting languages (sh, bash, Perl, Python, Ruby, etc.) are generally distinguished by the fact that the "program" is a regular file containing plain text that is interpreted into machine code at the time you run it.  Other languages (c, C++, Haskell) have a separate compilation step to turn their regular text source files into binary executable.  If you "less" a compiled file, you'll see a mess that might even lock up your window.  (If that happens, refer back to "Make it stop!" to kill it or just close the window and start over.)

So, basically a "script" is a plain text file that is often executable by virtue of having the executable bit(s) turned on (cf. "Permissions").  It does not have to be executable, however.  It's acceptable to put some commands in a file and simply tell the appropriate program to interpret the file as a series of commands:

```
$ echo "echo Hello, World" > hello.sh
$ sh hello.sh
Hello, World
```

But it looks cooler to do this:

```
$ chmod +x hello.sh
$ ./hello.sh
Hello, World
```

But what's going on here?

```
$ echo 'print("Hello, World")' > hello.py
$ chmod +x hello.py
$ ./hello.py
./hello.py: line 1: syntax error near unexpected token `"Hello, World"'
./hello.py: line 1: `print("Hello, World")'
```

We put some Python code in a file and then asked our shell (which is bash) to interpret it.  That didn't work.  If we ask Python to run it, everything is fine:

```
$ python hello.py
Hello, World
```

So we just need to let the shell know that this is Python code, and that is what the "shebang" (see "Pronuciations") line is for.  It looks like a comment, but it's special line that the shell looks for to determine how to interpret the script.  I'll use an editor to add this to the "hello.py" script, then I'll ```cat``` the file so you can see what it looks like.  

```
$ cat hello.py
#!/usr/bin/env python
print("Hello, World")
$ ./hello.py
Hello, World
```

Often the shebang line will indicate the absolute path to a program like "/bin/bash" or "/usr/local/bin/gawk," but here I used an absolute path not to Python but to the "env" program which I then passed "python" as the argument.  Why did I do that?  To make this script "poratable" (for certain values of "portable", cf. "It's easier to port a shell than a shell script." – Larry Wall), I prefer to use the "python" that is found by the environment.  There are two major versions of Python (2.x and 3.x), and I can't actually be sure where the user has them installed and which is their default version.  In fact, just from my laptop to the two HPC systems I use, my "python" can be:

* /work/03137/kyclark/local/bin/python
* /rsgrps/bhurwitz/hurwitzlab/bin/python
* /Users/kyclark/anaconda3/bin/python

Go to your command line and type and type ```env python```, and you should be dropped into the Python REPL (Read-Evaluate-Print-Loop):

```
$ env python
Python 2.7.11 (default, Apr 22 2016, 18:12:30)
[GCC 4.4.7 20120313 (Red Hat 4.4.7-16)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> print("Hi there.")
Hi there.
>>> exit
Use exit() or Ctrl-D (i.e. EOF) to exit
```
