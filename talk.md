
# Introduction to the PDC environment

## Radovan Bast

PDC Center for High Performance Computing &
Department of Theoretical Chemistry and Biology,
KTH Royal Institute of Technology, Stockholm

---

layout: false

.left-column[
# Outline
]

.right-column[
- Different types of nodes
- System access
- Unix shell (bash)
- File systems, permissions, and transfer
- Parallel computing and parallel programming
- Programming environment
- How to run programs
- Software development tools and good practices
- Performance analysis and debugging
- How to get help
]

---

## Different types of nodes

- A node is like a computer without keyboard or monitor, and often without disk
- A cluster consists of many nodes
- A node typically contains several processors
- A processor typically contains several cores
- Zoo of nodes:
    - Login nodes
    - Compute nodes
    - Shared interactive nodes
    - Exclusive interactive nodes
    - Transfer nodes
    - Service nodes
- A login node is for submitting jobs, editing files, and compiling small programs

---

template: inverse

# System access

---

## System access

- PDC uses SSH together with Kerberos for authentication and login
- Kerberos protocol for authentication developed at MIT
- Non-US version developed at KTH
- Uses tickets to authenticate

## Kerberos jargon

- **Ticket**: proof of identity encrypted with a secret key for the particular service requested
  (tickets have a lifetime)
- **Realm**: collection of resources that authenticate against a central user/service database
- **Principal**: unique identity to which Kerberos assigns tickets
- **Key Distribution Center (KDC)**: service which stores encrypted secret keys

---

## Kerberos

- Avoids storing passwords locally or sending them over the internet
- Avoids re-typing passwords
- Involves a trusted 3rd-party
- Built on symmetric-key cryptography
- Offers less attack vectors than SSH with passwords
- Kerberos v5 software (Heimdal or MIT) needed to get Kerberos tickets
- Need kerberized SSH software that supports GSSAPI with KeyExchange
- For installation instructions see https://www.pdc.kth.se/resources/software/login-1

---

## Kerberos

.left-column[
## Linux
]

.right-column[
- Installation: https://www.pdc.kth.se/resources/software/login-1/linux/

- Login

```shell
# without editing ~/.ssh/config
kinit --forwardable user@NADA.KTH.SE
ssh -o GSSAPIKeyExchange=yes -o GSSAPIAuthentication=yes \
    -o GSSAPIDelegateCredentials=yes user@machine.pdc.kth.se

# with editing ~/.ssh/config; see PDC web
kinit --forwardable user@NADA.KTH.SE
ssh user@machine.pdc.kth.se

# after editing krb5.conf (see PDC web) you can drop the realm
kinit user
ssh user@machine.pdc.kth.se
```
]

---

- It is convenient to insert the following into `~/.ssh/config`:

```shell
# Hosts we want to authenticate to with Kerberos
Host *.kth.se *.kth.se.
# User authentication based on GSSAPI is allowed
GSSAPIAuthentication yes
# Key exchange based on GSSAPI may be used for server authentication
GSSAPIKeyExchange yes

# Hosts to which we want to delegate credentials. Try to limit this to
# hosts you trust, and were you really have use for forwarded tickets.
Host *.csc.kth.se *.csc.kth.se. *.nada.kth.se *.nada.kth.se. *.pdc.kth.se *.pdc.kth.se.
# Forward (delegate) credentials (tickets) to the server.
GSSAPIDelegateCredentials yes
# Prefer GSSAPI key exchange
PreferredAuthentications gssapi-keyex,gssapi-with-mic

# All other hosts
Host *
```

- For the configuration of krb5.conf, see https://www.pdc.kth.se/resources/software/login-1

---

## Kerberos

.left-column[
## Windows
]

.right-column[
- PuTTY (only PuTTY downloaded from this link works at PDC)
  https://www.pdc.kth.se/resources/software/login-1/windows/putty
- SecureCRT
  https://www.pdc.kth.se/resources/software/login-1/windows/securecrt
- We prefer PuTTY because we prefer to pay less license fees
- Another option is CygWin
]

---

## Kerberized PuTTY

.left-column[
## Windows
]

.right-column[
![]({{ base }}/img/putty-kerberos.png)
]

---

## Kerberos

.left-column[
## Mac OS X
]

