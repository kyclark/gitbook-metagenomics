# Counting Things

Given a file like this:

```
$ cat -n names.txt
     1 	cat
     2 	dog
     3 	mouse
     4 	bird
     5 	cat
     6 	cat
     7 	dog
```

We can count the number of times each animal occurs with command-line tools:

```
$ sort names.txt | uniq -c
   1 bird
   3 cat
   2 dog
   1 mouse
```

If we wanted the output sorted numerically up or down:

```
$ sort names.txt | uniq -c | sort -n
   1 bird
   1 mouse
   2 dog
   3 cat
$ sort names.txt | uniq -c | sort -nr
   3 cat
   2 dog
   1 mouse
   1 bird
```

If we wanted them sorted instead on the animals:

```
$ sort names.txt | uniq -c | awk 'OFS="\t" { print $2,$1 }' | sort
bird   	1
cat    	3
dog    	2
mouse  	1
$ sort names.txt | uniq -c | awk 'OFS="\t" { print $2,$1 }' | sort -r
mouse  	1
dog    	2
cat    	3
bird   	1
```

Obviously this is getting a bit complicated, and it will get worse as we need to handle ugly data like "Mouse/MOUSE/mice/muose," etc.  