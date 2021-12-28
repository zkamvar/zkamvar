---
title: Installing the ABySS
author: Zhian N. Kamvar
date: '2017-10-05'
slug: installing-the-abyss
categories:
  - science
  - example
tags:
  - ABySS
  - command line
  - HPC
---

A couple of weeks ago, I wanted to explore assembling 55 genomes of *Sclerotinia sclerotiorum* to check for structural rearrangements that [could be caused by sub-lethal fungicide exposure](https://apsnet.confex.com/apsnet/2017/meetingapp.cgi/Paper/6023). 
I figured the best way to do this was to assemble these genomes *de novo* as opposed to aligning them to a reference.
This brought me to installing the [ABySS] *de novo* assembler on the [HCC Tusker Cluster]... and I can happily report, that after half a day of [command line bullshittery][^1], I had ABySS up and running 👍. 

> Note: this blogpost will be boring. Stop reading now.

Because of the idiosyncrasies of the cluster, I figured I would write a post about this for anyone who finds themselves in a similar situation. 
What do I mean by idiosyncrasies? 
For PATH management, the cluster uses [module management](http://modules.sourceforge.net/).
This means that, in order to use a specific piece of software, I have to load its module first (and all the modules it depends on). 

Moreover, the cluster itself is separated into two main filesystems (both run BASH), `/home` and `/work`. 
The `/home` filesystem is alotted 25GB per user and is meant to store small data sets and programs, but the caveat is that you can only write to this filesystem through the login node. 
The `/work` filesystem is alotted 50TB per user and is meant for temporary storage of data from jobs that are sent to the worker nodes. 

Because it's bad form to compile software on the login node, I wrote submission scripts for compilation in my `/work` directory and then manually copied the output to the `/home` directory.

What goes in the ABySS
-----------------------

Unlike the HCC documentation, the [ABySS documentation] is fairly straightforward and easy to understand. 
One of the first things they show is how to install ABySS to a specific directory:




```bash
./configure --prefix=/path/to/abyss
make
sudo make install
```


Then they explain the software needed by ABySS and give examples of how to tell the compiler where the software is if it's not installed in the expected place on your machine:

 | Program         | exists on HCC        |
 | --------------- | --------------------:|
 | `gcc (>= 4.2)`  | ✔️ |
 | `boost`         | ✔️ |
 | `openmp`        | ✔️ |
 | [`sparsehash`](https://github.com/sparsehash/sparsehash) | ❌ |
 
In this case, we didn't have `sparsehash`, but that's OK because we can simply download and install [sparsehash](https://github.com/sparsehash/sparsehash) and then use the following in our ABySS configuration step:


```bash
./configure CPPFLAGS=-I/path/to/local/include
```

Compiling
---------

So my steps were to:

1. Unzip and compile sparsehash with `--prefix=/work/sparsehash`
2. move the results to `/home/local/`
3. Unzip and compile ABySS with `--prefix=/work/abyss CPPFLAGS=-I/home/local/include`
4. move abyss to `/home/bin`

Simple, right? Now all I have to do is write the scripts to compile these and then move them to their proper places. 
Well, on my first try, I compiled everything and moved them to their correct places, but ended up with an error when abyss tried to run `ABYSS-P`, which prompted me to write myself the [following note](https://github.com/zkamvar/read-processing/blob/a1c151952e95f5b4f0c4c3d919c24b5868086170/scripts/make-abyss-assembly.sh#L41-L45):


```bash
# NOTE TO FUTURE ZHIAN:
#
# It appears as if we need to have ABYSS-P installed for the parallel version
# The way we can do that... I think... is by running the installation process
# of abyss on the cluster using openmp
```

I wrote this note to myself because I had foolishsly tried to run the installation by logging in to a work node interactively because I didn't want to write 6 extra SLURM lines 😒. 
Let this be a lesson to you readers, write a script even if it's only a few lines 👍.
The scripts that finally worked were:

> Note, I'm using the environment variables `${WORK}` and `${HSH}` to mean `/work` and `/home/shared`, respectively.


#### build-sparsehash.sh


```bash
#!/usr/bin/env bash
#SBATCH --job-name=SPARSEHASH-BUILD
#SBATCH --time=04:00:00
#SBATCH --output=compilation-jobs/SPARSEHASH.out
#SBATCH --error=compilation-jobs/SPARSEHASH.err
#SBATCH --mem=4gb
#SBATCH --cpus-per-task=1

module load compiler/gcc/6.1 openmpi/2.1

mkdir -p ${WORK}/compilation-jobs
tar -xzvf ${HSH}/tarballs/sparsehash-2.0.3.tar.gz -C ${WORK}
cd sparsehash-sparsehash-2.0.3/
./configure --prefix=${WORK}/sparsehash
make
make install
```

#### build-abyss.sh


```bash
#!/usr/bin/env bash
#SBATCH --job-name=ABYSS-BUILD
#SBATCH --time=04:00:00
#SBATCH --output=compilation-jobs/ABYSS.out
#SBATCH --error=compilation-jobs/ABYSS.err
#SBATCH --mem=4gb
#SBATCH --cpus-per-task=1

module load compiler/gcc/6.1 openmpi/2.1 boost/1.63

#  Now that we have sparehash compiled, we need to compile ABySS in a similar
#  fashion. Again, I normally use an interactive terminal for this.

tar -xzvf ${HSH}/tarballs/abyss-2.0.2.tar.gz -C ${WORK}
cd abyss-2.0.2/
./configure --prefix=${WORK}/abyss CPPFLAGS=-I${HSH}/local/include
make
make install
make check
```


I say the scripts that *finally* worked because I forgot to include boost in the modules originally, which [resulted in only some of the ABySS programs being built](https://github.com/bcgsc/abyss/issues/166) 🤦.
When I looked at the options I had for boost, I naturally wanted to go with the most recent version, 1.64.
When I added boost/1.64 to the module line, the compilation never got off the ground because I got the following error 💢:

```
Lmod has detected the following error:  These module(s) exist but cannot be loaded as requested: "boost/1.64"
   Try: "module spider boost/1.64" to see how to load the module(s).
```

I found out that boost 1.64 depended on gcc 7.1, so instead of reconfiguring sparsehash to be built with gcc 7.1, I decided to downgrade boost to version 6.1. 

-----

I should say that, I never expect any bioinformatics software to be straightforward to install.
I'm really glad that the authors of ABySS had the detailed installation instructions for those who may not be familiar with all the different flags in the `./configure` step. 
If this post helps at least one person (including my future self), I'll be happy.

[HCC Tusker Cluster]: https://hcc-docs.unl.edu/display/HCCDOC/HCC+Documentation
[ABySS]: https://github.com/bcgsc/abyss#readme
[ABySS documentation]: https://github.com/bcgsc/abyss#compiling-abyss-from-source
[command line bullshittery]: http://pgbovine.net/cmdline-bs-example.htm
[^1]: okay, my example isn't nearly as bad, but this still took me in a few unexpected directions.
