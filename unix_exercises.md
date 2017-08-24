# Unix exercises

> Note: When you see a "$" given in the example prompts, it is a metacharacter indicating that this is the prompt for a normal \(not super-user\) account.  Your default prompt may be different, and it is highly configurable \(search for "PS1 unix prompt" to learn more\).  Anyway, point is that you should type \(copy/paste\) all the stuff _after_ the $.  If you ever see a prompt with "\#," it's indicating a command that should be run as the super-user/root account, e.g., installing some software into a system-wide directory so it can be shared by all users.

## Number of unique users

Find the number of unique users on a shared system

We know that "w" will tell us the users logged in.  Try it now on a system that has many users \(i.e., not your laptop\) and see the output.  We'll connect the output of "w" to "head" using a pipe "\|", but we only want the first five lines:

```
$ w | head -5
 14:36:01 up 21 days, 21:51, 176 users,  load average: 3.83, 4.31, 4.47
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
antontre pts/1    149.165.156.129  Sat14    3days  0.10s  0.10s /bin/sh -i
huddack  pts/3    128.4.131.189    09:38    4:56m  0.15s  0.15s -bash
```

We want to see the first five _users_, not the first five lines of output.  To skip the first two lines of headers from "w," we first pipe "w" into "awk" and tell it we only want to see output when the Number of Records \(NR\) is greater than 2:

```
$ w | awk 'NR>2' | head -5
antontre pts/1    149.165.156.129  Sat14    4days  0.10s  0.10s /bin/sh -i
huddack  pts/3    128.4.131.189    09:38    5:13m  0.15s  0.15s -bash
antontre pts/5    149.165.156.129  Sun19    2days  0.14s  0.14s /bin/sh -i
minyard  pts/8    129.114.64.18    29Jul16  4:24m  3:46m  3:46m top
antontre pts/11   149.165.156.129  Sun23    2days  0.24s  0.24s /bin/sh -i
```

Let's "cut" out just the first column of data.  The manpage for "cut" says that it defaults to using the tab character to determine columns, so we'll need to tell it to use spaces:

```
$ w | awk 'NR>2' | head -5 | cut -d ' ' -f 1
antontre
huddack
antontre
minyard
antontre
```

We can see right away that the some users like "antontre" are logged in multiple times, so let's "uniq" that output:

```
$ w | awk 'NR>2' | head -5 | cut -d ' ' -f 1 | uniq
antontre
huddack
antontre
minyard
antontre
```

Hmm, that's not right.  Remember I said earlier that "uniq" only works _on sorted input_?  So let's sort those names first:

```
$ w | awk 'NR>2' | head -5 | cut -d ' ' -f 1 | sort | uniq
antontre
huddack
minyard
```

OK, that is correct.  Now let's remove the "head -5" and use "wc" to count all the lines \(-l\) of input:

```
$ w | awk 'NR>2' | cut -d ' ' -f 1 | sort | uniq | wc -l
138
```

So what you see is that we're connecting small, well-defined programs together using pipes to connect the "standard input" \(STDIN\) and "standard output \(STDOUT\) streams.  There's a third basic file handle in Unix called "standard error" \(STDERR\) that we'll come across later.  It's a way for programs to report problems without simply dying.  You can redirect errors into a file like so:

```
$ program 2>err
$ program 1>out 2>err
```

The first example puts STDERR into a file called "err" and lets STDOUT print to the terminal.  The second example captures STDOUT into a file called "out" while STDERR goes to "err."

> Protip: Sometimes a program will complain about things that you cannot fix, e.g., "find" may complain about file permissions that you don't care about.  In those cases, you can redirect STDERR to a special filehandle called "/dev/null" where they are forgotten forever.  Kind of like the "memory hole" in 1984.

```
find / -name my-file.txt 2>/dev/null
```

## Count "oo" words

On almost every Unix system, you can find "/usr/share/dict/words."  Let's use "grep" to find how many have the "oo" vowel combination.  It's a long list, so I'll pipe it into "head" to see just the first five:

