# PBS

The University of Arizona's HPC cluster uses the "PBS Pro" (portable batch system) scheduler.

Here are some important links:

* hpc-consult@list.arizona.edu (email list for help)
* https://confluence.arizona.edu/display/UAHPC/HPC+Documentation
* http://rc.arizona.edu/hpc-htc/high-performance-computing-high-throughput-computing
* http://rc.arizona.edu/hpc-htc/using-systems/pbs-example


# Allocations

Your "allocation" is how much compute time you are allowed on the cluster.  Use the command ```va``` to view your allocation of compute hours, e.g.:

```
$ va
kyclark current allocation (remaining/encumbered/total):
---------------------------------------------
Group           	standard             	qualified
bhurwitz        	17215:23/00:00/108000:00	99310:56/72:00/100000:00
bh_admin        	00:00/00:00/00:00	00:00/00:00/00:00
bh_dev          	00:00/00:00/00:00	00:00/00:00/00:00
gwatts          	12000:00/00:00/24000:00	00:00/00:00/00:00
mbsulli         	228000:00/00:00/228000:00	00:00/00:00/00:00
```

The UA has three queues: high-priority, normal, and windfall. If you exhaust your normal hours in a month, then your jobs must run under "windfall" (catch as catch can) until your hours are replenished.

# Job submission

The PBS command for submitting to the queue is ```qsub```. Since this command takes many arguments, I usually write a small script to gather all the arguments and execute the command so it's documented how I ran the job. Most of the time I call this "submit.sh" it basically does ```qsub $ARGS run.sh```. To view your queue, use ```qstat -u $USER```. 


# Hello

Here is a "hello" script:

```
$ cat -n hello.sh
     1 	#!/bin/bash
     2
     3 	#PBS -W group_list=bhurwitz
     4 	#PBS -q standard
     5 	#PBS -l jobtype=cluster_only
     6 	#PBS -l select=1:ncpus=1:mem=1gb
     7 	#PBS -l walltime=01:00:00
     8 	#PBS -l cput=01:00:00
     9
    10 	echo "Hello from sunny \"$(hostname)\"!"
```

The ```#PBS``` lines almost look like comments, but they are directives to PBS to describe your job.  Lines 6-7 says that we require a very small machine with just one CPU and 1G of memory and that we only want it for 1 hour.  The less you request, the more likely you are to get a machine meeting (or exceeding) your needs.  On line 10, we are including the ```hostname``` of the compute node so that we can see that, though we submit the job from a head node (e.g., "login1"), the job is run on a different machine.

Here is a Makefile to submit it:

```
$ cat -n Makefile
     1 	submit: clean
     2 		qsub hello.sh
     3
     4 	clean:
     5 		find . -name hello.sh.[eo]* -exec rm {} \;  
```

Just typing ```make``` will run the "clean" command to remove any previous out/error files, and this it will ```qsub``` our "hello.sh" script:

```
$ make
find . -name hello.sh.[eo]* -exec rm {} \;
qsub hello.sh
818089.service0
$ type qs
qs is aliased to `qstat -u kyclark'
$ qs

service0:
                                                            Req'd  Req'd   Elap
