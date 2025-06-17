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

I've written a few different [posts](https://blog.wytamma.com/blog/hpc-rstudio/) and [tutorials](https://blog.wytamma.com/remote-computing-bioinfo-clinic/#rstudio-server) about how to run RStudio on an HPC using RStudio Server. In this post, I'll detail the process specifically for the Unimelb Spartan HPC.

Once you have RStudio running on the Unimelb HPC, you can perform interactive analysis with Spartan's vast compute resources.

## Containers

Container platforms (Docker, Apptainer—previously known as Singularity, etc.) allow you to create and run containers that package up software in a way that is portable and reproducible.

The Unimelb Spartan HPC uses [Apptainer](https://dashboard.hpc.unimelb.edu.au/software/containers) to run containers. This lets you run RStudio in a container on the compute nodes while maintaining the security of the host system.

## Setup (once)

1. SSH into the login node (hint: use the VS Code Remote extension) and create a containers directory:


    ```bash
    mkdir -p $HOME/containers/rstudio/
    ```

2. Download the latest tidyverse container in Singularity image format (`.sif`) to your new directory:

    ```bash
    module purge
    module load GCCcore/11.3.0
    module load Apptainer/1.1.8

    singularity pull \
      $HOME/containers/rstudio/tidyverse_latest.sif \
      docker://rocker/tidyverse:latest
    ```

3. Create the RStudio “job” script and make it executable:

    ```bash
    wget -O $HOME/containers/rstudio/rstudio.job \
      https://gist.githubusercontent.com/Wytamma/4d5a8f763aa602deaee0bfbd64d1a3ae/raw/e08527234b5d13c3a6bf65c7f1c3aa72612d36ce/rstudio.spartan.job

    chmod u+x $HOME/containers/rstudio/rstudio.job
    ```

4. Download the submission wrapper into your \$PATH (e.g. `\$HOME/.local/bin/rstudio`) and make it executable:

    ```bash
    wget -O $HOME/.local/bin/rstudio \
      https://gist.githubusercontent.com/Wytamma/4d5a8f763aa602deaee0bfbd64d1a3ae/raw/3e996c64b79c864b8e11984b8e01c053f7303012/rstudio.spartan.submit

    chmod u+x $HOME/.local/bin/rstudio
    ```

You should now be able to run:

```bash
rstudio --help
```

## Run RStudio

Submit a job to start RStudio on a compute node:

```bash
rstudio
```

By default, it requests 4 cores and 16 GB of RAM. To customize:

```bash
rstudio --cpus 8 --mem 32G
```

Once the job starts, you’ll see instructions in the job log file named `rstudio.<job_id>.out`. It will include something like:

```bash
1. SSH tunnel from your workstation (after logging out of Spartan) using:

   ssh -N -L 8787:spartan-bm004:45737 wytamma@spartan.hpc.unimelb.edu.au

   Then point your browser to http://localhost:8787

2. Log in to RStudio Server with:

   user: wytamma  
   password: 5TtKzC06G4GCAjvgjkNP

When done using RStudio Server, terminate the job by:

1. Exit the RStudio Session ("power" button in the top right corner of the RStudio window)
2. Issue the following command on the login node:

      scancel -f 8936310
```

The key part is the port forwarding command i.e. the `-L 8787:spartan-bm004:45737` part of the `ssh` command. This will create a tunnel from your local machine to the compute node running RStudio. You can then access RStudio by pointing your web browser to http://localhost:8787.

## Conclusion

In this post, I’ve detailed how to run RStudio on the Unimelb Spartan HPC. This setup lets you perform interactive analysis with large compute resources. The process is similar on other HPCs, with just a few site-specific tweaks. If you have any questions or comments, please feel free to reach out.
