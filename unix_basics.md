# Unix basics

Given that our class will have students on a variety of operating systems (Windows, OSX, Linux), we will use the HPC (high performance computing) cluster at the University of Arizona for our work.  Students Windows might find the Cygwin tools useful for getting access to tools like "ssh" and "scp."  Students on OSX and Linux machines may choose to install software locally using package management tools like "homebrew" (OSX) or "apt-get" and "yum" (Linux).

## University of Arizona HPC

We need to get into a Unix system.  If you're on Linux or OSX, open a terminal and type:

```
ssh <NetID>@hpc.arizona.edu
```

Protocol about setting up SSH keys to get around two-factor authentication.

If all goes well, you should be on a machine called "gatekeeper.hpc.arizona.edu."  Verfiy this with the command "hostname<Enter>".  Use the command "menuon" to get the following menu:

```
===============
HPC.ARIZONA.EDU
===============
Please select a target system to connect to:

(1) Ocelote
(2) El Gato
(3) Cluster/HTC/SMP
(Q) Quit
(D) Disable menu
```

The menu will be present on your next login session.  Make a selection from the menu to login to a head node.

## Stampede (TACC)

The "stampede" cluster is another HPC resource that is freely available to our students.  It is located at TACC, the Texas Advanced Computing Center at the University of Texas in Austin.  Go to https://portal.tacc.utexas.edu/ to create an account.  We recommend our students use the same username at TACC and Cyverse.

## Cyverse (UA)

The Cyverse project is an NSF-funding cyberinfrastructure project headquartered at the University of Arizona.  It began life as "iPlant" in 2008. Our students may choose to make use of Cyverse infrastructure, as well, such as "apps" for assemblies, gene calling, protein clustering, and such.  Students should go to https://user.cyverse.org/ to create an account.  Again, we recommend you use the same username as your TACC account to make communication between the TACC and UA systems easier.