```
$ grep 'oo' /usr/share/dict/words | head -5
abloom
aboon
aboveproof
abrood
abrook
```

Yes, that works, so redirect those words into a file and count them:

```
$ grep 'oo' /usr/share/dict/words > oo-words
$ wc -l !$
10460 oo-words
```

Let's count them directly out of "grep":

```
$ grep 'oo' /usr/share/dict/words | wc -l
10460
```

How many of those words additionally contain the "ow" sequence?

```
$ grep 'oo' /usr/share/dict/words | grep 'ow' | wc -l
158
```

How many _do not_ contain the "ow" sequence?

```
$ grep 'oo' /usr/share/dict/words | grep -v 'ow' | wc -l
10302
```

Do those numbers add up?

```
$ bc <<< 158+10302
10460
```

Excellent.  Smithers, massage my brain.

## Something with sequences

Now we will get some sequence data from the iMicrobe FTP site.  Both "wget" and "ncftpget" will do the trick:

```
$ mkdir -p ~/work/abe487/contigs
$ cd !$
$ wget ftp://ftp.imicrobe.us/abe487/contigs/contigs.zip
```

> Protip: How do we know we got the correct data?  Go back and look at that FTP site, and you will see that there is a "contigs.zip.md5" file that we can "less" on the server to view the contents:

```
$ ncftp ftp://ftp.imicrobe.us/abe487/contigs
NcFTP 3.2.5 (Feb 02, 2011) by Mike Gleason (http://www.NcFTP.com/contact/).
Connecting to 128.196.131.100...
Welcome to the imicrobe.us repository
Logging in...
Login successful.
Logged in to ftp.imicrobe.us.
Current remote directory is /abe487/contigs.
ncftp /abe487/contigs > ls
contigs.zip        contigs.zip.md5
ncftp /abe487/contigs > less contigs.zip.md5

1b7e58177edea28e6441843ddc3a68ab  contigs.zip
ncftp /abe487/contigs > exit
```

