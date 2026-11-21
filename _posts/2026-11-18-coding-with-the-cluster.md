---
layout: post
title: Coding with the iTiger AI Cluster
date: 2026-11-18 16:00:00
description: login and compute nodes, srun with tmux, SSH ProxyJump, and remote debugging with PyCharm
---

This tutorial is adapted from the *Instructions for Coding with UofM AI Cluster* guide written by **Dr. Haomiao Ni** and **Yanliang Qi** (yqi2@memphis.edu). It walks through a complete day-to-day workflow on the cluster: getting a GPU on a compute node, connecting to it directly from your laptop, and running and debugging your code from **PyCharm**.

If you use iTiger resources in a publication, please follow our [citation]({{ '/citation/' | relative_url }}) and [acknowledgement]({{ '/acknowledgement/' | relative_url }}) guidelines:

1. **Acknowledgment**: state the use of the iTiger Cluster resources in the acknowledgments section of your paper.
2. **Citation**: cite Dr. Huang's paper describing the iTiger Cluster in your references.

#### Prerequisites

You need an iTiger account. If you don't have one yet, follow [Creating an Account]({% post_url 2024-03-26-account %}). Throughout this tutorial, replace `your_username` with your UofM ID.


## How the Cluster Is Organized

The iTiger cluster has one **login node** (`itiger`) and several **compute nodes** (`itigerXX`, where `XX` is a two-digit ID such as `itiger01`).

- **Login node**: manage data, edit code, and set up environments here. Only the login node has the usual Linux tools such as `vim` and `git`.
- **Compute nodes**: run all computation (training, inference, notebooks) here, after requesting one through Slurm.

Each user gets:

- **20 GB** of storage in `/home`
- **200 GB** of storage in `/project`
- Up to **48 hours** (2 days) of continuous use of a compute node per job


## Request a Compute Node

**1. [local machine, optional] Connect to the UofM VPN**

If you are off campus, connect to `vpn.memphis.edu` with Cisco Secure Client first. Skip this step if you are on the UofM network.

<div class="row mt-3">
    <div class="col-sm-6 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/pycharm/vpn.jpg" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

**2. [local machine] Log in to the login node**

```bash
ssh your_username@itiger.memphis.edu
```

**3. [login node] Install conda in `/project`**

Install Anaconda3 or Miniconda under `/project/your_username` rather than `/home`. Your environments will grow as you install more packages, and `/home` is only 20 GB. It is a good habit to keep everything, including code and data, in `/project`. Our [first tutorial]({% post_url 2024-10-08-run-model %}) walks through the Miniconda installation.

```
[your_username@itiger ~]$ cd /project/your_username/
[your_username@itiger your_username]$ ls
miniconda3  code  experiments
```

**4. [login node] Open a background terminal with `screen` or `tmux`**

A `srun` session ends when your SSH connection drops. To keep it alive, start it inside `screen` (or `tmux`) so you can detach and come back later:

```bash
screen -S gpu        # start a new session named "gpu"
screen -ls           # list your sessions
screen -r gpu        # reattach to it later
```

Inside the session:

a. Activate your conda environment:

```bash
conda activate your_env_name
```

b. Request a compute node. The command below asks for 1 GPU, 4 CPU cores, and 24 GB of memory for 48 hours (see the [srun documentation](https://slurm.schedmd.com/srun.html) for each option):

```bash
srun --partition=bigTiger --time=48:00:00 --mem=24G --gres=gpu:1 \
     --cpus-per-task=4 --ntasks=1 --pty bash
```

Once the job starts, the prompt changes to the compute node you were given, e.g. `[your_username@itiger01 ~]$`. You can check the GPU with:

```bash
nvidia-smi
```

```
| NVIDIA-SMI 560.35.03     Driver Version: 560.35.03     CUDA Version: 12.6   |
|   0  NVIDIA H100 80GB HBM3   Off |   00000000:A6:00.0 Off |                0 |
| N/A   30C    P0     70W /  700W |       1MiB /  81559MiB |      0%  Default |
```

c. **[optional, Jupyter only, compute node]** Start a Jupyter server:

```bash
jupyter notebook --no-browser --port 9099
```

d. Detach from the background terminal without killing the job: press `Ctrl + A`, then `D` (for `tmux`: `Ctrl + B`, then `D`).

**5. [optional, Jupyter only, local machine] Forward the Jupyter port**

In a new terminal on your local machine, run the following, replacing `itiger01` with your compute node:

```bash
ssh -t -t your_username@itiger.memphis.edu -L 7077:localhost:8088 ssh itiger01 -L 8088:localhost:9099
```

This chains two tunnels: local `7077` → login node `8088` → compute node `9099`.

**6. [optional, Jupyter only, local machine]** Open `http://localhost:7077` in your browser.

> **Tip**: For a batch-job alternative to this interactive setup, see our [Jupyter Notebook on the HPC]({% post_url 2024-09-17-jupyter-notebook %}) tutorial.


## Set Up SSH Access from Your Local Machine to a Compute Node

With a little SSH configuration you can reach a compute node from your laptop in a single command. This is also what PyCharm needs in the next section.

**1. Generate an SSH key pair on your local machine**

If you don't already have an SSH key:

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

This creates `id_rsa` and `id_rsa.pub` in `~/.ssh/`.

**2. Copy the key to the login node**

```bash
ssh-copy-id your_username@itiger.memphis.edu
```

You can now SSH from your local machine to `itiger` without a password.

> **Windows**: `ssh-copy-id` is not available in PowerShell. Instead, append the contents of your `id_rsa.pub` to `~/.ssh/authorized_keys` on the login node:  
> `type $env:USERPROFILE\.ssh\id_rsa.pub | ssh your_username@itiger.memphis.edu "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"`

**3. Configure SSH to use ProxyJump**

Edit `~/.ssh/config` on your local machine so that `itiger` is used automatically as a jump host. First, add the login node:

```
Host itiger
    HostName itiger.memphis.edu
    User your_username
```

- **macOS / Linux**: add a pattern that matches every compute node:

```
Host itiger0? itiger1?
    HostName %h
    User your_username
    ProxyJump itiger
```

The `?` matches any single character, as supported by OpenSSH.

- **Windows**: if the `?` pattern or `ProxyJump` doesn't work with your SSH client (or with PyCharm), add one entry per compute node you use instead, e.g. for `itiger01`:

```
Host itiger01
    HostName %h
    User your_username
    ProxyCommand ssh -W %h:%p your_username@itiger
```

**4. Connect to the compute node**

While you have a job running on that node (from the previous section), you can connect directly:

```bash
ssh itiger01
```

```
Last login: Tue Jun  3 16:36:58 2025 from 172.16.0.2
(base) [your_username@itiger01 ~]$
```

This uses your SSH key and routes through `itiger` automatically.

**5. Port forwarding (optional)**

```bash
ssh -L 8088:localhost:9099 itiger01
```

This forwards `localhost:8088` on your local machine to `localhost:9099` on `itiger01`, via `itiger`. With this, the two-hop Jupyter command above becomes a single command, and you can open `http://localhost:8088`.


## Code and Debug Remotely with PyCharm

**1. Get PyCharm Professional**

Remote interpreters require PyCharm **Professional**. Students can [apply for a free license](https://www.jetbrains.com/community/education/#students). The screenshots below use PyCharm 2022.2.5, so menus may differ slightly in newer versions. Open your project folder in PyCharm:

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="lazy" path="assets/img/pycharm/pycharm_01.jpg" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

**2. Configure the remote Python interpreter**

This interpreter runs and debugs your code on the compute node. Make sure `ssh itigerXX` works from your terminal (previous section) before continuing.

Open **PyCharm ⇒ Settings ⇒ Project: XXXX ⇒ Python Interpreter ⇒ Add Interpreter ⇒ On SSH**.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="lazy" path="assets/img/pycharm/pycharm_02.jpg" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

Enter the host `itigerXX` and your username, then keep clicking **Next**.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="lazy" path="assets/img/pycharm/pycharm_03.jpg" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

Choose **System Interpreter**, then set:

- **Interpreter**: the Python of a conda environment you created for this project on the cluster, e.g. `/project/your_username/miniconda3/envs/MVOC/bin/python`.
- **Sync folders**: the folder on the cluster that will hold a copy of your local code. We will set up automatic upload to it next.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="lazy" path="assets/img/pycharm/pycharm_04.jpg" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

Next, go to **Build, Execution, Deployment ⇒ Deployment**. You should see the server you just added. Mark it as the **default** server.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="lazy" path="assets/img/pycharm/pycharm_05.jpg" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

Select your project folder (`MVOC` in this example) and choose **Tools ⇒ Deployment ⇒ Upload to your_username@itigerXX:22** to upload your files to the sync folder. Also enable **Automatic Upload (Always)**, so that every `Ctrl+S` uploads the file you just saved.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="lazy" path="assets/img/pycharm/pycharm_06.jpg" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

> **Important**: the remote interpreter runs the code in the **sync folder** on the cluster, not your local files. If automatic upload is off, your latest edits won't be what runs.

You can now run your code on the cluster from PyCharm.

**3. Debug on the remote server**

PyCharm's debugger works with the remote interpreter: set breakpoints, watch variables, and evaluate expressions while paused. In recent versions, however, the debugger may open a **temporary copy** of the file instead of your local one. To fix this, map local paths to remote paths.

Select the file you want to debug (`pipeline_i2vgen_xl.py` in this example) and choose **Run ⇒ Debug**:

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="lazy" path="assets/img/pycharm/pycharm_07.jpg" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

Then **Edit the configuration**. Under **Path mappings**, map the **local path** (your local `.py` file) to the **remote path** (the same file on the cluster), and click **OK**:

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="lazy" path="assets/img/pycharm/pycharm_08.jpg" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

You can now debug as usual. Explore each tab of the debugger to find more useful functions.

**4. When your compute node changes**

Each new `srun` job may land on a different node (e.g. `itiger01` today, `itiger03` tomorrow). When that happens, update PyCharm. Right-click in the editor and choose **More Run/Debug ⇒ Modify Run Configuration**:

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="lazy" path="assets/img/pycharm/pycharm_09.jpg" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

Check that the interpreter and the deployment (file transfer) server both point to the node you are currently assigned, and set that node as the default server:

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="lazy" path="assets/img/pycharm/pycharm_10.jpg" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="lazy" path="assets/img/pycharm/pycharm_11.jpg" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>


## Tips

### Restart the Jupyter kernel after changing a package

Python does not reload a package that is already imported. If you edit the source of a package you are using in Jupyter, restart the kernel for the change to take effect. For example, this applies after editing Hugging Face `diffusers` code behind a Text2Video pipeline.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="lazy" path="assets/img/pycharm/pycharm_12.jpg" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

### Keep big files out of `/home`

`/home/your_username` has only 20 GB, so redirect large installs and caches to `/project`:

**1. Install Miniconda in `/project/your_username`**: when the installer prompts for a location, enter a path under `/project/your_username`.

**2. Point the Hugging Face cache to `/project`**: pass `cache_dir` when loading a model,

```python
tokenizer = AutoTokenizer.from_pretrained("roberta-base", cache_dir="/project/your_username/.cache/huggingface")
model = AutoModelForMaskedLM.from_pretrained("roberta-base", cache_dir="/project/your_username/.cache/huggingface")
```

or set it once for every library with an environment variable (before importing `transformers`):

```bash
export HF_HOME=/project/your_username/.cache/huggingface
```

**3. Or move the whole cache directory**:

```bash
mkdir -p /project/$USER/.cache
ln -s /project/$USER/.cache ~/.cache   # only if ~/.cache does not exist yet
```

### Install libraries without root

You don't have `sudo` on the cluster. Install packages into your own conda environment (`conda install ...` or `pip install ...` with the environment activated) instead of system-wide.


## GPU Request Script

This script lists the GPU types currently available, asks you for the job parameters, and starts `srun` inside a detached `tmux` session. Save it as `/home/your_username/apply_gpu.sh`:

```bash
#!/bin/bash

# Get available GPU types and deduplicate
# (skip nodes that are down or draining, and nodes without GPUs)
gpu_types=$(sinfo -N -h -o "%G %t" | grep -v -E "down|drain" | awk -F: '/^gpu:/ {print $2}' | sort -u)

# Convert to array
IFS=$'\n' read -d '' -r -a gpu_options <<< "$gpu_types"

# Check if GPU types are empty
if [ ${#gpu_options[@]} -eq 0 ]; then
    echo "Error: No available GPU nodes found!"
    exit 1
fi

# Display selection menu
echo "Available GPU types:"
for i in "${!gpu_options[@]}"; do
    printf "%2d) %s\n" $((i+1)) "${gpu_options[i]}"
done

# Get user input for GPU type
while true; do
    read -p "Select GPU type (1-${#gpu_options[@]}): " choice
    if [[ "$choice" =~ ^[0-9]+$ ]] && [ "$choice" -ge 1 ] && [ "$choice" -le ${#gpu_options[@]} ]; then
        selected_gpu="${gpu_options[$((choice-1))]}"
        break
    else
        echo "Invalid input, please enter a number between 1-${#gpu_options[@]}"
    fi
done

# Allow user to define custom parameters
read -p "Enter partition (default: bigTiger): " partition
partition=${partition:-bigTiger}

read -p "Enter time limit (default: 24:00:00): " time_limit
time_limit=${time_limit:-24:00:00}

read -p "Enter memory (default: 24G): " memory
memory=${memory:-24G}

# Add the G unit if the user typed only a number
if [[ "$memory" =~ ^[0-9]+$ ]]; then
    memory="${memory}G"
fi

read -p "Enter GPU count (default: 1): " gpu_count
gpu_count=${gpu_count:-1}

read -p "Enter CPUs per task (default: 4): " cpus_per_task
cpus_per_task=${cpus_per_task:-4}

read -p "Enter number of tasks (default: 1): " ntasks
ntasks=${ntasks:-1}

# Check if the tmux session already exists
if tmux list-sessions 2>/dev/null | grep -q "^gpu-keeper:"; then
    echo "Error: tmux session 'gpu-keeper' already exists!"
    while true; do
        read -p "Would you like to create a new tmux session? (yes/no): " create_new
        case "$create_new" in
            [Yy]*)
                while true; do
                    read -p "Enter a new tmux session name: " new_session
                    if [ -n "$new_session" ]; then
                        break
                    else
                        echo "Session name cannot be empty."
                    fi
                done
                break
                ;;
            [Nn]*)
                echo "Exiting script."
                exit 0
                ;;
            *)
                echo "Please answer yes or no."
                ;;
        esac
    done
else
    new_session="gpu-keeper"
fi

# Create new tmux session and submit job
tmux new-session -d -s "$new_session" "srun --partition=$partition --time=$time_limit --mem=$memory --gres=gpu:${selected_gpu}:${gpu_count} --cpus-per-task=$cpus_per_task --ntasks=$ntasks --pty bash"

# Output usage instructions
echo -e "\nSession '$new_session' started in tmux."
echo "Useful commands:"
echo "    tmux attach -t $new_session"
echo "    Detach with: Ctrl+B followed by D"
echo "    tmux list-sessions"
echo ""
```

Run it with:

```bash
bash apply_gpu.sh
```

Pick a GPU type by entering its number. The **H100** is the fastest. If you run into CUDA compatibility issues, choose the **RTX 6000 Ada** or **RTX 5000 Ada** instead. Then press **Enter** at each prompt to accept the defaults:

- Partition: `bigTiger` (do not change)
- Time limit: 24 hours (48 hours maximum)
- Memory: 24 GB
- GPUs: 1
- CPU cores: 4
- Tasks: 1

Then attach to your node with `tmux attach -t gpu-keeper`.


## Useful Commands

| Task | Command |
| --- | --- |
| Cancel all your jobs | `scancel -u $(whoami)` |
| Check `/home` usage | `du -sh /home/your_username/` |
| Check `/project` usage | `df -h /project/your_username/` |
| Start TensorBoard (compute node) | `tensorboard --logdir . --port 6016 --samples_per_plugin "images=100"` |
| View TensorBoard (local machine) | `ssh -L 16006:localhost:6016 itiger01`, then open `http://localhost:16006` |


## Useful Links

- [PyCharm: Configure an interpreter using SSH](https://www.jetbrains.com/help/pycharm/configuring-remote-interpreters-via-ssh.html)
- [Slurm `srun` documentation](https://slurm.schedmd.com/srun.html)
