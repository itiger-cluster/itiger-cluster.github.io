---
layout: post
title: Submitting a Python Job with Hello World
date: 2024-04-04 15:00:00
description: Learn how to submit a job with Python
featured: false
---

<strong>Let's learn how to submit a job to the HPC!</strong>

<br/>
To start, open up an IDE of your choice and save the following code as `hello.py`. It prints "Hello World" to the terminal:


```python
print("Hello World")
```

Next, upload your file onto the HPC. Depending on your system/OS, you may have to follow different steps to put your file onto the HPC. For that, follow this <a href="https://github.com/itiger-cluster/itiger-cluster.github.io/wiki/Transferring-Data-Guide">tutorial</a>.



<br/>
<br/>

Next, we must create a basic BASH file to submit to the HPC. The BASH file acts as sort of a set of instructions for the HPC, including what resources the HPC will provide, the length of the job we shall submit, the modules we will need for the job (in this case, Python 3.12.1) and the file we will run (our Python file, which in this case is `hello.py`). Save it as `hello.sh`.

```bash
#!/bin/bash

#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=1G
#SBATCH --partition=bigTiger
#SBATCH --job-name=basicname

module load python/3.12.1

echo "*** Running Python on: $(hostname)"

python hello.py
```


After we include the right instructions into our BASH file, we are now able to upload/add it to our home directory of the HPC (<a href="https://github.com/itiger-cluster/itiger-cluster.github.io/wiki/Transferring-Data-Guide">see how to transfer files to the HPC</a>) and use the command `sbatch hello.sh` to begin the job. Once the job has submitted and finished, you should receive an output file named `slurm-<jobid>.out` within your directory. It contains the "Hello World" output you'd typically see on your terminal:

```console
$ sbatch hello.sh
Submitted batch job 12345
$ ls
hello.py  hello.sh  slurm-12345.out
$ cat slurm-12345.out
*** Running Python on: <compute node name>
Hello World
```

Open the file in a text editor (or use `cat` as above), and you should see your output!
