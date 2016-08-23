# HPC

HPC is an acronym for "high-performance computing," and it generally means using a cluster of computers.  Our students have access to two clusters at the University of Arizona, and most anyone is welcome to use the clusters at TACC.  To use the cluster, it's necessary to submit a batch job along with a description of the resources you need (e.g., memory, number of CPUs, number of nodes) to a scheduler that will start your job when the resources become available.  We will discuss schedulers "PBS" used at UA and "SLURM" used at TACC.  In the Github repo, you will find an "hpc" directory that contains examples for submitting to each queue.

To interact PBS and SLURM, you must log in to the "head" node(s).  Often you will be placed on a random nodes such as "login1."  YOU ARE NOT ALLOWED TO DO HEAVY LIFTING ON THE HEAD NODE.  For our class, you can write files, interact with the Perl RELP, run small scripts, etc., but you should never run BLAST or launch long-running jobs on these machines.  They are intended to be used to submit jobs to the queue.

# PBS

The PBS command for submitting to the queue is ```qsub```.  Since this command takes many arguments, I usually write a small script to gather all the arguments and execute the command so it's documented how I ran the job.  Most of the time I call this "submit.sh" it basically does ```qsub $ARGS run.sh```.  To view your queue, use ```qstat -u $USER```.  Use ```va``` to view your allocation of compute hours.  The UA has three queues: high-priority, normal, and windfall.  If you exhaust your normal hours in a month, then your jobs must run under "windfall" (catch as catch can) until your hours are replenished.

# SLURM

SLURM's command for queue submission is ```sbatch```, and ```showq``` will show you your queue.  Compute nodes are shared by default.  You must request exclusive access if you need.  

* hpc-consult@list.arizona.edu is the help account

# TACC/Stampede

* TACC is part of the XSEDE (xsede.org) project.  
* TACC does not allow the use of job arrays on their clusters.  Instead, they have written their "parametric launcher" (https://www.tacc.utexas.edu/research-development/tacc-software/the-launcher).
* Your three important directories are ```$HOME```, ```$WORK```, and ```$SCRATCH```, and they can be accessed with ```cd```, ```cdw```, and ```cds```, respectively.  
* Compute nodes are not shared

# Aliases

To make it easier to go back and forth between PBS and SLURM, I create aliases so that I can execute the same command on both systems:

## UA HPC/PBS

```
alias qstat="/usr/local/bin/qstat_local"
ME="kyclark"
alias qs="qstat -u $ME"
alias qt="qstat -Jtu $ME"
function qkill() {
  if [[ "${#1}" -eq 0 ]]; then
    echo {Now I crush you!"
    OUT=$(qstat -u $ME | grep $ME | cut -f 1 -d ' ' | sed 's/\[\]\..*/[]/' | xargs qdel)

    if [[ $? -eq 0 ]]; then
        echo "Jobs killed"
    else
        echo -e "\nError submitting job\n$OUT\n"
    fi
  else
    echo Argument = \"$1\"
    echo "This isn't the command you're looking for. I don't take arguments"
  fi
}

function qr() {
  WHO=${1:-$ME}
  echo qstat for \"$WHO\"
  OUT=`qstat -Jtu $WHO | tail -n +6 | awk '{print $10}' | sort | uniq -c`
  if [ -n "$OUT" ]; then
      echo "$OUT"
  else
      echo No jobs currently running.
  fi
}
```

## Stampede/SLURM

```
ME="kyclark"
alias qs='squeue -u $ME | column -t'
```
