---
title: "RStudio on the HPC"
date: 2020-06-25T00:00:00-00:00
excerpt_separator: "<!--more-->"
published: false
categories:
  - blog
tags:
  - R
  - HPC
---

# Getting Rstudio to work on the HPC
## x11 

The traditional way of running GUI application on a remote server (like the HPC) is via X11 forwarding. X11 is... the downside to x11 forwarding is it is slow. 


To run rstudio on the HPC using X11 forwarding do the follow:

```bash
$ brew cask install xquartz # install xquartz using homebrew (mac os)
$ xquartz # don't stop this process
```
In a new terminal window
```bash
$ export DISPLAY=:0
$ ssh -Y USERNAME@zodiac.hpc.jcu.edu.au  # -Y for mac os x
$ mkdir ~/.config/RStudio
$ touch ~/.config/RStudio/desktop.ini
$ echo "desktop.renderingEngine=software" > ~/.config/RStudio/desktop.ini
$ module load R rstudio 
$ rstudio
```

## portforwarding and singularity (rstudio server)

image: https://www.paraview.org/Wiki/Reverse_connection_and_port_forwarding

```bash
$ ssh USERNAME@zodiac.hpc.jcu.edu.au  
$ singularity pull --name singularity-rstudio.simg shub://nickjer/singularity-rstudio  
$ qsub -I -l nodes=1:ppn=2 -l pmem=8gb -l walltime=2:00:00  
$ hostname # remember hostname <HOSTNAME>
$ RSTUDIO_PASSWORD="password" singularity run -B /etc,/opt --auth-none 0 --auth-pam-helper rstudio_auth  
$ PASSWORD="password" singularity exec -B home_rstudio:/home/rstudio -B home_rstudio:$HOME ggtree_2.0.4.sif     rserver --www-port 8787 --www-address 0.0.0.0 --auth-none=0 --auth-pam-helper-path=pam-helper
```

```bash
singularity pull docker://rocker/tidyverse:3.6.3
singularity exec tidyverse_3.6.3.sif rserver --www-address=127.0.0.1 # no password
```

```bash
# example job script https://www.rocker-project.org/use/singularity/ 
```

PASSWORD="password" singularity exec rstudio_latest.sif rserver --www-port 8787 --www-address 0.0.0.0 --auth-none=0 --auth-pam-helper-path=pam-helper

```bash
ssh -L 8787:<HOSTNAME>:8787 USERNAME@zodiac.hpc.jcu.edu.au  
```


ssh to host for PBS commands
https://groups.google.com/a/lbl.gov/forum/#!topic/singularity/syLcsIWWzdo


need to bind the required directories for qsub amd module   https://sylabs.io/guides/3.1/user-guide/bind_paths_and_mounts.html

source /etc/profile.d/modules.sh

# 
https://github.com/hpcng/singularity/issues/729

template https://bioinformaticsworkbook.org/Appendix/HPC/Containers/creatingContainers.html#gsc.tab=0

use [future.batchtools](https://github.com/HenrikBengtsson/future.batchtools) to harness the HPC

explane infix operator

https://www.datamentor.io/r-programming/infix-operator/

http://www.aroma-project.org/share/presentations/BengtssonH_20170314-BARUG/BengtssonH_20170314-A_Future_for_R,BARUG.pdf
https://yihui.org/knitr/hooks/
https://ulriklyngs.com/post/2019/02/03/how-to-create-new-chunk-options-in-r-markdown/
https://github.com/ulyngs/personal-website/blob/master/content/post/2019-02-01-how-to-create-chunk-options.Rmd
https://www.rocker-project.org/use/singularity/
https://pawseysc.github.io/containers-bioinformatics-workshop/4.graphical/index.html


```bash
BootStrap: docker
From: rocker/tidyverse:3.6.3

# https://sylabs.io/guides/3.5/user-guide/definition_files.html

%environment
    # add env vars

%post
    # install packages 
    apt-get update -qq && apt-get -y --no-install-recommends install \
    openssh-server

    # e.g. install ggtree
    R -e 'BiocManager::install("ggtree")'
    
    # add host machine commands using ssh
    # https://groups.google.com/a/lbl.gov/forum/#!topic/singularity/syLcsIWWzdo
    
    BIN_DIR=/usr/local/bin
    
    function add_host_command {
      host_exe=$1
      echo \
        '#!/bin/bash
        ssh $VERBOSE_FLAG ${USER}@${HOSTNAME} ' " $host_exe " '$@' >> ${BIN_DIR}/${host_exe}
    
      chmod +x ${BIN_DIR}/${host_exe}
    }

    # add PBS commands
    add_host_command qsub
    add_host_command qstat
    add_host_command qdel

%labels
    Author Wytamma Wirth
    Version v0.0.1
```