.right-column[
- Mac OS X Lion 10.7.2 and above come with Heimdal and kerberized SSH
- Mac OS X Snow Leopard comes with MIT Kerberos and kerberized SSH
- For configuration of see https://www.pdc.kth.se/resources/software/login-1/macintosh
]

---

## Working with Kerberos

```shell
# get a new forwardable ticket
kinit --forwardable user@NADA.KTH.SE

# list available tickets
klist

# destroy tickets
kdestroy

# change password
kpasswd
```

---

## Good practice

![]({{ base }}/img/kerberos-good.png)

- Only type your password on your local machine that has an up-to-date OS and that you trust

---

## Bad practice

![]({{ base }}/img/kerberos-bad.png)

- Do not kinit on a remote machine
* PDC accounts cannot be shared

---

template: inverse

# Introducing the unix shell

---

## Introducing the unix shell

- We are greeted with a command-line interface (CLI)
- Teleported back to the 70ies
```shell
$ ssh beskow.pdc.kth.se
Last login: Fri Feb 13 20:20:06 2015 from example.com
bast@beskow-login2:~$ _
```
- CLI often more efficient than GUI
- High action-to-keystroke ratio (expense: terse and "cryptic")
- Creativity through pipelines
- System is configured with text files
- Calculations are configured and run using text files
- Good for working over network
- Good for reproducibility
- Good for unsupervised workflows

---

## Bash: Files and directories

- Command `pwd` tells me where I am. After login I am in the "home" directory:
```shell
user@machine:~$ pwd
/afs/pdc.kth.se/home/u/user
```

- I can change the directory with `cd`:
```shell
user@machine:~$ cd tmp/talks/
user@machine:~/tmp/talks$ pwd
/afs/pdc.kth.se/home/u/user/tmp/talks
```
- I can go one level up with `cd ..`

- List the contents with `ls -l`:
```shell
user@machine:~/tmp/talks$ ls -l
total 237
drwx------ 3 user csc-users   2048 Aug 17 15:21 img
-rw------- 1 user csc-users  18084 Aug 17 15:21 pdc-env.html
-rw------- 1 user csc-users 222051 Aug 17 15:22 remark-latest.min.js
```

---

- Files and directories form a tree
- We can explore the tree with `ls`, and `cd`:

```shell
user@machine:~/tmp/talks$ ls -l img/
total 343
drwx------ 2 user csc-users   2048 Aug 17 15:21 kerberos
-rw------- 1 user csc-users 310579 Aug 17 15:21 xc30-blade.png
-rw------- 1 user csc-users  37812 Aug 17 15:21 xc30-cabinet.jpg
```

- All these commands bring you back to home:

```shell
$ cd $HOME
$ cd ~
$ cd
$ cd /afs/pdc.kth.se/home/u/user
```

---

## Bash: Creating directories and files

- We create  a new directory called "results" and change to it:
```shell
$ mkdir results
$ cd results
```

## Creating and editing files

- Easy but not powerful:
```shell
$ nano draft.txt
```

- More powerful: Emacs or Vi(m):
```shell
$ emacs draft.txt
$ vi    draft.txt
$ vim   draft.txt # this is Vi "improved"
```

---

## Copying, moving, renaming, and deleting

```shell
# copy file
$ cp draft.txt backup.txt

# recursively copy directory
$ cp -r results backup

# move/rename file
$ mv draft.txt draft_2.txt

# move/rename directory
$ mv results backup

# move directory one level up
$ mv results ..

# remove file
$ rm draft.txt

# remove directory and all its contents
$ rm -r results
```
- From the Unix point of view, deleting is forever!

---

## Bash: History and tab completion

- From the Unix point of view, deleting is forever!

```shell
$ history

1860  vi /home/user/devel/gpunch/src/twoints/EriBlock.cpp
1861  cd ..
1862  git grep GenPrimSSSD
1863  vi src/twoints/EriBlock.h
1864  find . | grep SSSD
1865  vi ./src/twoints/GenPrimSSSD.h
1866  git pull
1867  cd build/
1868  make -j12
```

- Commands are numbered, I can repeat a command by number:
```shell
!1864
```

- Use the tab key for tab completion
- Try CTRL+R and type a command you used in the past

---

## Bash: Finding things

- Extract lines which contain an expression with `grep`:

```shell
# extract all lines that contain "fixme"
$ grep fixme draft.txt
```