Job ID          Username Queue    Jobname    SessID NDS TSK Memory Time  S Time
--------------- -------- -------- ---------- ------ --- --- ------ ----- - -----
818089.service0 kyclark  clu_stan hello.sh      --    1   1    1gb 01:00 Q   --
```

Until the job is picked up, the "S" (status) column will show "Q" for "queued," then it will change to "R" for "running," "E" for "error," or "X" for "exited."   When ```qstat``` returns nothing, then the job has finished.  You should see files like "hello.sh.o[jobid]" for the output and "hello.sh.e[jobid]" for the errors (which we hope are none):

```
$ ls -lh
total 128K
-rw-rw-r-- 1 kyclark staff     210 Aug 30 10:09 hello.sh
-rw------- 1 kyclark bhurwitz    0 Aug 30 10:19 hello.sh.e818089
-rw------- 1 kyclark bhurwitz  181 Aug 30 10:19 hello.sh.o818089
-rw-rw-r-- 1 kyclark staff      81 Aug 30 10:08 Makefile
-rw-rw-r-- 1 kyclark staff    1.3K Aug 26 08:12 README.md
$ cat hello.sh.o818089
Hello from sunny "r1i3n10"!
Your group bhurwitz has been charged 00:00:01 for 1 cpus.
You previously had 42575:01:07.  You now have 42575:01:06 remaining for the queue clu_standard
```

# FTP

While Makefiles can be a great way to document for myself (and others) how I submitted and ran a job, I will often write a "submit.sh" script to check input, decide on resources, etc.  Here is a more complicated submission for retrieving data from an FTP server:

```
$ cat -n submit.sh
     1 	#!/bin/bash
     2
     3 	set -u
     4
     5 	export FTP_LIST=get-me
     6 	export OUT_DIR=$PWD
     7 	export NCFTPGET=/rsgrps/bhurwitz/hurwitzlab/bin/ncftpget
     8 	export PBSDIR=pbs
     9
    10 	if [[ -d $PBSDIR ]]; then
    11 	  rm -rf $DIR/*
    12 	else
    13 	  mkdir $DIR
    14 	fi
    15
    16 	NUM_FILES=$(wc -l $FTP_LIST | cut -d ' ' -f 1)
    17
    18 	if [[ $NUM_FILES -gt 0 ]]; then
    19 	  JOB_ID=$(qsub -N ftp -v OUT_DIR,FTP_LIST,NCFTPGET -j oe -o $PBSDIR ftp-get.sh)
    20 	  echo "Submitted \"$FILE\" files to job \"$JOB_ID\""
    21 	else
    22 	  echo "Can't find any files in \"$FTP_LIST\""
    23 	fi
$ cat get-me
ftp://ftp.imicrobe.us/projects/33/CAM_PROJ_HumanGut.asm.fa.gz
ftp://ftp.imicrobe.us/projects/121/CAM_P_0001134.csv.gz
ftp://ftp.imicrobe.us/projects/66/CAM_PROJ_TwinStudy.csv.gz
```

Here I have a file "get-me" with a few files on an FTP server that I want to download using the program "ncftpget" (http://ncftp.com/) which is installed in our shared "bin" directory.  Since I don't like having the output files from PBS scattered about my working directory, I like to make a place ("pbs") to put them (lines 8-14), and then I include the "-j oe" flag to "join output/error" files together and "-o" to put the output files in $PBSDIR.  On line 16, I check that there is legitimate input from the user.   Line 19 captures the output from the ```qsub``` command to report on the submission.  

One way to pass arguments to the compute node is by ```export```ing variables (lines 5-8) and then using the "-v" option to send those parts of the environment with the job.  If you ever get an error on ```qsub``` that say it can't send the environment, it's because you failed to ```export``` the variable.

Here is the script that actually downloads the files:

```
$ cat -n ftp-get.sh
     1 	#!/bin/bash
     2
     3 	#PBS -W group_list=bhurwitz
     4 	#PBS -q standard
     5 	#PBS -l jobtype=serial
     6 	#PBS -l select=1:ncpus=2:mem=4gb
     7 	#PBS -l place=pack:shared
     8 	#PBS -l walltime=24:00:00
     9 	#PBS -l cput=24:00:00
    10
    11 	set -u
    12
    13 	cd $OUT_DIR
    14
    15 	echo "Started $(date)"
    16
    17 	i=0
    18 	while read FTP; do
    19 	  let i++
    20 	  printf "%3d: %s\n" $i $FTP
    21 	  $NCFTPGET $FTP
    22 	done < $FTP_LIST
    23
    24 	echo "Ended $(date)"
 ```
 
All of the "#PBS" directives in this script could also have been specified as options to the ```qsub``` command in the submit script.  Even though I have "set -u" on and have not declared ```$OUT_DIR```, I can ```cd``` to it because it was exported from the submit script.  When your job is placed on the compute node, it will be placed into your $HOME directory, so it's important to have your job place its output files into the correct location.  The rest of the script is fairly self-explanatory, reading the ```$FTP_LIST``` one line at a time, using "ncftpget" to fetch it ("wget" would work just fine, too).

# Job Arrays

Downloading files doesn't usually take a long time, but for our purposes let's pretend each file would take upwards of 10 hours.  We are only allowed 24 hours on a compute node, so we think we can fetch at most two files for each job.  If we have 200 files, then we need 100 jobs which exceeds the polite and allowed number of jobs we can put into queue at any one time.  This is when we would use a job array to submit just one job that will be turned into the required 100 jobs to handle the 200 files:

```
$ cat -n submit.sh
     1 	#!/bin/bash
     2
     3 	set -u
     4
     5 	export FTP_LIST=get-me
     6 	export OUT_DIR=$PWD
     7 	export NCFTPGET=/rsgrps/bhurwitz/hurwitzlab/bin/ncftpget
     8 	export PBSDIR=pbs
     9 	export STEP_SIZE=2
    10
    11 	if [[ -d $PBSDIR ]]; then
    12 	  rm -rf $DIR/*
    13 	else
    14 	  mkdir $DIR;
    15 	fi
    16
    17 	NUM_FILES=$(wc -l $FTP_LIST | cut -d ' ' -f 1)
    18
    19 	if [[ $NUM_FILES -lt 1 ]]; then
    20 	  echo "Can\'t find any files in \"$FTP_LIST\""
    21 	  exit 1
    22 	fi
    23
    24 	JOBS=""
    25 	if [[ $NUM_FILES -gt 1 ]]; then
    26 	  JOBS="-J $NUM_FILES"
    27 	  if [[ $STEP_SIZE -gt 1 ]]; then
    28 	    JOBS="$JOBS:$STEP_SIZE"
    29 	  fi
    30 	fi
    31
    32 	JOB_ID=$(qsub $JOBS -N ftp -v OUT_DIR,STEP_SIZE,FTP_LIST,NCFTPGET -j oe -o $PBSDIR ftp-get.sh)
    33
    34 	echo "Submitted \"$FILE\" files to job \"$JOB_ID\""
```

The only difference from the previous version is that we have a new ```$STEP_SIZE``` variable set to "2" meaning we want to handle 2 jobs per node.  Lines 24-30 build up the a string to describe the job array which will only be needed if there is more than 1 job.  

The FTP downloading now needs to take into account which files to download from the ```$FTP_LIST```

```
$ cat -n ftp-get.sh
     1 	#!/bin/bash
     2
     3 	#PBS -W group_list=bhurwitz
     4 	#PBS -q standard
     5 	#PBS -l jobtype=serial
     6 	#PBS -l select=1:ncpus=1:mem=1gb
     7 	#PBS -l place=pack:shared
     8 	#PBS -l walltime=24:00:00
     9 	#PBS -l cput=24:00:00
    10
    11 	set -u
    12
    13 	echo "Started $(date)"
    14
    15 	cd $OUT_DIR
    16
    17 	TMP_FILES=$(mktemp)
    18 	sed -n "${PBS_ARRAY_INDEX:-1},${STEP_SIZE:-1}" $FTP_FILES > $TMP_FILES
    19 	NUM_FILES=$(wc -l $TMP_FILES | cut -d ' ' -f 1)
    20
    21 	if [[ $NUM_FILES -lt 1 ]]; then
    22 	  echo "Failed to fetch files"
    23 	  exit 1
    24 	fi
    25
    26 	echo "Will fetch $NUM_FILES"
    27
    28 	i=0
    29 	while read FTP; do
    30 	  let i++
    31 	  printf "%3d: %s\n" $i $FTP
    32 	  $NCFTPGET $FTP
    33 	done < $TMP_FILES
    34
    35 	rm $TMP_FILES
    36
    37 	echo "Ended $(date)"
```

To extract the files for the given compute node, we use the ```$PBS_ARRAY_INDEX``` variable created by PBS along with the ```$STEP_SIZE``` variable as arguments to a ```sed``` command, redirecting that output into a temporary file.  From there, the script proceeds as before only reading from the ```$TMP_FILES``` and removing it when the job is done.

# Interactive job

You can use ```qsub -I``` flag to be placed onto a compute node to run your job interactively.  This is a good way to debug your script in the actual runtime environment.  TACC has a nifty alias called ```idev``` that will fire up an interactive node for you to play with, so here is a PBS version to do the same.  Place this line in your "~/.bashrc" (be sure to "source" the file afterwards):

```
alias idev="qsub -I -N idev -W group_list=bhurwitz -q standard -l walltime=01:00:00 -l select=1:ncpus=1:mem=1gb"
```

Then from a login node (here "service2") I can type ```idev``` to get a compute node.  When I'm finished, I can CTRL-D or type "exit" or "logout" to go back to the login node:

```
$ hostname
service2
$ idev
qsub: waiting for job 652560.service2 to start
qsub: job 652560.service2 ready

$ hostname
htc50
$ logout

qsub: job 652560.service2 completed
```

# Dependency Chain

To schedule a job with a dependency chain, it's necessary to be on the correct login/head node for the type of job:

* service0 - cluster scheduler
* service1 - smp scheduler
* service2 - htc scheduler