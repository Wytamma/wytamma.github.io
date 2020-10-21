---
title: "Running RStan in a singularity container on the HPC "
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

Back to the task at hand, running a RStan model. Exit out of the `Singularity>` prompt by typing `exit`. Make a `rstan.R` file and populate it with the following RStan model code (taken from this [blog post](https://baconzhou.github.io/post/r-stan-example/)). 

```R
library(rstan)

# Set the number of cores to use
options(mc.cores = 4)

linkage_code <- '
data {
  int<lower=0> y[4];
}

parameters {
  real<lower=0, upper=1> theta;
}

transformed parameters {
  vector[4] ytheta;
  ytheta[1] = 0.5+theta/4;
  ytheta[2] = 0.25-theta/4;
  ytheta[3] = 0.25-theta/4;
  ytheta[4] = theta/4;
}

model {
  y ~ multinomial(ytheta);
}
'

y <- c(125, 18, 20, 34)
dat <- list()
dat$y <- y

model_obj <- stan_model(model_code = linkage_code, model_name = "GeneticLinkage")

fit <- sampling(model_obj, data = dat, iter = 1000, chains = 4) 

print(fit)
```

The singularity `exec` command is used to run a command within a container from outside the container. The `exec` command stands for execute e.g. execute the following command within the specified container. The bash command below tells singularity to execute the command `Rscript rstan.R` inside the `rstan_latest.sif` container.

```bash
singularity exec rstan_latest.sif Rscript rstan.R
```

After running the command above the model should compile then run and produce the following output.

```
Inference for Stan model: GeneticLinkage.
4 chains, each with iter=1000; warmup=500; thin=1; 
post-warmup draws per chain=500, total post-warmup draws=2000.

             mean se_mean   sd    2.5%     25%     50%     75%   97.5% n_eff
theta        0.62    0.00 0.05    0.52    0.59    0.62    0.65    0.71   794
ytheta[1]    0.66    0.00 0.01    0.63    0.65    0.66    0.66    0.68   794
ytheta[2]    0.09    0.00 0.01    0.07    0.09    0.09    0.10    0.12   794
ytheta[3]    0.09    0.00 0.01    0.07    0.09    0.09    0.10    0.12   794
ytheta[4]    0.16    0.00 0.01    0.13    0.15    0.16    0.16    0.18   794
lp__      -207.64    0.02 0.66 -209.58 -207.81 -207.37 -207.21 -207.17   872
```

## Wrapping up

Singularity can be used to install isolated application containers on the HPC. You can find other container images on [duckerhub](https://hub.docker.com/). 

Singularity commands to remember inculde:

- singularity pull docker://`repository_name`  # download a container from Dockerhub
- singularity shell `container_image`.sif  # interactive shell inside conatier
- singularity exec `container_image`.sif `command`  # run a command in a container 

Check out the [Singularity introduction](https://sylabs.io/guides/3.6/user-guide/introduction.html) for more information on using containers in your research. 