- Unix commands have many options/flags - examine them with `man`:

```shell
$ man grep
```

- Another useful command is `apropos` - try it

- Find files with `find`:

```shell
$ find ~ | grep lostfile.txt
```

- We can pipe commands and filter results with `|`

```shell
$ grep energy results.out | sort | uniq
```

---

## Bash: Redirecting output

- Redirect output to a file:

```shell
$ grep energy results.out | sort | uniq > energies.txt
```

- Append output to a file:

```shell
$ grep dipole results.out | sort | uniq >> energies.txt
```

- Print contents of a file to screen:

```shell
$ cat results2.txt
```

- Append contents of a file to another file:

```shell
$ cat results2.txt >> results_all.txt
```

---

## Bash: Writing shell scripts

```shell
#!/usr/bin/env bash

# here we loop over all files that end with *.out
for file in *.out; do
    echo $file
    grep energy $file
done
```

- We make the script executable and execute it:

```shell
# make it executable
$ chmod u+x my_script

# run it
$ ./my_script
```

---

## Bash: Passing arguments to scripts

```shell
#!/usr/bin/env bash

echo "the script was called with the arguments: $1 and $2"
```

- Arguments are numbered with `$1`, `$2`, `$3`, etc.

- We can now call the script with:

```shell
$ ./my_script apples oranges

the script was called with the arguments: apples and oranges
```

---

template: inverse

# File systems, permissions, and transfer

---

## File systems at PDC

- AFS (Andrew File System)
    - distributed
    - global
    - backup

- Lustre (Linux cluster file system)
    - distributed
    - high-performance
    - no backup

---

## AFS

- Andrew File System
- Named after the Andrew Project (Carnegie Mellon University)
- Distributed file system
- Homogeneous, location-transparent file name space
- Security and scalability
- Accessible "everywhere" (remember that when you make your files readable/writeable!)
- Access via Kerberos tickets and AFS tokens
- Your PDC home directory is located in AFS, example:

```shell
/afs/pdc.kth.se/home/u/user
```

- OldFiles mountpoint (created by default) contains a snapshot of
  the files as they were precisely before the last nightly backup was
  taken.

```shell
/afs/pdc.kth.se/home/u/user/OldFiles
```

- By default you get a very limited quota (0.5 GB) but you can ask for more

---

## AFS permissions

- You probably know about Unix file permissions (chown, chmod):

```shell
-rw-r--r-- 1 user csc-users  3153 Feb 20 11:04 intro_pdc.rst
-rw-r--r-- 1 user csc-users   175 Feb 16 20:50 Makefile
```

- AFS permissions work differently:

```shell
$ fs listacl
Access list for . is
Normal rights:
  user:remote-users rlidwka
  system:anyuser l
  user rlidwka
```

- Google "AFS ACL" to find out how to change permissions.
- Example: give user "alice" read permissions for the current directory:

```shell
$ fs setacl . alice read
```

- Always remember that AFS is global.

---

## Lustre

- Parallel distributed file system
- Large-scale cluster computing
- High-performance
- `/cfs/klemming`
- Unix permissions
- Not global

---

## Overview: PDC storage

- `/afs`
    - Home
    - Small, quota
    - **Backup**
    - AFS permissions
    - Not good for temporary job files

- `/cfs`
    - Large, no quota (but please be considerate and do not fill up the disk)
    - **No backup**
    - Unix permissions
    - Good for temporary job files
    - Submit jobs from `/cfs` (except Ferlin, explanation: next slide)

---

## Overview: mountpoints

```shell
*Beskow    /cfs/klemming/nobackup  # persistent files
*          /cfs/klemming/scratch   # temporary files
*          /afs                    # only login node

Milner    /cfs/milner/scratch
          /cfs/klemming/nobackup
          /cfs/klemming/scratch
          /afs                    # only login node

Ferlin    /scratch                # temporary files, local disks
          /afs                    # persistent files

Povel     /cfs/klemming/nobackup  # persistent files
          /cfs/klemming/scratch   # temporary files
          /afs                    # important files, backup

Ellen     /cfs/klemming/nobackup  # persistent files
          /cfs/klemming/scratch   # temporary files
          /afs                    # important files, backup
```

---

## File transfer between PDC machines

