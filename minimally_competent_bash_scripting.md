# Minimally competent bash scripting

Bash is the worst shell scripting language except for all the others.  Much of the time, all you need is a simple bash script, so let's figure out how to write a decent one.  Though there are plenty of entire books you can read, I'll share with you what I've found to be the minimal amount I use.

# Shebang

Scripting languages (sh, bash, Perl, Python, Ruby, etc.) are generally distinguished by the fact that the "program" is a regular file containing plain text that is interpreted into machine code at the time you run it.  Other languages (c, C++, Haskell) have a separate compilation step to turn their regular text source files into binary executable.  If you "less" a compiled file, you'll 