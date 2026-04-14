---
layout: post
title: How to Run a Qwen3 Chatbot on the Cluster
date: 2025-08-16 16:00:00
description: run a chatbot, srun  
---

In our [previous chatbot tutorial](https://itiger-cluster.github.io/tutorials/2025/llama-chatbot/), we ran a **Meta LLaMA 3.1-8B-Instruct** chatbot on the cluster. LLaMA is a *gated* model, meaning you need to accept a license agreement and set up a Hugging Face access token before downloading it.

This time, we will use **Qwen3-8B**, an open-weight model released by Alibaba under the Apache 2.0 license. **No access token or license agreement is needed** — you can download and run it directly. Qwen3 also supports seamless switching between a "thinking" mode (for complex reasoning) and a "non-thinking" mode (for general chat).

We will start with a minimal **terminal chatbot**, then upgrade it to a **web interface**, both running interactively via `srun`.

#### Prerequisites

This tutorial assumes you already have **conda** (Miniconda or Anaconda) installed on the cluster. If you haven't set it up yet, follow the instructions in our [first tutorial](https://itiger-cluster.github.io/tutorials/2024/run-model/) which covers how to download and configure Miniconda.


## Environment Setup

1. Create a conda environment

Make sure you are working under the `/project` directory (to avoid `/home` quota issues).  
Then create a dedicated conda environment and install the required libraries:

```bash
cd /project/your_username/
mkdir -p qwen-chatbot
cd qwen-chatbot
conda create -n qwen_chat python=3.10 -y
conda activate qwen_chat
```

2. Install PyTorch, Hugging Face ecosystem, and Gradio

Qwen3 requires `transformers>=4.51.0`. Install with:

```bash
pip install torch torchvision torchaudio
pip install -U "transformers>=4.51.0" "accelerate>=0.33.0" "tokenizers>=0.19.0" safetensors gradio
```

That's it for setup. Since Qwen3-8B is not a gated model, no Hugging Face login or access token is required.

> **Note**: If you see a warning like `CUDA initialization: The NVIDIA driver on your system is too old`, it means the default PyTorch ships a newer CUDA version than the cluster driver currently supports. In that case, reinstall PyTorch with a matching CUDA version, e.g.:  
> `pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu124`


## Run a Chatbot in your terminal

1. Prepare the script

Save the following script in your project directory, e.g. `/project/your_username/qwen-chatbot/qwen_chat.py`:

```python
from transformers import AutoTokenizer, AutoModelForCausalLM, logging
import torch
import sys
logging.set_verbosity_error()

MODEL_ID = "Qwen/Qwen3-8B"

tok = AutoTokenizer.from_pretrained(MODEL_ID)

mdl = AutoModelForCausalLM.from_pretrained(
    MODEL_ID,
    torch_dtype=torch.bfloat16 if torch.cuda.is_available() else torch.float32,
    device_map="auto",
)

print("💬 Qwen3 Lab Assistant — type 'exit' to quit")

history = []

while True:
    user = input("").strip()
    if user.lower() == "exit":
        sys.exit(0)

    messages = history + [{"role": "user", "content": user}]

    text = tok.apply_chat_template(
        messages,
        tokenize=False,
        add_generation_prompt=True,
        enable_thinking=False,  # set True to enable chain-of-thought reasoning
    )
    inputs = tok([text], return_tensors="pt").to(mdl.device)

    with torch.no_grad():
        output = mdl.generate(
            **inputs,
            max_new_tokens=512,
            temperature=0.7,
            do_sample=True,
        )

    output_ids = output[0][len(inputs.input_ids[0]):].tolist()
    reply = tok.decode(output_ids, skip_special_tokens=True).strip()

    print(f"Assistant: {reply}\n")

    history.append({"role": "user", "content": user})
    history.append({"role": "assistant", "content": reply})
```

2. Run it interactively on a GPU node:

```bash
srun --pty --gres=gpu:1 --cpus-per-task=4 --mem=64G --time=2:00:00 --partition=bigTiger \
     python /project/your_username/qwen-chatbot/qwen_chat.py
```

3. Start Conversations:

Once the chatbot starts, you can interact in the terminal, for example:

```
💬 Qwen3 Lab Assistant — type 'exit' to quit

Can you explain what a GPU is?
Assistant: A Graphics Processing Unit (GPU) is a specialized processor originally designed to accelerate graphics rendering. Today, GPUs are widely used for parallel computing tasks such as deep learning, scientific simulation, and data processing...

exit
```


## Upgrade to a Web Interface (Gradio)

For a more user-friendly interface, we can use Gradio to run the chatbot in a browser.

1. Prepare the script

Save this script as `/project/your_username/qwen-chatbot/qwen_gradio.py`, and **change line 5** to your actual directory (`your_username`) and PORT number to a four digit number you like and larger than 1024 (`your_4_digits_number`):

```python
import os, socket, gradio as gr
from transformers import AutoTokenizer, AutoModelForCausalLM
import torch

# Put HF cache under /project to avoid home quota
os.environ.setdefault("HF_HOME", "/project/your_username/.cache/huggingface")

MODEL_ID = "Qwen/Qwen3-8B"
PORT = your_4_digits_number  # change if the port is busy

tok = AutoTokenizer.from_pretrained(MODEL_ID)
mdl = AutoModelForCausalLM.from_pretrained(
    MODEL_ID,
    torch_dtype=torch.bfloat16 if torch.cuda.is_available() else torch.float32,
    device_map="auto",
)

def chat_fn(message, history):
    """
    Gradio passes:
      - message: current user input (str)
      - history: list of (user, assistant) tuples
    Convert to Qwen3 chat template messages.
    """
    msgs = []
    for u, b in (history or []):
        if u:
            msgs.append({"role": "user", "content": u})
        if b:
            msgs.append({"role": "assistant", "content": b})
    msgs.append({"role": "user", "content": message})

    text = tok.apply_chat_template(
        msgs,
        tokenize=False,
        add_generation_prompt=True,
        enable_thinking=False,
    )
    inputs = tok([text], return_tensors="pt").to(mdl.device)

    with torch.no_grad():
        out = mdl.generate(
            **inputs,
            max_new_tokens=512,
            temperature=0.7,
            do_sample=True,
        )

    output_ids = out[0][len(inputs.input_ids[0]):].tolist()
    reply = tok.decode(output_ids, skip_special_tokens=True).strip()
    return reply

print(f"[Gradio] compute node = {socket.gethostname()}, port = {PORT}", flush=True)

demo = gr.ChatInterface(fn=chat_fn, title="Qwen3 Lab Assistant")
demo.launch(server_name="0.0.0.0", server_port=PORT, show_error=True)
```

2. Run it on a GPU node

```bash
srun --pty --gres=gpu:1 --cpus-per-task=4 --mem=64G --time=2:00:00 --partition=bigTiger \
     python /project/your_username/qwen-chatbot/qwen_gradio.py
```

This will start the Qwen3 chatbot on a GPU node. In the terminal you should see output like:

```
[Gradio] compute node = itiger06, port = xxxx
Running on local URL:  http://0.0.0.0:xxxx
```

xxxx (`your_4_digits_number` in the script) is the PORT number you chose before.


3. Connect from your local machine

Open a new terminal on your local computer and create an SSH tunnel.
Replace `NODE` with the actual compute node name printed above e.g. `itiger06`, and `your_username` with your actual username:

```bash
ssh -Nf -L LOCAL_PORT:NODE:REMOTE_PORT your_username@itiger.memphis.edu
```

REMOTE_PORT is the PORT number you chose before (`your_4_digits_number` in the script), and you may choose the same or another number as LOCAL_PORT as long as it's not occupied on your local machine.

Now open a browser on your local machine and go to: `http://localhost:LOCAL_PORT`

You should see the Qwen3 Lab Assistant Gradio interface, where you can start chatting with the model interactively.


<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/qwen_chatbot.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>


## Notes

- **Thinking mode**: Qwen3 supports a "thinking" mode for step-by-step reasoning. Set `enable_thinking=True` in `apply_chat_template` to enable it. This adds a chain-of-thought block before the final answer, which is useful for math or coding questions but slower for general chat.