- `/afs` is mounted and visible on all machines (at least login node)
- No need to "transfer" files which are on `/afs`
- You can share large files between machines via `/cfs` (except Ferlin)

## PDC machines and outside world (Linux and Mac OS X)

```shell
# from my laptop to Beskow
$ scp myfile username@beskow.pdc.kth.se:~/Private

# from Beskow to my laptop
$ scp username@beskow.pdc.kth.se:~/Private/myfile .

# from Beskow to Triolith (NSC)
$ ssh user@beskow.pdc.kth.se
$ scp Private/myfile user@triolith.nsc.liu.se:
```

- If the username is the same on source and destination machine, you can leave it out

---

## Host aliases

- If you are tired of typing long hostnames, create aliases in `~/.ssh/config`

```shell
Host trio
     Hostname triolith.nsc.liu.se
     ForwardX11 yes
     User eminem
```

## PDC machines and outside world (Windows)

- PuTTY (only PuTTY downloaded from this link works at PDC)
  https://www.pdc.kth.se/resources/software/login-1/windows/putty

```shell
# have not tested this myself
C:\Program Files\PuTTY\pscp.exe myfile username@beskow.pdc.kth.se:~/Private
```

---

## Alternative to scp

- Install AFS client and copy directly
- Then AFS is mounted just like another disk on your computer
- https://www.pdc.kth.se/resources/software/file-transfer/file-transfer-with-afs/linux
- https://www.pdc.kth.se/resources/software/file-transfer/file-transfer-with-afs/mac
- https://www.pdc.kth.se/resources/software/file-transfer/file-transfer-with-afs/windows

## Transferring large (or very many) files to `/cfs`

- In this case do not scp to the login node
- Transfer files to the transfer node

```shell
# from my laptop to Beskow
$ scp gigantic.tar.gz username@cfs-aux-4.pdc.kth.se:/cfs/klemming/nobackup/u/user
```

---

template: inverse

# Parallel computing and parallel programming

---

## Parallel computing

- Typical example

```c
double r = 0.0;

// n is reasonably large
for (int i = 0; i < n; i++)
{
    // this routine does most of the work
    r += compute_something(...);
}
```

- We would try to split the work (loop) among cores/threads/workers/tasks
- Each thread should either do something different or do the same thing on different data
- We need to think about synchronization
- We need to think about load balance

---

## Typical parallel programming standards

- **MPI** (message passing interface)
    - C, C++, Fortran, R, Python
    - Distributed memory (mostly)
    - Explicit
- **OpenMP** (CPU, GPU)
    - C, C++, Fortran
    - Shared memory
    - Implicit
- **CUDA** (GPU)
    - NVIDIA
    - C, C++, Fortran
- **OpenACC** (GPU and accelerators)
    - NVIDIA, Cray, PGI
- **OpenCL** (GPU, CPU, and accelerators)
    - AMD, Intel
- ...

---

## Parallel performance: Ideal vs. real

- It is **very important** to check that increase in number of processors is worth it
- In your interest (user/developer) and in our interest (computing center)

![]({{ base }}/img/amdahl.png)

---

## Parallel performance bottlenecks

- Synchronization overhead
- False sharing
- Load imbalance
- Amdahl law
- Memory bandwidth
- File I/O

---

## Developers facing parallelism

- There are several levels of parallelism
- Frequency race is over
- Vectorization is back
- Memory is limited
- File I/O should be minimal
- Single core performance will not matter in future
- Codes which cannot scale will not run on future hardware
- Not only time to rewrite codes, it is also time to revisit algorithms

---

template: inverse

# Programming environment

---

## Working with modules

- At PDC we use the module environment.
- Modules are used to load specific software into your environment.

```shell
# list all available modules
$ module avail

# show information about a module
$ module show fftw/3.3.0.4

# load a module
$ module load fftw/3.3.0.4

# list currently loaded modules
$ module list

# swap modules
$ module swap PrgEnv-cray PrgEnv-intel

# unload module
$ module unload fftw/3.3.0.4
```

---

