---
title: "pyoinformatics 🐍"
date: 2020-06-25T00:00:00-00:00
excerpt_separator: "<!--more-->"
categories:
  - project
tags:
  - package
  - CI/CD
  - bioinformatics
  - rosalind
---
Introducing **pyoinformatics**, a simple bioinformatics package.

`pip install pyoinformatics`

I'm working on the bioinformatics problems on [rosalind.info](http://rosalind.info/problems/list-view/). Some of the code I was using to solve these problems became a bit repetitive, so I abstracted the repetitive and useful code into a python package. 

My design goal was to make everything* a simple sequence object. Most of the bioinformatics I've done as part of the Rosalind problem is just string manipulation, so that's what the primary role of the package is.

*almost everything, some small results, i.e. nucleotides at a position are returned as strings. 

# Seq class 

Most of the work happens in a Seq class. This class takes the nucleotide string and an identifier as initialising variables and implements all of the methods for sequence manipulation. 

```python
seq = Seq('ATG', 'test sequence')
```

Because almost every method returns a new Seq object, methods can be chained. 

```python
rc_fasta_sequence = seq.reverse_complement().to_fasta()
```

I was interested in trying some natural feeling operator overloads. For example: Adding two Seq object concatenates them, subtracting two Seq objects returns the hamming distance between them, and inverting a Seq object returns its reverse complement. The following is valid code using pyoinformatics:

```python
seq1 = Seq('ATG', 'Sequence 1')
seq2 = Seq('ATC', 'Sequence 2')
(seq1 + ~seq1) - seq2 == 4
```

# Deploying

I've never deployed a python package to pypi, but it was surprisingly easy. I set up a [CI/CD script](https://github.com/Wytamma/pyoinformatics/actions?query=workflow%3ACI%2FCD) using Github's actions to automatically test and deploy the code with every push to the repo. Because you can't upload the same version to the package to pypi, I had to auto-increment the version number with every push. It was simple to implement auto-increment version numbers in python's `setup.py` file.

```python
r = requests.get("https://pypi.org/pypi/pyoinformatics/json")
version = r.json()["info"]["version"].split(".")
patch_version = str(int(version[-1]) + 1)
if patch_version == "9":
    patch_version = "0"
    minor_version = str(int(version[1]) + 1)
    version[1] = minor_version
version=".".join(version[:-1] + [patch_version])
```

Be sure to check out pyoinfomatics in detail on [Github](https://github.com/Wytamma/pyoinformatics)!
