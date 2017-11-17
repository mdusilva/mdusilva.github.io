---
layout: post
title: "Mamuto package - an easy distributed map in Python"
author: "Manuel Silva"
categories: journal
tags: [Python,code]
image:
  feature: 
  teaser: 
  credit:
  creditlink:
---

When it comes to parallel computating, Python has some limitations. This is specially true in the case of distributed computing (worker processes distributed across different nodes). Even though is possible to do it using the standard module `multiprocessing` as shown [here](https://eli.thegreenplace.net/2012/01/24/distributed-computing-in-python-with-multiprocessing), and there are many paralleiztion solutions [available](https://wiki.python.org/moin/ParallelProcessing), these solutions are often difficult to setup and configure when all one wants to do is a simple function `map()` or `reduce()`.

For this reason I decided to create my own solution called `mamuto` implementing a `map()` method requiring minimal configuration. Using the package [execnet](http://codespeak.net/execnet/) to provide communication between nodes using ssh, it was a matter of performing the necessary setup of the master/worker processes so it is ready to use. I had the following requirements:

* Apply functions to items in large lists or Numpy arrays using remote or local
processes.
* Send just some or all of the arguments only once to the remote processes. Useful when many calls 
are made, as it avoids sending the same data repeatedly.
* Easily configure your cluster environment using configuration files which can be created by a simple 
function call. It must be easier to configure than one of the easiest solution available, [pyspark](http://spark.apache.org/docs/2.1.0/api/python/pyspark.html).

The package is very easy to use. Say you want to compute a log-likelihood function defined in a module `model.py` like this:

```python
import numpy as np

def like(x, data):
    f = -(data-x)**2.
    e = 2.*0.1**2.
    return np.sum(f/e)
```

And we want to compute in parallel for two lists of arguments `p` and `data` with 1000 elements which are 3D vectors.

```python
import numpy as np
import model
import mamuto

n = 3000
p = np.ones(n)
data = np.random.normal(0,0.1,n)

p = p.reshape((n,3))
data = data.reshape((n,3))
```

To map the function `like` to these parameters using `mamuto` we first need to create a configuration file. If we want to use 4 cores in our local machine we do:

```python
mamuto.create_config_file(filename="cluster.cfg", hosts={'localhost':4}, workdir="", python="/home/user/anaconda3/envs/env1/bin/python", nice=0)
```

This creates a configuration file called "cluster.cfg" and tells the worker processes to use the interpreter in anaconda virtual environment: "/home/user/anaconda3/envs/env1/bin/python". We only need to create this file once for each cluster environment.

Then to perform the actual computation we only need 3 lines:

```python
function_mapper = mamuto.Mapper("cluster.cfg", depends=["model"])
function_mapper.add_function(model.like)
args = [p, data]
result = function_mapper.remap(model.like, args)
```

It is also possible to pass arguments (some or all) in advance:

```python
args = [None, data]
function_mapper.add_remote_arguments(model.like, args)
```

Then we pass the rest of the arguments when calling map:

```python
args = [p, None]
result = function_mapper.remap(model.like, args)
```

This is usefull to reduce communication lag when arguments are very large iterables and are the same between different calls.

We can compare to what we need to do if we want to do the same thing using [pyspark](http://spark.apache.org/docs/2.1.0/api/python/pyspark.html):

```python
import pyspark

sc = pyspark.SparkContext(master='local[*]')

args = ([x,y] for x, y in zip(p0, data))
rdd = sc.parallelize(args)
result = rdd.map(lambda x: model.like(x[0],x[1])).collect()
```

It takes roughly the same ammount of work as using `mamuto`, However this is for the simple case of running only local workers. TO configure [pyspark](http://spark.apache.org/docs/2.1.0/api/python/pyspark.html) to use a cluster takes much more work, whereas using `mamuto` we don't need to chenge anythin appart from the list of hosts written on the configuration file, it is only one call to `create_config_file` just as in the local configuration case.

It also interesting to note that `mamuto` is much faster than [pyspark](http://spark.apache.org/docs/2.1.0/api/python/pyspark.html) when running on a local machine for small jobs. In our example with arguments with a dimension of 1000 `mamuto` takes 0.016 sec whereas [pyspark](http://spark.apache.org/docs/2.1.0/api/python/pyspark.html) is much slower taking 0.11 sec. For larger jobs, for example for `n=100000`, [pyspark](http://spark.apache.org/docs/2.1.0/api/python/pyspark.html) becomes faster, taking 0.54 sec and `mamuto` taking 2.07 sec.