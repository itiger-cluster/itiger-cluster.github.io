---
layout: post
title: Submitting Jobs (Slurm)
date: 2024-03-28 15:00:00
description: how to submit jobs
tags: metadata
categories: sample-posts external-services
---

Running jobs on HPC requires submitting your jobs to the job scheduler as
either Batch (through a submission script) or Interactively. The scheduler will
find the resources you requested and will execute your job on those resource
when they become available.

## Minimum Requirements for Slurm Scripts:

- Partition ('-partition', 'p')
- CPUs, tasks, or nodes: 'cpus-per-task' or '-c' for multiple threads/cores per node/task (pthreads/OpenMP): '-ntasks' or '-n' for multiple message passing tasks (MPI); '-nodes' or '-N' for multiple nodes (MPI)
- Time ('-time' or '-t')
- Memory ('mem-per-cpu' for memory per CPU core; '-mem' for memory per node)
<strong>Useful options</strong>
- Job name ('-job-name' or '-J') for identification
- Output and error ('-output' or '-o' and '-error' or '-e') to redirect script standard output and error ('stdout' and 'stderr')
- Generic RESource (-gres') used for gpus, licenses, and interconnects

Here's an example of how your bash script should look like:

<div class="row mt-3">
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/bash_example.png" class="img-fluid rounded z-depth-1" zoomable=true %} 
    </div>
</div>

