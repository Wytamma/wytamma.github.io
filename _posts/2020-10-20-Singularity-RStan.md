---
title: "Containerised applications on the HPC (RStan example)"
date: 2020-10-10T00:00:00-00:00
excerpt_separator: "<!--more-->"
header:
  image: assets/images/rstan.png
categories:
  - blog
tags:
  - R
  - hpc
  - workflow
  - singularity
---

[Singularity](https://sylabs.io/docs/) is a container platform. It allows you to create and run containers that package up pieces of software in a way that is portable and reproducible. Singularity is the preferred container platform for HPC clusters as each container is only a single file and users don't need root access to run the containers.<!--more--> This post shows you how to use Singularity on the JCU HPC (where it is already installed), however, if you want to use it on a different machine (i.e. your personal computer) simply follow the [installaltion instructions](https://sylabs.io/guides/3.6/user-guide/quick_start.html#quick-installation-steps) for Singularity.

## Creating a singularity image from a docker image in docker hub:

To start, `ssh` to your `HOME` directory on the HPC. I wrote a [guide](https://blog.wytamma.com/blog/hcp-vscode/) for using VScode to make this process easy.

Singularity can build containers from [dockerhub](https://hub.docker.com/) (a different container platform). The `bash` command below pulls an RStan docker container from Dockerhub and converts it to a singularity image format (.sif) container. The Dockerfile (i.e. recipe file) that generates this container can be found [here](https://hub.docker.com/r/jrnold/rstan/dockerfile).

```bash
singularity pull docker://jrnold/rstan
```

Once the download completes you will have a singularity container file called  `rstan_latest.sif`.

## Running a Stan model in the Singularity RStan container

Inside the RStan container is a version of `R` with RStan already installed. You can access a shell prompt (i.e. bash) inside a specified container (e.g. the `rstan_latest.sif` container we pulled from Dockerhub) with the bash command below.

```bash
singularity shell rstan_latest.sif
```

This will give you a `Singularity>` prompt "inside" the container. Try running `R` i.e. the command to get an interactive `R` prompt. Notice you don't need to run `load module R` as `R` is already installed inside this container. Inside the container is essentially a different computer - a container is like a lightweight virtual machine that shares components of the host operating system. For further reading, I suggest you look at the [Singularity introduction](https://sylabs.io/guides/3.6/user-guide/introduction.html).

Back to the task at hand, running a RStan model. Exit out of the `Singularity>` prompt by typing `exit`. Make a `rstan.R` file and populate it with the following RStan linear regression example code (taken from this [blog post](https://michaeldewittjr.com/resources/stan_linear_regression.html)). 

```R
library(rstan)

# set number of cores to use
options(mc.cores = 4)

# save models
rstan_options(auto_write = TRUE)

x <- rnorm(40, 10, 5)
noise <- rnorm(40,0,1)
y <- x*.5 + noise

data <- list( x= x, y = y, N = length(x))

stan_linear_regression <- '
data {
  int<lower=0> N;
  vector[N] x;
  vector[N] y;
}
parameters {
  real alpha;
  real beta;
  real<lower=0> sigma;
}
model {
  y ~ normal(alpha + beta * x, sigma);
}
'

linear_regression <- stan_model(model_code = stan_linear_regression, model_name = "LinearRegression")

fit1 <- sampling(linear_regression, data = data, iter = 1000)

print(fit1)
```

The singularity `exec` command is used to run a command within a container from outside the container. The `exec` command stands for execute e.g. execute the following command within the specified container. The bash command below tells singularity to execute the command `Rscript rstan.R` inside the `rstan_latest.sif` container.

```bash
singularity exec rstan_latest.sif Rscript rstan.R
```

After running the command above the model should compile then run and produce the following output. Note the beta (slope) parameter estimate is close to the real value of `0.5`.

```
Inference for Stan model: LinearRegression.
4 chains, each with iter=1000; warmup=500; thin=1; 
post-warmup draws per chain=500, total post-warmup draws=2000.

        mean se_mean   sd   2.5%    25%    50%    75%  97.5% n_eff Rhat
alpha  -0.60    0.01 0.32  -1.23  -0.83  -0.60  -0.39   0.02   861 1.00
beta    0.54    0.00 0.03   0.48   0.52   0.54   0.55   0.59   869 1.00
sigma   0.90    0.00 0.11   0.72   0.83   0.89   0.97   1.16   694 1.01
lp__  -14.97    0.06 1.27 -18.35 -15.55 -14.64 -14.06 -13.49   407 1.01
```

## Submitting to qsub

Finally, to take full advantage of Singularity on the JCU HPC we need to run our code as a job on the compute nodes using `qsub`. 

Create a PBS script called `rstan_PBS.sh` with the following content.

```bash
#!/bin/bash
#PBS -N RStan
#PBS -l walltime=00:30:00
#PBS -l select=1:ncpus=4:mem=8gb

# Move to working dir
cd $PBS_O_WORKDIR

# The bash command below tells singularity to execute the command
# `Rscript rstan.R` inside the `rstan_latest.sif` container
singularity exec rstan_latest.sif Rscript rstan.R
```

We can now submit our script to the queue using the bash command `qsub rstan_PBS.sh`. Once our job completes the output will appear in the current directory named something like `RStan.o1744909`. 

## Wrapping up

Singularity can be used to install isolated application containers on the HPC. You can find other container images on [dockerhub](https://hub.docker.com/). Check out the [Singularity introduction](https://sylabs.io/guides/3.6/user-guide/introduction.html) for more information on using containers in your research. 

### Singularity commands to remember

Download a container from a given URI:  
`singularity pull [pull options...] [output file] <URI>`  
Download a container from Dockerhub:  
`singularity pull docker://<repository_name>`  
Start an interactive shell inside container:  
`singularity shell <container>`  
Run a command within a container:  
`singularity exec [exec options...] <container> <command>`  
Clean the local cache:  
`singularity cache clean`