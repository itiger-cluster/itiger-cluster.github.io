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

<br/>
To start, open up an IDE of your choice and write code that will output "Hello World" into the terminal:


```
print("Hello World")
```

Next, upload your file onto the HPC. Depending on your system/OS, you may have to follow different steps to put your file onto the HPC. For that, follow this <a href="https://itiger-cluster.github.io//blog/2024/images/">tutorial.</a>



<br/>
<br/>

Next, we must create a basic BASH file to submit to the HPC. The BASH file acts as sort of a set of instructions for the HPC, including what resources the HPC will provide, the length of the job we shall submit, the modules we will need for the job (in this case, python 3.8.7) and the file we will run (our python file, which in this case is animal.py).

```
#!/bin/bash

#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=1G
#SBATCH --partition=acomputeq
#SBATCH --job-name=basicname

echo "*** Running Python on: "$(hostname)

python animal.py
```


After we include the right instructions into our BASH file, we are now able to upload/add it to our home directory of the HPC (<a href="https://itiger-cluster.github.io/blog/2024/images/">link for how to transfer files to the HPC</a> and use the command `sbatch` to begin the job. Once the job has submitted and finished, you should recieve an output file within your directory--this contains the "hello world" output you'd typically see on your terminal: 

<p align="center">
<img src="https://itiger-cluster.github.io/assets/img/helloworld2-1400.webp" />
</p>



Open the file in a text editor, and you should see your output!  


<p align="center">
<img src="https://itiger-cluster.github.io/assets/img/helloworld3-1400.webp">
</p>
