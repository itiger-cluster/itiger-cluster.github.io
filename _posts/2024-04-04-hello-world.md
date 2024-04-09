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

Next, we must create a basic BASH file to submit to the HPC. The BASH file acts as sort of a set of instructions for the HPC, including what resources the HPC will provide, the length of the job we shall submit, the modules we will need for the job (in this case, python 3.8.7) and the file we will run ( our python file ).

<br/>
<br/>
After we include the right instructions into our BASH file, we are now able to upload/add it to our preferred directory of the HPC and use the command `sbatch` to begin the job. Once the job has submitted and finished, you should recieve an output file within your directory--this contains the "hello world" output you'd typically see on your terminal.

