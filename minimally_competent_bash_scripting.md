# Minimally competent bash scripting

Bash is the worst shell scripting language except for all the others.  Much of the time, all you need is a simple bash script, so let's figure out how to write a decent one.  Though there are plenty of entire books you can read, I'll share with you what I've found to be the minimal amount I use.

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

So we just need to let the shell know that this is Python code, and that is what the "shebang" (see "Pronuciations") line is for:

