# **Local AI Distributed Coding Environment**

This repo documents the infrastructure and configuration for a distributed, locally hosted Large Language Model (LLM) coding environment, utilizing llama.cpp as the bare-metal inference engine.

By offloading the AI inference to a dedicated high-RAM host (Apple M5 Pro, 48GB Unified Memory), this setup provides the start of privacy-first low-latency AI code assistance to multiple client machines across a small local area network.

The first iteration of this project used Ollama, which you can find in the Ollamma branch.


## **🏗️ Architecture**

* **Inference Host:** MacBook Pro (M5 Pro, 48GB RAM) running the llama.cpp HTTP server.  
* **Clients:**  
  * VS Code run locally on the MacBook Pro  
  * Windows 11 Laptop with VS Code and Visual Studio Professional 2026  
* **IDE Integration:** 
  * VS Code's integrated GitHub Copilot accepts Custom Endpoints for models.
  * Visual Studio Professional 2026 currently accepts Ollama as a model, but will require more testing.
* **Models (GGUF Format):**  
  * Chat & Reasoning/Tooling:  Qwen3-Omni-30B-A3B-Thinking-Q8_0.gguf 
  * Chat & Reasoning: qwen3-4b-thinking-2507-q8_0.gguf
  * Autocomplete:  Qwen2.5-Coder-7B-Instruct-Q4_K_M.gguf


## **⚙️ Host Setup (MacBook Pro)**

Unlike Ollama, llama.cpp requires you to download model weights manually and run the server via the command line.

### **1\. Install llama.cpp**

The easiest way to install a Metal-optimized version of llama.cpp on macOS is via Homebrew:

brew install llama.cpp

If you need more control, follow the build steps at the [llama.cpp GitHub Repo](https://github.com/ggml-org/llama.cpp/blob/master/docs/build.md).

### **2\. Download the Model**

Download the quantized GGUF model files from HuggingFace:

[https://huggingface.co/ggml-org/models?search=qwen\&p=0](https://huggingface.co/ggml-org/models?search=qwen&p=0)

\# Create a directory for your models  
mkdir \-p \~/ai-models  
cd \~/ai-models

\# Download a 4-bit quantized version of the 32B model using wget or your browser  
wget https://huggingface.co/ggml-org/Qwen3-4B-Thinking-2507-Q8\_0-GGUF/resolve/main/qwen3-4b-thinking-2507-q8\_0.gguf

### **3\. Start the Inference Server**

Since I downloaded and manually built the llama.cpp repo, this command is run from within the built llama.cpp directory at build/bin:

cd ~/Development/llama.cpp/build/bin

**Model:** `qwen3-4b-thinking-2507-q8_0.gguf`

* **Size/Type:** 4 Billion Parameters, 8-bit Quantization.
* **RAM Footprint:** ~4.5 GB.
* **Best For:** Quick Q&A, battery-saving mode, or tasks requiring massive context windows.
* **Notes:** Leaves over 40GB of RAM free, allowing for massive 32k+ context windows to process dozens of files at once without slowing down.

**Start Command:**

./llama-server -m ~/Development/ai-models/qwen3-4b-thinking-2507-q8_0.gguf -c 32768 -ngl 99 --host 0.0.0.0 --port 11434


### **Server Flags Explained**

* **\-c 32768 (Context Window):**  
  Sets the context window to 32k tokens. To put a 32k context window into perspective, that is roughly 24,000 words. You can easily highlight dozens of complex C\# files, drop in documentation, and ask the model to architect a solution without it "forgetting" the beginning of your prompt.  
* **\-ngl 99 (Number of GPU Layers):**  
  This flag tells llama.cpp how many layers of the AI model to offload to the GPU instead of the CPU. The Apple M5 Pro chip uses a highly efficient "Unified Memory" architecture and a powerful Metal GPU. By setting this to a ridiculously high number like 99 (which is higher than the total layers in most models), you force llama.cpp to load 100% of the model into your blazing-fast GPU memory.  
* **\--host 0.0.0.0 (Network Binding):**  
  By default, web servers (including llama-server) only listen to 127.0.0.1 (localhost). This means they will only talk to programs running on the exact same computer. Setting the host to 0.0.0.0 tells the server to listen to all available network interfaces, including your Wi-Fi card. This is what allows another client to access the engine.  
* **\--port 11434 (Port Assignment):**  
  This simply tells the server which network port to keep open for incoming HTTP requests. 11434 is the default port used by Ollama, so the client configuration from the Ollama iteration of this project can be used without changing the settings.

  ### Host URL Determination

  ipconfig getifaddr en0

  

## **💻 Client Configurations **

### VS Code (Local Mac Client)

VS Code introduced a way to bypass third-party extensions entirely and hook custom, locally-hosted LLMs directly into the native VS Code Chat and Copilot UI.

1. Manage Models in Copilot, and click Add models | Custom Endpoint


2. Add the Custom Endpoint

{
    "name": "Local llama.cpp",
    "vendor": "customendpoint",
    "apiType": "openai",
    "models": [
        {
            "id": "qwen3-4b-thinking",
            "name": "Local Qwen3 (llama.cpp)",
            "url": "http://127.0.0.1:11434/v1/chat/completions",
            "toolCalling": true,
            "vision": false,
            "maxInputTokens": 32768,
            "maxOutputTokens": 4096
        }
    ]
}



### **VS Code (Windows 11 Client)**

1. Manage Models in Copilot, and click Add models | Custom Endpoint


2. Add the Custom Endpoint

{
    "name": "Local llama.cpp",
    "vendor": "customendpoint",
    "apiType": "openai",
    "models": [
        {
            "id": "qwen3-omni-30b",
            "name": "Local Qwen3 30B (llama.cpp)",
            "url": "http://<mac ip address>:11434/v1/chat/completions",
            "toolCalling": true,
            "vision": false,
            "maxInputTokens": 16384,
            "maxOutputTokens": 4096
        }
    ]
}


### **Visual Studio 2026 Professional (Windows 11 Client)**

In Visual Studio 2026 Professional, the "Custom Endpoint" terminology doesn't exist in the same way it does in VS Code.  

It currently only has Ollama as a provider, so an LLM Router Proxy will be experimented with to use with llama.cpp


