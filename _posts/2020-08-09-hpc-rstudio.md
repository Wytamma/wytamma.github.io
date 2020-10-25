---
title: "RStudio server on the HPC"
date: 2020-08-09T00:00:00-00:00
excerpt_separator: "<!--more-->"
categories:
  - blog
header:
  image: assets/images/rstudio-server.png
tags:
  - workflow
  - R
  - HPC
  - singularity
---

This post details how to get RStudio server running on the compute nodes of the HPC. This is achieved though a combination of `singularity` and  `ssh` commands. Once you have RStudio running on the HPC you can perform interactive analysis with large compute resources.

## Singularity?

Singularity in a container platform. Container platforms allow you to create and run containers that package up pieces of software in a way that is portable and reproducible. 

You should have some idea of how to use singularity to follow this guide. I have a post about using containerised applications on the HPC [here](https://blog.wytamma.com/blog/Singularity-RStan/). If you're not sure where to start, work though the [singularity introduction](https://sylabs.io/guides/3.6/user-guide/introduction.html).

## Instructions

Clone [this](https://github.com/Wytamma/rstudio-hpc) repo into your home directory on the HPC. The repo contains the `bash` scripts needed to run Rstudio on the HPC.  

```bash
git clone https://github.com/Wytamma/rstudio-hpc.git && cd rstudio-hpc
```

Pull the RStudio container 

```bash
singularity pull library://wytamma/default/rstudio_hpc:latest
```

Submit the `start_rstudio.sh` script to `qsub`

```bash
qsub start_rstudio.sh -o ~/rstudio-hpc/output/
```

Check the end of the log file for instructions on how to connect to the rstudio server

```bash
cat log.txt
```

In a new terminal (e.g. disconnect form the HPC with `exit`) use the `ssh` command from the log file to reconnect to the HPC

```bash
ssh -L 8787:${HOSTNAME}:${PORT} ${USER}@zodiac.hpc.jcu.edu.au -p 8822 # only include -p 8822 if you are off-campus
```

Point your web browser to http://localhost:8787

Log in to RStudio Server using the following credentials:

```bash  
user: ${USER}
password: ${PASSWORD}
```

When done using RStudio Server, terminate the job by:

1. Exit the RStudio Session ('power' button in the top right corner of the RStudio window)
2. Issue the following command on the login node:

```bash
qdel -x ${PBS_JOBID}
```

### Build the RStudio container yourself

You could modify the image (i.e. install other packages) by changing the .def file then rebuilding.

```bash
singularity build --remote rstudio-hpc.def rstudio-hpc.sif
```

## Conclusions

Having Rstudio on the HPC is very useful, however, you should still abide by the rules of the HPC. Don't waste resources, only request the memory/CPUs you need and kill you job once you've finished working.

