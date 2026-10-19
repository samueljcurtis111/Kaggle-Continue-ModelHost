# OmniCoder-9B on Kaggle with VS Code Continue

This project lets you run **OmniCoder-9B Q4** on Kaggle's free GPU notebooks and use it from VS Code through the Continue extension.

Basically: use Kaggle's dual Tesla T4 GPUs to run the model remotely, then connect to it from your local VS Code installation. That way, you can use the model for coding questions, inline edits, and other tasks without running it on your own hardware.

The setup uses Ollama to serve the model and ngrok to expose the server so Continue can connect to it. It also uses an 8-bit KV cache (`q8_0`) to reduce memory usage when working with longer prompts.

## What it does

- **Runs on Kaggle:** Uses a Kaggle notebook with two Tesla T4 GPUs.
- **Supports longer contexts:** Configured to target context lengths of up to 128K tokens, with higher limits depending on available memory and model support.
- **Reduces KV-cache memory usage:** Uses `q8_0` quantization to make longer contexts more practical.
- **Works with VS Code:** Connects to the running model through Continue for chat and code editing.
- **Uses free-tier resources:** Designed around Kaggle's free GPU allowance, so usage is subject to its quotas and session limits.

## Requirements

Before getting started, you'll need:

1. **Kaggle:** An account with access to GPU notebooks and internet access enabled.
2. **ngrok:** An account and an authentication token from the [ngrok dashboard](https://dashboard.ngrok.com/).
3. **VS Code:** Installed on your computer.
4. **Continue:** The [Continue extension](https://marketplace.visualstudio.com/items?itemName=Continue.continue) installed in VS Code.

## Setup

### Step 1: Create a Kaggle notebook

1. Create a new notebook on [Kaggle](https://www.kaggle.com/).
2. Open the notebook's session options.
3. Set the accelerator to **GPU T4 x2**.
4. Enable internet access.
5. Create two code cells and run them in order.

### Step 2: Install the dependencies

Run this in the first cell:

```python
!apt-get update && apt-get install -y zstd
!curl -fsSL https://ollama.com/install.sh | sh
!pip install pyngrok
```

### Step 3: Launch the Ollama and Ngrok tunnel

Replace "YOUR_NGROK_TOKEN" with your actual one

In new Cell
```python
import subprocess
import time
import os
from pyngrok import ngrok

# Configure Ollama environment
os.environ["OLLAMA_HOST"] = "0.0.0.0"
os.environ["OLLAMA_ORIGINS"] = "*"
os.environ["OLLAMA_CONTEXT_LENGTH"] = "262144"  
os.environ["OLLAMA_KV_CACHE_TYPE"] = "q8_0"     

# Launch server
subprocess.Popen(["ollama", "serve"])
print("Starting Ollama server...")
time.sleep(5)

# Pull model 
print("Pulling OmniCoder-9B GGUF...")
subprocess.run(["ollama", "pull", "hf.co/Tesslate/OmniCoder-9B-GGUF:Q4_K_M"])

# Tunnel connection
ngrok.set_auth_token("YOUR_NGROK_TOKEN") 
public_url = ngrok.connect(11434).public_url

print("\n" + "="*50)
print(f"Copy this URL into VS Code: {public_url}")
print("="*50)
```

### Step 4: Continue config

Go into the config YAML file 

WINDOWS:
```
%USERPROFILE%\.continue\config.yaml
```
MAC: (I use Windows so not 100% sure if someone with mac uses this I would like to know what the path is)
```
~/.continue/config.yaml
```

then add this the the bottom

```
models:
  - name: Kaggle OmniCoder
    provider: ollama
    model: hf.co/Tesslate/OmniCoder-9B-GGUF:Q4_K_M
    apiBase: "https://<your-ngrok-id>.ngrok-free.dev"
    contextLength: 262144
```

### TROUBLE SHOOTING
#### 1 or 2 word unfinished responses: 
Click the + New Session button in Continue to clear your prompt history. This happens when the active chat history fills up the context limit.

#### Session Disconnections: 
Kaggle sessions automatically shut down after 60 minutes of inactivity or 12 hours total runtime. Rerun Cell 2 to start a new tunnel and update your config.yaml URL.

#### ngrok Auth Error: 
Ensure your auth token in Cell 2 is wrapped in quotes ("...").