```shell
$ module list # on Beskow

Currently Loaded Modulefiles:
  1) modules/3.2.6.7
  2) nodestat/2.2-1.0502.53712.3.109.ari
  3) sdb/1.0-1.0502.55976.5.27.ari
  4) alps/5.2.1-2.0502.9041.11.6.ari
  5) lustre-cray_ari_s/2.5_3.0.101_0.31.1_1.0502.8394.10.1-1.0502.17198.8.51
  6) udreg/2.3.2-1.0502.9275.1.12.ari
  7) ugni/5.0-1.0502.9685.4.24.ari
  8) gni-headers/3.0-1.0502.9684.5.2.ari
  9) dmapp/7.0.1-1.0502.9501.5.219.ari
 10) xpmem/0.1-2.0502.55507.3.2.ari
 11) hss-llm/7.2.0
 12) Base-opts/1.0.2-1.0502.53325.1.2.ari
 13) craype-network-aries
 14) craype/2.2.1
 15) cce/8.3.4
* 16) cray-libsci/13.0.1
 17) pmi/5.0.5-1.0000.10300.134.8.ari
 18) rca/1.0.0-2.0502.53711.3.127.ari
 19) atp/1.7.5
* 20) PrgEnv-cray/5.2.40
* 21) craype-haswell
* 22) cray-mpich/7.0.4
 23) slurm/slurm-14.11.4
 24) openssh/5.3p1-with-pam-gsskex-20100124
 25) openafs/1.6.10-3.0.101-0.31.1_1.0502.8394-cray_ari_s
 26) heimdal/1.5.2
 27) snic-env/1.0.0
 28) slurm-utils/0.0
 29) pdc-utils/0.0
```

---

## Compiling code on Beskow

- Available compiler sets: Cray, GNU, and Intel
- By default, the Cray compiler set is loaded
- Use `module swap` to switch between these

```shell
# select Intel
$ module swap PrgEnv-cray PrgEnv-intel
```

- Module `cray-libsci` provides BLAS, LAPACK, BLACS, and SCALAPACK
- Module `cray-mpich` provides MPI
- On Cray we compile using compiler wrappers: ftn, cc, and CC

### Advantages

- Compiler wrappers will automatically link to BLAS, LAPACK, BLACS, SCALAPACK, FFTW
- Compiler wrappers will automatically use MPI wrappers

### Disadvantage

- Sometimes you need to edit Makefiles which are not designed for Cray

---

.left-column[
## Using compiler wrappers on Cray
]

.right-column[
- Fortran

```shell
$ gfortran [flags] source.F90 # wrong
$ ifort    [flags] source.F90 # wrong

# correct
$ ftn [flags] source.F90
```

- C

```shell
$ gcc [flags] source.c # wrong
$ icc [flags] source.c # wrong

# correct
$ cc [flags] source.c
```

- C++

```shell
$ g++  [flags] source.cpp # wrong
$ icpc [flags] source.cpp # wrong

# correct
$ CC [flags] source.cpp
```
]

---

.left-column[
## Using compiler wrappers on Cray
]

.right-column[
- Fortran

```shell
$ mpif90   [flags] source.F90 # wrong
$ mpiifort [flags] source.F90 # wrong

# correct
$ ftn [flags] source.F90
```

- C

```shell
$ mpicc  [flags] source.c # wrong
$ mpiicc [flags] source.c # wrong

# correct
$ cc [flags] source.c
```

- C++

```shell
$ mpicxx  [flags] source.cpp # wrong
$ mpiicpc [flags] source.cpp # wrong

# correct
$ CC [flags] source.cpp
```

- Same wrappers for MPI and sequential code
]

---

## Compiling code on Ferlin/Povel

- **Intel**

```shell
$ ifort -openmp source.F90
$ icc   -openmp source.c
$ icpc  -openmp source.cpp
```

- PGI

```shell
$ pgf90 -mp source.F90
$ pgcc  -mp source.c
$ pgcpp -mp source.cpp
```

- GNU

```shell
$ gfortran -fopenmp source.F90
$ gcc      -fopenmp source.c
$ g++      -fopenmp source.cpp
```

---

## Compiling code on Ferlin/Povel

- MPI (OpenMPI or MPICH)

```shell
$ mpif90 source.F90
$ mpicc  source.c
$ mpicxx source.cpp
```

- Intel MPI

```shell
$ mpiifort source.F90
$ mpiicc   source.c
$ mpiicpc  source.cpp
```

## General recommendation (any machine, any vendor)

- Study the effect of optimization flags

---

template: inverse

# How to run programs

---

## How to run programs

