---
layout: post
title: "Dictionary Learning with SPAMS - I Installation"
author: "Manuel Silva"
categories: journal
tags: [Data Science,Linux, SPAMS]
image:
  feature: 
  teaser: 
  credit:
  creditlink:
---

At the moment I have been working with a library called [SPAMS](http://spams-devel.gforge.inria.fr/), an "optimization toolbox for solving various sparse estimation problems". It is a nice tool implementing several methods of dimensionality reduction, including Dictionary Learning.

I am using the Python interface provided in the [website](http://spams-devel.gforge.inria.fr/) and because I found the installation was not trivial I thought it would be a good idea to share my experience in case someone else finds the same pitfalls.

My system has Linux Ubuntu 16.04 and an Anaconda 4.4 Python installation. I installed the [SPAMS](http://spams-devel.gforge.inria.fr/) package in a  fresh 3.6 Python virtual environment.

After downloading and unpacking the package, the first step was to compile the c++ library using ```python setup.py build```. Initially I had a problem with the compiler not finding the BLAS/LAPACK libraries. I solved this initally by creating symbolic links in ```/usr/bin``` but eventually I decided to use the same MKL libraries used by Anaconda to compile ```Numpy``` . In order to that I edited the file ```setupy.py``` and replaced the line ```libs = ['stdc++', 'blas', 'lapack', 'gomp']``` by:

```python
libs = ['mkl_intel_lp64', 'mkl_intel_thread', 'mkl_core', 'iomp5', 'pthread']+['stdc++', 'gomp']
```

After this I installed the package with ```python setup.py install```. However when I tried to ```import spams``` in the Python interpreter I got the error:

```python
ImportError: /path/to/Work/spams/_spams_wrap.cpython-36m-x86_64-linux-gnu.so: undefined symbol: _ZTTNSt7__cxx1119basic_istringstreamIcSt11char_traitsIcESaIcEEE
```

To solve this problem I had to update the package ```libgcc``` to a new version:

```python
conda install libgcc
```

And this is it, I was now able to use [SPAMS](http://spams-devel.gforge.inria.fr/).