> You can read up on MD5 \([https://en.wikipedia.org/wiki/Md5sum](https://en.wikipedia.org/wiki/Md5sum)\) to understand that this is a signature of the file.  If we calculate the MD5 of the file we dowloaded and it matches what we see on the server, then we can be sure that we have the exact file that is on the FTP site:

```
$ md5sum contigs.zip
1b7e58177edea28e6441843ddc3a68ab  contigs.zip
```

> Yes, those two sums match.  Note that sometimes the program is just called "md5."

So, back to the exercise.  Let's unpack the contigs:

```
$ unzip contigs.zip
Archive:  contigs.zip
  inflating: group12_contigs.fasta
  inflating: group20_contigs.fasta
  inflating: group24_contigs.fasta
$ rm contigs.zip
```

These files are in FASTA format \([https://en.wikipedia.org/wiki/FASTA\_format](https://en.wikipedia.org/wiki/FASTA_format)\), which basically looks like this:

```
>MCHU - Calmodulin - Human, rabbit, bovine, rat, and chicken
ADQLTEEQIAEFKEAFSLFDKDGDGTITTKELGTVMRSLGQNPTEAELQDMINEVDADGNGTID
FPEFLTMMARKMKDTDSEEEIREAFRVFDKDGNGYISAAELRHVMTNLGEKLTDEEVDEMIREA
DIDGDGQVNYEEFVQMMTAK*
>gi|5524211|gb|AAD44166.1| cytochrome b [Elephas maximus maximus]
LCLYTHIGRNIYYGSYLYSETWNTGIMLLLITMATAFMGYVLPWGQMSFWGATVITNLFSAIPYIGTNLV
EWIWGGFSVDKATLNRFFAFHFILPFTMVALAGVHLTFLHETGSNNPLGLTSDSDKIPFHPYYTIKDFLG
LLILILLLLLLALLSPDMLGDPDNHMPADPLNTPLHIKPEWYFLFAYAILRSVPNKLGGVLALFLSIVIL
GLMPFLHTSKHRSMMLRPLSQALFWTLTMDLLTLTWIGSQPVEYPYTIIGQMASILYFSIILAFLPIAGX
IENY
```

Header lines start with "&gt;", then the sequence follows.  Sequences may be broken up over several lines of 50 or 80 characters, but it's just as common to see the sequences take only one \(sometimes very long\) line.  Sequences may be nucleotides, proteins, very short DNA/RNA, longer contigs \(shorter strands assembled into contiguous regions\), or entire chromosomes or even genomes.

So, how many sequences are in "group12\_contigs.fasta"?  To answer, we just need to count how many times we see "&gt;".  We can do that with "grep":

```
$ grep > group12_contigs.fasta
Usage: grep [OPTION]... PATTERN [FILE]...
Try 'grep --help' for more information.
```

What is going on?  Remember when we captured the "oo" words that we used the "&gt;" symbol to tell Unix to _redirect_ the output of "grep" into a file.  We need to tell Unix that we mean a literal greater-than sign by placing it in single or double quotes or putting a backslash in front of it:

```
$ grep '>' group12_contigs.fasta
$ grep \> group12_contigs.fasta
```

You should actually see nothing because something quite insidious happened with that first "grep" statement -- it overwrote our original "group12\_contigs.fasta" with the result of "grep"ing for nothing, which is nothing:

```
$ ls -l group12_contigs.fasta
-rw-rw---- 1 kyclark staff 0 Aug 10 15:08 group12_contigs.fasta
```

Ugh, OK, I have to go back and "wget" the "contigs.zip" file to restore it.  That's OK.  Things like this happen all the time.

```
$ ls -lh group12_contigs.fasta
-rw-rw---- 1 kyclark staff 2.9M Aug 10 14:38 group12_contigs.fasta
```

Now that I have restored my data, I want to count how many greater-than signs in the file:

```
$ grep '>' group12_contigs.fasta | wc -l
132
```

Hey, I could see doing that often.  Maybe we should make this into an "alias" \(see above\).  The problem is that the "argument" to the function \(the filename\) is stuck in the middle of the chain of commands, so it would make it tricky to use an alias for this.  We can create a bash function that we add to our .bashrc:

```
function countseqs() {
  grep '>' $1 | wc -l
}
```

After you add this, remember to source this file to make it available:

```
$ source ~/.bashrc
$ countseqs group12_contigs.fasta
132
```

Same answer.  Good.  However, someone beat us to the punch.  There is a powerful tool called "seqmagick" \([https://github.com/fhcrc/seqmagick](https://github.com/fhcrc/seqmagick)\) that will do this \(and much, much more\).  It's installed into the "hurwitzlab/bin" directory, or you can install it locally:

```
$ seqmagick info group12_contigs.fasta
name                  alignment    min_len   max_len   avg_len  num_seqs
group12_contigs.fasta FALSE           5136    116409  22974.30       132
```

Run "seqmagick -h" to see everything it can do.

Moving on, let's find how many contig IDs in "group12\_contigs.fasta" contain the number "47":

```
$ grep 47 group12_contigs.fasta > group12_ids_with_47
[login3@~/work/sequences]$ cat !$
cat group12_ids_with_47
>Contig_247
>Contig_447
>Contig_476
>Contig_1947
>Contig_4764
>Contig_4767
>Contig_13471
```

Here we did a little "useless use of cat," but it's OK.  We also could have used "less" to view the file.  Here's another useless use of cat to copy a file:

```
$ cat group12_ids_with_47 > temp1_ids
```

Additionally, we want to copy the file again to make duplicates:

```
$ cp group12_ids_with_47 temp2_ids
```

How can we be sure these files are the same?  Let's use "diff":

```
$ diff temp1_ids temp2_ids
```

You should see nothing, which is a case of "no news is good news."  They don't differ in any way.  We can verify this with "md5sum":

```
$ md5sum temp*
957390ab4c31db9500d148854f542eee  temp1_ids
957390ab4c31db9500d148854f542eee  temp2_ids
```

They are the same file.  If there were even one character difference, they would generate different hashes.

Now we will create a file with duplicate IDs:

```
$ cat temp1_ids temp2_ids > duplicate_ids
```

Check contents of "duplicate\_ids" using "less" or "cat."  Now grab all of the contigs IDs from "group20\_contigs.fasta" that contain the number "51."  Concatenate the new IDs to the duplicate\_ids file in a file called "multiple\_ids":

```
$ cp duplicate_ids multiple_ids
$ grep 51 group20_contigs.fasta >> !$
grep 51 group20_contigs.fasta >> multiple_ids
```

Notice the "&gt;&gt;" arrows to indicate that we are _appending_ to the existing "multiple\_ids" file.

Remove the existing "temp" files using a "\*" wildcard:

```
$ rm temp*
```

Now let's explore more of what "sort" and "uniq" can do for us.  We want to find which IDs are unique and which are duplicated.  If we read the manpage \("man uniq"\), we see that there are "-d" and "-u" flags for doing just that.  However, we've already seen that input to "uniq" needs to be sorted, so we need to remember to do that:

```
$ sort multiple_ids | uniq -d > temp1_ids
$ sort multiple_ids | uniq -u > temp2_ids
$ diff temp*
1,7c1,11
< >Contig_13471
< >Contig_1947
< >Contig_247
< >Contig_447
< >Contig_476
< >Contig_4764
< >Contig_4767
---
> >Contig_10051
> >Contig_1651
> >Contig_4851
> >Contig_5141
> >Contig_5143
> >Contig_5164
> >Contig_5170
> >Contig_5188
> >Contig_6351
> >Contig_9651
> >Contig_9851
```

Let's remove our temp files again and make a "clean\_ids" file:

```
$ rm temp*
$ sort multiple_ids | uniq > clean_ids
```

We can use "sed" to alter the IDs.  The "s//" command say to "substitute" the first thing with the second thing, e.g., to replace all occurences of "foo" with "bar", use "s/foo/bar" \([http://stackoverflow.com/questions/4868904/what-is-the-origin-of-foo-and-bar](http://stackoverflow.com/questions/4868904/what-is-the-origin-of-foo-and-bar)\).

```
$ sed 's/C/c/' clean_ids
$ sed 's/_/./' clean_ids
$ sed 's/>//' clean_ids > newclean_ids
```

That last one removes the FASTA file artifact that identifies the beginning of an ID but is not part of the ID.  We can use this with "seqmagick" now to extract those sequences and find out how many were found:

```
$ seqmagick convert --include-from-file newclean_ids group12_contigs.fasta newgroup12_contigs.fasta
$ seqmagick info !$
seqmagick info newgroup12_contigs.fasta
name                     alignment    min_len   max_len   avg_len  num_seqs
newgroup12_contigs.fasta FALSE           5587     30751  16768.14         7
```

We can get stats on all our files:

```
$ seqmagick info *fasta > fasta-info
$ cat !$
name                     alignment    min_len   max_len   avg_len  num_seqs
group12_contigs.fasta    FALSE           5136    116409  22974.30       132
group20_contigs.fasta    FALSE           5029     22601   7624.38       203
group24_contigs.fasta    FALSE           5024     81329  12115.70       139
newgroup12_contigs.fasta FALSE           5587     30751  16768.14         7
```

We can use "cut" to view various columns:

```
$ cut -f 2 fasta-info
$ cut -f 2,4 fasta-info
$ cut -f 2-4 fasta-info
```

But it does not line up very nicely.  We can use "column" to fix this:

```
$ cut -f 2-4 fasta-info | column -t
alignment  min_len  max_len
FALSE      5136     116409
FALSE      5029     22601
FALSE      5024     81329
FALSE      5587     30751
```



