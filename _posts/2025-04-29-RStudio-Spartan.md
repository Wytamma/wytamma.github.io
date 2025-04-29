---
title: "Run RStudio on Unimelb Spartan HPC"
published: true
categories:
  - Blog
tags:
  - RStudio
  - HPC
  - Unimelb
---

I've written a few different [posts](https://blog.wytamma.com/blog/hpc-rstudio/) and [tutorials](https://blog.wytamma.com/remote-computing-bioinfo-clinic/#rstudio-server) about how to run RStudio on the of a HPC using RStudio Server. In this post I'll detail the process specifically for the Unimelb Spartan HPC.

## RStudio on Spartan

This post details how to get RStudio server running on the compute nodes of the Unimelb Spartan HPC. For an old (out of date) post about JCU's HPC see [here](https://blog.wytamma.com/blog/hpc-rstudio/). Once you have RStudio running on the HPC you can perform interactive analysis with large compute resources.

## Containers 

Container platforms (docker, singularity, etc.) allow you to create and run containers that package up pieces of software in a way that is portable and reproducible.

The Unimelb Spartan HPC uses [Apptainer](https://dashboard.hpc.unimelb.edu.au/software/containers) previously Singularity to run containers. This allows you to run RStudio in a container on the compute nodes while maintaining the security of the host system.


## Setup (once)

Start by `ssh`ing to the remote server (hint use VScode remote extension) and create a containers directory.

```bash
mkdir -p $HOME/containers/rstudio/
```

Download the latest tidyverse container and save it in singularity image format (`.sif`) at $HOME/containers/rstudio/tidyverse_latest.sif:
```bash
singularity pull $HOME/containers/rstudio/tidyverse_latest.sif docker://rocker/tidyverse:latest
```

Create a rstudio script and save is in you $PATH e.g. `$HOME/.local/bin/rstudio`. This is the script that will run the RStudio server. The script should look like this:

You can download the script from [here](https://gist.github.com/Wytamma/4d5a8f763aa602deaee0bfbd64d1a3ae). Or use the following command to create the script:

```bash
wget -O $HOME/containers/rstudio/rstudio.job https://gist.githubusercontent.com/Wytamma/4d5a8f763aa602deaee0bfbd64d1a3ae/raw/e08527234b5d13c3a6bf65c7f1c3aa72612d36ce/rstudio.spartan.job
```

Create an a submission script to run the RStudio server. This script will be used to submit a job to the HPC. You can use the following command to create the script:

```bash
wget -O $HOME/.local/bin/rstudio https://gist.githubusercontent.com/Wytamma/4d5a8f763aa602deaee0bfbd64d1a3ae/raw/3e996c64b79c864b8e11984b8e01c053f7303012/rstudio.spartan.submit
```


Make the script executable:
```bash
chmod u+x $HOME/.local/bin/rstudio
```

You should now be able to run the script:
```bash
rstudio --help
```

## Run RStudio

To run RStudio you need to submit a job to the HPC. You can do this by running the following command:

```bash
rstudio
```

Use the `--help` flag to see the options available. The default options will run RStudio on the first available compute node with 4 cores and 16GB of RAM. You can change the number of cores and RAM by using the `--cpus` and `--mem` flags.

For example, to run RStudio with 8 cores and 32GB of RAM you can use the following command:

```bash
rstudio --cpus 8 --mem 32G
```

This will submit a job to the HPC and start RStudio on the first available compute node with 8 cores and 32GB of RAM. You can then access RStudio by following the instructions in the in the job log file. The log file will be saved in the current working directory and will be named `rstudio.<job_id>.out`. 

The log file will look something like the following:

```bash

1. SSH tunnel from your workstation (log out of spartan) using the following command:

   ssh -N -L 8787:spartan-bm004:45737 wytamma@spartan.hpc.unimelb.edu.au

   and point your web browser to http://localhost:8787

2. log in to RStudio Server using the following credentials:

   user: wytamma
   password: 5TtKzC06G4GCAjvgjkNP

When done using RStudio Server, terminate the job by:

1. Exit the RStudio Session ("power" button in the top right corner of the RStudio window)
2. Issue the following command on the login node:

      scancel -f 8936310
```

The key command is the port forwarding command i.e. the `-L 8787:spartan-bm004:45737` part of the `ssh` command. This will create a tunnel from your local machine to the compute node running RStudio. You can then access RStudio by pointing your web browser to `http://localhost:8787`.

## Conclusion

In this post I've detailed how to run RStudio on the Unimelb Spartan HPC. This allows you to perform interactive analysis with large compute resources. The process is similar to running RStudio on other HPCs, but there are some differences in the setup and submission process. If you have any questions or comments please feel free to reach out.