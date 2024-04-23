---
layout: post
title: Submitting a Python Job with Hello World
date: 2024-04-04 15:00:00
description: Learn how to submit a job with Python
tags: getting-started
categories: tutorial
featured: false
---

<strong>Let's learn how to submit a job to the HPC!</strong>

<strong>Step 1: Write your "hello world" code:</strong>
<br/>
To start, open up an IDE of your choice and write code that will output "Hello World" into the terminal:


<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/hello_world_img1.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>


Next, upload your file onto the HPC. Depending on your system/OS, you may have to follow different steps to put your file onto the HPC. For that, follow this [tutorial]({% post_url 2015-05-15-images %})

<br/>
<br/>

Next, we must create a basic BASH file to submit to the HPC. The BASH file acts as sort of a set of instructions for the HPC, including what resources the HPC will provide, the length of the job we shall submit, the modules we will need for the job (in this case, python 3.8.7) and the file we will run ( our python file, which in this case is animal.py ).

```
#!/bin/bash
#SBATCH --cpus-per-task=1
#SBATCH --partition=acomputeq
#SBATCH --time=0-0:1:2
#SBATCH --job-name=gpurun
#SBATCH --mail-user=username@memphis.edu
#SBATCH --output=output-%j.out
#SBATCH --error=error-%j.err
#SBATCH --mem=8GB



#################################################
# [SOMETHING]                                   #
#-----------------------------------------------#
# Please replace anything in [] brackes with    #
# what you want.                                #
# Example:                                      #
# #SBATCH --partition=[PARTITION/QUEUE]         #
# Becomes:                                      #
# #SBATCH --partition=computeq                  #
#################################################



#################################################
# --partition=[PARTITION/QUEUE]                 #
#-----------------------------------------------#
# For this script we are assuming:              #
#   gpuq: 40 cores, 192 GB mem, 2 v100 GPUs     #
#################################################
# --ntasks=[NTASKS] and --nodes=[NNODES]        #
#-----------------------------------------------#
# Number of threads and nodes needed per job.   #
# Note that there are only 2 GPUs available per #
# node. Multiple programs and users may use each#
# card at one time, but you may run into        #
# performance issues. To mitigate this, use more#
# --ntasks to reduce overlap.                   #
#################################################

# Go to submission directory
cd $SLURM_SUBMIT_DIR

#################################################
# modules                                       #
#-----------------------------------------------#
# Any modules you need can be found with        #
# 'module avail'. If you compile something with #
# a particular compiler using a module, you     #
# probably want to call that module here. You   #
# might need one of the cuda modules:           #
#   cuda92/toolkit: nvcc, libraries, etc...     #
#   cuda92/fft: fast fourier transforms         #
#   cuda92/blas: linear algebra                 #
#   cuda92/profiler: nvidia profiler, debugging #
#   cuda92/nsight: IDE, suggested use is with   #
#  'srun -N1 -n1 -p gpuq --pty --x11 /bin/bash' #
#       for a GUI terminal                      # 
#################################################
module load cuda92/toolkit
module load python/3.8.7
python animal.py


#################################################
# Run your executable here                      #
#################################################
#[EXECUTABLE] [OPTIONS]
```

<br/>
<br/>
After we include the right instructions into our BASH file, we are now able to upload/add it to our home directory of the HPC ([link for how to transfer files to the HPC](https://itiger-cluster.github.io/blog/2024/images/)) and use the command `sbatch` to begin the job. Once the job has submitted and finished, you should recieve an output file within your directory--this contains the "hello world" output you'd typically see on your terminal:

<br/>

<div class="row mt-6">
    <div class="col-sm mt-6 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/helloworld2.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>

<br/>


Open the file in a text editor, and you should see your output!

<br/>


<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/helloworld3.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>

