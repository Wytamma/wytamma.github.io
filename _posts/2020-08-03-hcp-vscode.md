---
title: "Using VScode to access the HPC"
date: 2020-08-03T00:00:00-00:00
excerpt_separator: "<!--more-->"
header:
  image: assets/images/remote.png
categories:
  - blog
tags:
  - workflow
  - hpc
---

One of the biggest hurdles to using a high performance computer (HPC) is the user interface. Researchers have to `ssh` into the HPC via the command line. This command-line-only interface can be very different from the day-to-day computer experience of researchers. 

To ease the transition to remote computing, I suggest that researchers use an integrated development environment (IDE) to access the HPC.

An IDE is basically a file explore, a text editor, and terminal all in one. There are many types of IDEs, the IDE researchers would be most familiar with is [RStudio](https://blog.wytamma.com/blog/hpc-rstudio/). While RStudio is an excellent IDE for R programming it is specific and thus not the ideal solution for everyone. A more general IDE is Visual Studio Code (VScode).

> Worldwide, Visual Studio is the most popular IDE, Visual Studio Code grew the most in the last 5 years... <cite><a href="https://pypl.github.io/IDE.html"> Top IDE index</a></cite>

[VScode](https://code.visualstudio.com/) is a fully featured IDE produced by Microsoft. VScode is open source and quickly become the default for IDEs. Check out the [Github repo](https://github.com/microsoft/vscode).

## Use VScode to access the HPC

[Download VScode](https://code.visualstudio.com/Download) and start to get a feel for how it works on you local machine. There's some very good [docs](https://code.visualstudio.com/docs/getstarted/introvideos) on how to get started. 

Because it is open source VScode is easily extendable. VScode has an [extensive library of extensions](https://code.visualstudio.com/docs/editor/extension-gallery). One extension of note is [Remote - SSH](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh), this is a core extension developed by Microsoft. The *Remote - SSH* extension lets you connect VScode to a remote server (like the HPC) via `ssh`.

![Remote](/assets/images/remote.png)

Use the Extensions tab to install *Remote - SSH*. Once it's installed a new tab (*Remote Explorer*) appears on the side panel. Press the plus (+) in the Remote Explorer tab to add a new `ssh` target for the HPC.

![ssh-command](/assets/images/ssh-target.png)
*Only include `-p 8822` if you are off campus.*

After you enter your SSH connection command enter your password and connect to the HPC (this will take a little while the first time you connect). In the *Explorer* panel you can open folders and files that are on the HPC. Copying files to the HPC or back to you local machine is as easy as dragging and dropping them into the *Explorer* panel. If you open a new terminal (control+shift+`) you'll open a bash terminal on the HPC. 

![vscode-hpc](/assets/images/vscode-hpc.png)

When you're finished you can simply save and close VScode. Any changes will be saved on the HPC. To reconnect to the HPC, open VScode and go to the Remote *Explorer* panel click on the folder icon next to `zodiac.hpc.jcu.edu.au`.

![ssh-readme](/assets/images/ssh-readme.gif)

For more information check out the Remote - SSH docs or [this helpful blog](https://code.visualstudio.com/blogs/2019/07/25/remote-ssh). 
