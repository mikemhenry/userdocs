# FAQ

## Description

This page provides a collection of frequently asked questions.

## File system

## What if my HOME is full?
If you reached your quota, you will get strange warning about not being able to write temporary files etc. You can check your quota using the 
1. Decluttering: Check for unnecessary data. This could be:

- unused application packages, e.g. Python(2) packages in `$HOME/.local/lib/python*/site-packages/*`
- temporary computational data, like already post processed output files
- duplicated data
- ...

2. Pack and archive: The HPC storage is a high performance parallel storage and not meant to be an archive. Data not used in the short to midterm should be packed and moved to an archive storage. 

In general, we consider data on our HPC systems as research data. Further we consider research data to be shared sooner or later. And we aim to support and enhance collaborations. Therefore, we introduce group shared spaces, called HPC Workspaces.
Ask your research group manager to add you to an existing Workspace or create a new one. 
There will be no quota increase for HOME directories. 

## Workspaces
### I need access to a HPC Workspace, whome I need to ask?

HPC Workspaces are managed by the group manager/leader and if applicable a deputy. Therewith you need to ask them to add you to the primary or secondary group. See also [HPC Workspace members](../hpc-workspaces/workspaces.md#members).

### I need to share data with my colleges. What can I do?
HPC Workspaces are meant to host shared data. See [HPC Workspaces](../hpc-workspaces/workspaces.md)

<!-- ## Where should I put my data?
A coarse classification may be: 

| data type | suggested target |
| :--- | :--- |
| private configuration data, e.g. SSH keys | HOME |
| temporary (weeks to month) application input/output data | SCRATCH |
| persistent application input/results, meant to be shared (some-when) | Workspace |
| applications, meant to be shared (some-when) | Workspace | -->

### Where can I get a Workspace?
A research group manager need to **create** the Workspace, since there are possibilities for charged extensions. 

If you want to **join an existing** Workspace. Ask the Workspace manager or its deputy to add you. 
See [HPC Workspaces](../hpc-workspaces/workspaces.md)

### How much does a Workspace cost?
Workspaces itself are free of charge. Every research group has 10TB disk space free of charge, which can be used in multiple Workspaces. 
If necessary, additional storage can be purchased per Workspace, where only the actual usage will be charged, see [Workspace Management](../hpc-workspaces/management.md#additional-storage)


### What if our 10TB free of charge research group quota is full?
Your Research group manager or a registered deputy can apply for an additional quota. Actual used quota will be charged. 

### Why can I not submit jobs anymore?
After joining an HPC Workspace the private lsf account gets deactivated and a Workspace account need to be specified. 
This can be done by loading the Workspace module, see [Workspace environment](../hpc-workspaces/environment.md):

```Bash 
module load Workspace
```

Otherwise lsf will present the following error message:
```Bash
sbatch: error: AssocGrpSubmitJobsLimit
sbatch: error: Batch job submission failed: Job violates accounting/QOS policy (job submit limit, user's size and/or time limits)
```

With this method we aim to distribute our resources in a more fair manner. HPC resources including compute power should be distriuted between registered research groups. We can only relate users with research groups by utilizing Workspace information. 


## Software issues
### Why is my private conda installation broken after migration
Unfortunately, Anaconda hard wires absolute paths into almost all files (including scripts and binary files). 
A proper migration process may have included `conda pack`. 
There is a way you may access your old environments and create new ones with the same specification:
```
export CONDA_ENVS_PATH=${HOME}/anaconda3/envs ## or where you had your old envs
module load Anaconda3
eval "$(conda shell.bash hook)"
conda info --envs
conda activate oldEnvName     ## choose your old environment name
conda list --explicit > spec-list.txt
unset CONDA_ENVS_PATH
conda create --name myEnvName --file spec-list.txt  # select a name
```
Please, also note that there is a system wide Anaconda installation, so no need for your own separate one. 
Finally, after recreating your environments please delete all old Anaconda installations and environments. These are not only big but also a ton of files. 

### Why the system is complaining abount not finding an existing module?

There are cases modules could not be found. This could be that the modules is not exiting in the target software stack, it could be hidden, or a version inconsitency. 

#### hidden modules
Some modules are provided as hidden modules to keep the presented software stack nice and clean. Hidden modules can be listed using `module --show-hidden avail`.

#### software stacks
On UBELIX there are multiple software stacks. There are software stacks for each architecture. There are custom software stacks in Workspaces (again architectural software stacks included) and the VitalIT software stack. 
The targeted software stack need to be available. The different architectural software stacks are available on the related architecture, e.g. epyc2 in a job on the epyc2 partion. The Workspace and VitalIT software stack can be loaded using `module load Workspace` or module load vital-it. 

#### software stack inconstency
It is strongly suggested to not mix different toolchains like foss or intel. Additionally, it is advised to stay with one version of a toolchain, e.g. foss/2021a and its dependency versions, e.g. GCC/10.3.0 etc. 
Further, LMOD has a confusing effect when loading inconsitent module combinations, e.g.

```Bash
$ module load foss/2021a
$ module load intel/2020b
Lmod has detected the following error:  The following module(s) are unknown: "zlib/.1.2.11-GCCcore-10.2.0"

Please check the spelling or version number. Also try "module spider ..."
It is also possible your cache file is out-of-date; it may help to try:
  $ module --ignore-cache load "zlib/.1.2.11-GCCcore-10.2.0"

Also make sure that all modulefiles written in TCL start with the string #%Module
```
The mentioned module `zlib/.1.2.11-GCCcore-10.2.0` is available in general. 
When loading `foss/2021a`, the `zlib/.1.2.11-GCCcore-10.3.0` should get loaded, but LMOD will not swap its version, but report the mentioned error. 

Please take this as an indication that you accidentality mix different toolchains, and rethink your procedure, and stay within the same toolchain and toolchain version. 

## Environment issues
### I am using zsh, but some commands and tools fail, what can I do?
There are known caveats with LMOD (or module system) and Bash scripts in zsh environments. Bash scripts do not source any system or user files. To initialize the (module) environment properly, you need to set `export BASH_ENV=/etc/bashrc` in your zsh profile (`.zshrc`).

### I modified my bashrc, but its not doing what I expect, how can I debug that bash script?
The bashrc can be debugged as all other bash scripts, using 

- `set -x` at the beginning of the script. This will print **all** commands executed on screen, including all subcommand also included in called scripts and tools
- print statements, e.g. `echo "DEBUG: variable PATH=$PATH"`

These should provide a good indication where the script diverge from your expectation. 

## Job issues
### Why is my job still pending?

!!! types note ""
    The REASON column of the _squeue_ output gives you a hint why your job is not running.

**(Resources)**  
The job is waiting for resources to become available so that the jobs resource request can be fulfilled.

**(Priority)**  
The job is not allowed to run because at least one higher prioritized job is waiting for resources.

**(Dependency)**  
The job is waiting for another job to finish first (--dependency=... option).

**(DependencyNeverSatisfied)**  
The job is waiting for a dependency that can never be satisfied. Such a job will remain pending forever. Please cancel such jobs.

**(QOSMaxCpuPerUserLimit)**  
The job is not allowed to start because your currently running jobs consume all allowed CPU resources for your user in a specific partition. Wait for jobs to finish.

**(AssocGrpCpuLimit)**  
dito.

**(AssocGrpJobsLimit)**  
The job is not allowed to start because you have reached the maximum of allowed running jobs for your user in a specific partition. Wait for jobs to finish.

**(ReqNodeNotAvail, UnavailableNodes:...)**  
Some node required by the job is currently not available. The node may currently be in use, reserved for another job, in an advanced reservation, `DOWN`, `DRAINED`, or not responding.**Most probably there is an active reservation for all nodes due to an upcoming maintenance downtime (see output of** `scontrol show reservation`) **and your job is not able to finish before the start of the downtime. Another reason why you should specify the duration of a job (--time) as accurately as possible. Your job will start after the downtime has finished.** You can list all active reservations using `scontrol show reservation`.

### Why can't I submit further jobs?

!!! types note ""
    _sbatch: error: Batch job submission failed: Job violates accounting/QOS policy (job submit limit, user's size and/or time limits)_


... means that you have reached the maximum of allowed jobs to be submitted to a specific partition.

### Job in state FAILED although job completed successfully

lsf captures the return value of the batch script/last command and reports this value as the completion status of the job/job step. lsf indicates status FAILED if the value captured is non-zero.

The following simplified example illustrates the issue:

**simple.c**

```Bash
#include <unistd.h>
#include <stdio.h>
int main (int argc, char *argv[]) {
  char hostname[128];
  gethostname(hostname, sizeof(hostname));
  printf("%s says: Hello World.\n", hostname);
}
```

**job.sh**

```Bash
#!/bin/bash
# lsf options
#SBATCH --mail-user=foo@bar.unibe.ch
#SBATCH --mail-type=END
#SBATCH --job-name="Simple Hello World"
#SBATCH --time=00:05:00
#SBATCH --nodes=1
# Put your code below this line
./simple
```

```Bash
bash$ sbatch job.sh
Submitted batch job 104
```

Although the job finished successfully...

**lsf-104.out**

```Bash
knlnode02.ubelix.unibe.ch says: Hello World.
```

...lsf reports job FAILED:

```Bash
bash$ sacct -j 104
       JobID    JobName  Partition    Account  AllocCPUS      State ExitCode
------------ ---------- ---------- ---------- ---------- ---------- --------
104          Simple He+        all                     1     FAILED     45:0
104.batch         batch                                1     FAILED     45:0
```


Problem: The exit code of the job is the exit status of batch script (job.sh) which in turn returns the exit status of the last command executed (simple) which in turn returns the return value of the last statement (printf()). Since printf() returns the number of characters printed (45), the exit code of the batch script is non-zero and consequently lsf reports job FAILED although the job produces the desired output.

Solution: Explicitly return a value:

```Bash
#include <unistd.h>
#include <stdio.h>
int main (int argc, char *argv[]) {
  char hostname[128];
  int n;
  gethostname(hostname, sizeof(hostname));
  // If successful, the total number of characters written is returned. On failure, a negative number is returned.
  n = printf("%s says: Hello World.\n", hostname);
  if (n < 0)
    return 1;
  return 0;
}
```

```Bash
bash$ sacct -j 105
       JobID    JobName  Partition    Account  AllocCPUS      State ExitCode
------------ ---------- ---------- ---------- ---------- ---------- --------
105          Simple He+        all                     1  COMPLETED      0:0
105.batch         batch                                1  COMPLETED      0:0
```



