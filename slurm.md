# SLURM

SLURM's command for queue submission is ```sbatch```, and ```showq``` will show you your queue.  Compute nodes are shared by default.  You must request exclusive access if you need.  

* hpc-consult@list.arizona.edu is the help account

# TACC/Stampede

* TACC is part of the XSEDE (xsede.org) project.  
* TACC does not allow the use of job arrays on their clusters.  Instead, they have written their "parametric launcher" (https://www.tacc.utexas.edu/research-development/tacc-software/the-launcher).
* Your three important directories are ```$HOME```, ```$WORK```, and ```$SCRATCH```, and they can be accessed with ```cd```, ```cdw```, and ```cds```, respectively.  
* Compute nodes are not shared


# SLURM Hello

Here is our "hello" script modified from PBS to SLURM:

```
$ cat -n hello.sh
     1 	#!/bin/bash
     2
     3 	#SBATCH -A iPlant-Collabs
     4 	#SBATCH -p development # or "normal"
     5 	#SBATCH -t 01:00:00
     6 	#SBATCH -N 1
     7 	#SBATCH -n 1
     8 	#SBATCH -J hello
     9 	#SBATCH --mail-user=kyclark@email.arizona.edu
    10 	#SBATCH --mail-type=BEGIN,END,FAIL
    11
    12 	echo "Hello from sunny \"$(hostname)\"!"
```

As with the "#PBS" directives, we have "#SBATCH" to describe the job and resources.  Most important for TACC is the "-A" allocation argument that decides which account will be charge for the compute time.  For the "-p" partition, I can choose either "normal" or "development," the latter of which allows me a maximum of two hours.  The idea is that your job gets picked up relatively quickly, which makes it much faster to test new code.  The "-J" here is not "job array" (those are not allowed on stampede) but the job name, and I also threw in the options to email me when the job starts and stops.

The Makefile is pretty similar to before.  The command "make" will run "clean" and then "sbatch" for us.  The "qs" alias shows the job in "R" running state and the "CG" for "completing."  The standard error and output go into "slurm-[jobid]" files. 

```
$ cat -n Makefile
     1 	submit: clean
     2 		sbatch hello.sh
     3
     4 	clean:
     5 		find . -name slurm-\* -exec rm {} \;
$ make
find . -name slurm-\* -exec rm {} \;
sbatch hello.sh
-----------------------------------------------------------------
              Welcome to the Stampede Supercomputer
-----------------------------------------------------------------

No reservation for this job
--> Verifying valid submit host (login4)...OK
--> Verifying valid jobname...OK
--> Enforcing max jobs per user...OK
--> Verifying availability of your home dir (/home1/03137/kyclark)...OK
--> Verifying availability of your work dir (/work/03137/kyclark)...OK
--> Verifying valid ssh keys...OK
--> Verifying access to desired queue (development)...OK
--> Verifying job request is within current queue limits...OK
--> Checking available allocation (iPlant-Collabs)...OK
Submitted batch job 7560143
$ type qs
qs is aliased to `squeue -u kyclark | column -t'
$ qs
JOBID    PARTITION    NAME   USER     ST  TIME  NODES  NODELIST(REASON)
7560143  development  hello  kyclark  R   0:00  1      c557-904
$ qs
JOBID    PARTITION    NAME   USER     ST  TIME  NODES  NODELIST(REASON)
7560143  development  hello  kyclark  CG  0:08  1      c557-904
$ ls -l
total 16
-rw------- 1 kyclark G-814141  262 Aug 30 12:56 hello.sh
-rw------- 1 kyclark G-814141   77 Aug 30 13:01 Makefile
-rw------- 1 kyclark G-814141 1245 Aug 30 12:52 README.md
-rw------- 1 kyclark G-814141   54 Aug 30 13:09 slurm-7560143.out
[tacc:login4@/work/03137/kyclark/metagenomics-book/hpc/slurm/hello]$ cat slurm-7560143.out
Hello from sunny "c557-904.stampede.tacc.utexas.edu"!
```