- After login we are on the login node
- A login node is for submitting jobs, editing files, and compiling small programs
- **We never run calculations interactively on the login node**
- We want balanced load of the resources
- Fair share according to time allocations
- Rather we use a queuing/batch system

![]({{ base }}/img/waiting.jpg)

- Only persons/groups belonging to a Charge Account Category (CAC; in other words time allocation) can submit

---

## Queuing systems at PDC

- Beskow
    - Slurm
    - Submit a batch script or book interactive node

- Ferlin/Povel
    - EASY
    - Submit a batch script or book interactive node

- Milner
    - Slurm
    - Submit a batch script or book interactive node

- Ellen
    - Only interactive

---

## MPI launchers

- Beskow and Milner
    - aprun

```shell
aprun -n 48 ./my_exe > my_output
```

- Other machines
    - mpirun or mpiexec

```shell
mpirun -np 48 ./my_exe > my_output
```

---

## Slurm basics

```shell
# submit the job
$ sbatch script.slurm

# see information about all your jobs
$ squeue -u $USER

# remove or stop a job
$ scancel [jobid]

# set the default allocation for your job
$ sacctmgr update user <user> set defaultaccount=<alloc> where cluster=beskow
```

- All jobs must belong to an allocation or they will only start if no other jobs are queued.

---

## Example Slurm job script (MPI)

```shell
#!/bin/bash --login

# name of the job
#SBATCH -J my_job

# wall-clock time given to this job (20 minutes)
#SBATCH -t 00:20:00

# set the allocation to be charged for this job
# not required if you have set a default allocation
#SBATCH -A 201X-X-XX

#SBATCH -o stdout.txt
#SBATCH -e stderr.txt

# number of nodes
#SBATCH -N 2

# number of MPI tasks per node (the following is actually the default)
#SBATCH --ntasks-per-node=32

# number of MPI tasks
#SBATCH -n 64

# run the executable named my_exe and write the output to my_output
cd $SLURM_SUBMIT_DIR
aprun -n 64 ./my_exe > my_output 2>&1
```

---

## Example Slurm job script (hybrid MPI/OpenMP)

```shell
#!/bin/bash --login

#SBATCH -J my_job
#SBATCH -t 00:20:00
#SBATCH -A 201X-X-XX
#SBATCH -o stdout.txt
#SBATCH -e stderr.txt

# number of nodes
#SBATCH -N 256

# number of MPI tasks per node
#SBATCH --ntasks-per-node=4

# number of MPI tasks
#SBATCH -n 1024

# number of cores hosting OpenMP threads
#SBATCH -c 8

export OMP_NUM_THREADS=8

cd $SLURM_SUBMIT_DIR
aprun -n 1024 -N 4 -d 8 -cc none ./myexe > my_output 2>&1
```

- In this case the job script is a Bash script but it can be Python or Perl instead

---

## How to get more memory

- Run 64 tasks with 32 tasks per node
```shell
$ aprun -n 64 -N 32 ./a.out
```

- Run the same MPI application but spread over twice as many nodes
```shell
$ aprun -n 64 -N 16 ./a.out
```

---

## Booking an interactive node (Beskow)

- You can request an interactive node with `salloc`:

```shell
$ salloc [-N 1 -t 60]

salloc: Granted job allocation 21223
```

- Then you get a node for interactive use.
- You can log out with `exit`:

```shell
$ exit

salloc: Relinquishing job allocation 21223
salloc: Job allocation 21223 has been revoked.
```

---

## Working with an interactive node

- This runs on the login node:

```shell
$ salloc
$ ./program
```

- This runs on a compute node node:

```shell
$ salloc
$ aprun -n 32 ./program
```

- You cannot (easily) log into a compute node directly.
- Note that compute nodes have Haswell and login nodes have Sandy Bridge.
- If you have a clever configure script which detects the processor,
  you may need to configure via salloc and aprun.

---

## Important note

On Beskow submit all jobs from `/cfs`

```shell
# submit from here
/cfs/klemming/nobackup/u/user
```

and **not** from your home directory on `/afs`

```shell
# not here
/afs/pdc.kth.se/home/u/user
```

---

template: inverse

# Software development tools and good practices

---

## Software development tools and good practices

