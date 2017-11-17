---
layout: post
title: "Installing CUDA 9 on Optimus laptop with Ubuntu"
author: "Manuel Silva"
categories: journal
tags: [Nvidia,CUDA,Linux, Optimus]
image:
  feature: 
  teaser: 
  credit:
  creditlink:
---

I wanted to install CUDA 9 on my Optimus laptop running Linux Ubuntu. Having Optimus means it is possibel to switch between the Nvidia discrete GPU and the integrated Intel one. I had two requirements: use only the Nvidia proprietary drivers (use `nvidia-prime` to manage the GPUs); use the integrated Intel GPU for display and the Nvidida GPU just for computations. My system has Ubuntu 16.04 and the Nvidia GPU is a GeForce 860M.

To install, I first removed all previous CUDA packages and Nvidia drivers:

```bash
sudo apt --purge remove nvidia-384
sudo apt --purge remove nvidia-prime
sudo apt --purge remove nvidia-settings
sudo apt --purge remove nvidia-modprobe
sudo apt --purge remove cuda-*
``` 
Then I simply followed the instructions in the [installation guide](http://docs.nvidia.com/cuda/pdf/CUDA_Installation_Guide_Linux.pdf) for Ubuntu:

```bash
sudo dpkg -i cuda-repo-<distro>_<version>_<architecture>.deb
sudo apt-key add /var/cuda-repo-<version>/7fa2af80.pub
sudo apt-get update
sudo apt-get install cuda
```

Finally perform the post-installation actions as advised in the manual. Added these lines to `.profile`:

```bash
export PATH=/usr/local/cuda-9.0/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-9.0/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```
Finally, an important step not on the manual and it is crucial to allow the use of Nvidia GPU while using the Intel GPU for display. I created a script called `cuda_ready` with the following contents:

```bash
#!/bin/bash
#Make Nvidia drivers available

export LD_LIBRARY_PATH=/usr/lib/nvidia-384:$LD_LIBRARY_PATH
```

Then, when I want to use CUDA in any terminal I only need to do:

```bash
source cuda_ready
```

So when I run `nvidia-smi` I get the the output:

```bash
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 384.90                 Driver Version: 384.90                    |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GTX 860M    Off  | 00000000:01:00.0 Off |                  N/A |
| N/A   59C    P0    N/A /  N/A |      0MiB /  2002MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+

```

Note how `Xorg` and `compiz` processes are not running in the GPU, as it is not being used for display.

However, if I run the command `deviceQuery` found inside the samples directory I get the correct output:

```bash
 CUDA Device Query (Runtime API) version (CUDART static linking)

Detected 1 CUDA Capable device(s)

Device 0: "GeForce GTX 860M"
  CUDA Driver Version / Runtime Version          9.0 / 9.0
  CUDA Capability Major/Minor version number:    5.0
  Total amount of global memory:                 2003 MBytes (2100232192 bytes)
  ( 5) Multiprocessors, (128) CUDA Cores/MP:     640 CUDA Cores

  ...

deviceQuery, CUDA Driver = CUDART, CUDA Driver Version = 9.0, CUDA Runtime Version = 9.0, NumDevs = 1
Result = PASS

```

Also if I run one of the samples, `./0_Simple/vectorAdd/vectorAdd` everything works well:

```bash
[Vector addition of 50000 elements]
Copy input data from the host memory to the CUDA device
CUDA kernel launch with 196 blocks of 256 threads
Copy output data from the CUDA device to the host memory
Test PASSED
Done
```