---
layout: distill
title: Jupyter Notebook on the HPC
description: How to run a jupyter notebook instance on the HPC
tags: user-tutorials
giscus_comments: false
date: 2024-09-17


authors:
  - name: Mayira Sharif
    affiliations:
      name: University of Memphis


bibliography: 2018-12-22-distill.bib

# Optionally, you can add a table of contents to your post.
# NOTES:
#   - make sure that TOC names match the actual section names
#     for hyperlinks within the post to work correctly.
#   - we may want to automate TOC generation in the future using
#     jekyll-toc plugin (https://github.com/toshimaru/jekyll-toc).
toc:
  - name: Introduction
    # if a section has subsections, you can add them as follows:
    # subsections:
    #   - name: Example Child Subsection 1
    #   - name: Example Child Subsection 2
  - name: Accessing your Jupyter Notebook


# Below is an example of injecting additional post-specific styles.
# If you use this post as a template, delete this _styles block.
_styles: >
  .fake-img {
    background: #bbb;
    border: 1px solid rgba(0, 0, 0, 0.1);
    box-shadow: 0 0px 4px rgba(0, 0, 0, 0.1);
    margin-bottom: 12px;
  }
  .fake-img p {
    font-family: monospace;
    color: white;
    text-align: left;
    margin: 12px 0;
    text-align: center;
    font-size: 16px;
  }
---

## Introduction

Jupyter Notebooks are often used in computational research and data science projects. Here's how to set it up on the HPC:

---

<h4>(Recommended) Creating a Python Virtual Environment</h4>

Creating a virtual environment before running a jupyter-notebook instance is recommended for module cleanliness--in addition, it is very simple to create one!

In the terminal, type `python -m venv [your desired directory]. The picture below shows an example directory you could try:

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/create_venv.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

Now, `cd` into your folder. You should see something like this:

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/inside_venv_folder.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

Now, we must install the required dependencies(in this case, jupyter). In the terminal, simply run `pip3 install jupyter` to install the required modules.


Finally, we'll kickstart this jupyter instance through running a `.sh` file with `sbatch`. Create a new file to submit to the cluster, and make sure to include `~/[directory]/bin/activate`, as well as `module load python/[your_version]`.

Now, type in `jupyter notebook` into the file. save it, then submit it with `sbatch`. For context, this is an example:

```
#!/bin/bash
#SBATCH --cpus-per-task=1
#SBATCH --mem=10G
#SBATCH --time=1-00:00:00
#SBATCH --gres=gpu:1
#SBATCH --partition=agpuq

#module load nvhpc/23.11
#module load cuda/12.3
module load python/3.12.1

. ~/pythonGPU/bin/activate

echo "*** Starting Jupyter on: "$(hostname)
jupyter notebook
```

Once running, you should see a new .out file. Open it up, and you should see something like this:


<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/jupyter_notebook_runs.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

## Accessing your Jupyter Notebook

Now that we have the instance running, we need to be able to view it on a browser tab, since if you try to use the link on a normal browser, it will not work, since we need specialized access.

The most convenient way to view your instances is through connecting to the cluster with X2go. To install X2go, simply download the version applicable to your device from [this link]("https://wiki.x2go.org/doku.php"). Once you do that, open up the application, and enter in the required parameters similar to the picture below. This is for the new session we will create. Make SURE to change the bottom parameter from the initial `KDE` to the `XFCE` option.


 
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/x2go.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

> Pro tip: Notice that the host name is **bigblue1.memphis.edu**, and not **bigblue.memphis.edu**? This is because we actually have two login nodes--and they are represented by the corresponding numbers *1* and *2*. If you wish to run both the virtual desktop session AND a terminal session at the same time, check for the last two digits after the @. If they say 01, make the host **bigblue2.memphis.edu**, and vice versa. This is **optional**, not required.

Now, continue by pressing OK. 

You should see something like this, where the box on the right represents the session you want to start. Click on it.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/client2.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

Now, enter in your credentials, and you should see a window open, looking like a virtual desktop.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/folders.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

 On the bottom bar, click on the browser option. There, a web browser should open. From there, paste the jupyter link the  `.out` file gave when we submitted that batch script. From there, you should be able to see your jupyter notebook running!

 > Don't forget to close your jupyter notebook instance when you are done. You can do this by running `scancel [your job id]` in the terminal.

---