- Version control (Git or Hg)
- Unit tests, regression tests, test-driven development
- Cross-platform configuration tool: CMake
- Code documentation with Sphinx
- Profile before you optimize anything
- Think about the cache
- http://toolbox.readthedocs.org

---

## Building portable software with CMake

- Write one `CMakeLists.txt` file which controls configuration

```cmake
cmake_minimum_required(VERSION 2.8 FATAL_ERROR)

project(ExcitingProject)

enable_language(Fortran)

# this is where we will place the Fortran module files
set(CMAKE_Fortran_MODULE_DIRECTORY ${PROJECT_BINARY_DIR}/modules)

# the executable is built from 3 source files
add_executable(
    exciting_project.x
    main.F90
    foo.F90
    bar.F90
    )
```

- CMake generates a Makefile for you

---

## Code documentation with Sphinx

- Write lightweight RST markup
- Generate html or LaTeX
- Host on https://readthedocs.org

## Continuous Integration with Travis

- Host code on GitHub
- Write tests using Google Test or pytest or nose or your favourite testing framework
- Deploy testing on https://travis-ci.org

---

template: inverse

# Performance analysis and debugging

---

## Performance analysis

- CrayPAT
- Allinea Performance Reports
- Vampir
- Paraver

## Debugging

- DDT
- TotalView
- Cray ATP
- gdb

---

## CrayPAT

```
 Samp%  | Samp  | Imb.  |  Imb.  |Group
        |       | Samp  | Samp%  | Function
        |       |       |        |  PE=HIDE

 100.0% |   6.0 |    -- |     -- |Total
|-----------------------------------------------------
|  87.5% |   5.2 |    -- |     -- |MPI
||----------------------------------------------------
||  72.9% |   4.4 |   0.6 |  14.3% |MPI_SENDRECV
||  14.6% |   0.9 |   0.1 |  14.3% |mpi_finalize
||====================================================
|  10.4% |   0.6 |    -- |     -- |ETC
||----------------------------------------------------
||   4.2% |   0.2 |   1.8 | 100.0% |png_write_chunk
||   2.1% |   0.1 |   0.9 | 100.0% |inflate
||   2.1% |   0.1 |   0.9 | 100.0% |deflateSetDictionary
||   2.1% |   0.1 |   0.9 | 100.0% |deflateEnd
||====================================================
|   2.1% |   0.1 |   0.9 | 100.0% |IO
||----------------------------------------------------
|   2.1% |   0.1 |   0.9 | 100.0% | write
|=====================================================
```

- http://docs.cray.com/books/S-2315-52/html-S-2315-52/z1055157958smg.html

---

## Allinea Performance Reports

![]({{ base }}/img/apr-example-summary.png)

---

## Vampir

![]({{ base }}/img/vampir.png)

- Example image courtesy Magnus Helmersson
- https://www.vampir.eu

---

## Paraver

![]({{ base }}/img/paraver.jpg)

- Example image courtesy Xavi Aguilar
- http://www.bsc.es/computer-sciences/performance-tools/paraver/

---

template: inverse

# How to get help

---

## PDC support

- Many questions can be answered by reading the web documentation: https://www.pdc.kth.se/support
- **Preferably contact PDC support by email**: support@pdc.kth.se
- Or by phone: +46 (0)8 790 7800
- You can also make an appointment to come and visit.
- Your email support request will be tracked - you get a ticket number.
- For follow-ups/replies always include the ticket number - they look like this: ``[SNIC support #12345]``

---

## How to report problems

- Do not report new problems by replying to old/unrelated tickets.
- Split unrelated problems into separate email requests.
- Use a descriptive subject in your email (unhelpful subject line: "problem").
- Give your PDC user name.
- Be as specific as possible.
- For problems with scripts/jobs, give an example. Either send the example or make it accessible to PDC support.
- Make the problem example as small/short as possible.
- Provide all necessary information to reproduce the problem.
- If you want the PDC support to inspect some files, make sure that the files are readable.
- Do not assume that PDC support personnel have admin rights to see all your files or change permissions.

### References

- A. Turner, X. Guo, L. Axner, M. Filipiak, Best Practice Guide - Cray XE/XC (V4.1), 2013 (http://www.prace-ri.eu/Best-Practice-Guide-Cray-XE-XC-HTML).
- https://www.pdc.kth.se/education/course-resources/introduction-to-cray-xc30-xc40/feb-2015/agenda
