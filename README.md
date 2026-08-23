Local AI Distributed Coding Environment

This repo documents the infrastructure and configuration for a distributed, locally hosted Large Language Model (LLM) coding environment, utilizing llama.cpp as the bare-metal inference engine.

By offloading the AI inference to a dedicated high-RAM host (Apple M5 Pro, 48GB Unified Memory), this setup provides privacy-first, zero-latency AI code assistance to multiple client machines across the local area network.


🏗️ Architecture

Inference Host: MacBook Pro (M5 Pro, 48GB RAM) running the llama.cpp HTTP server.
Clients: Windows 11 Laptop (Remote VS Code)
IDE Integration: Continued (or Continue.dev) extension for VS Code.
Models (GGUF Format):Chat & Reasoning: qwen2.5-coder-32b-instruct-q4_K_M.gguf


⚙️ Host Setup (MacBook Pro)

Unlike Ollama, llama.cpp requires you to download model weights manually and run the server via the command line.

1. Install llama.cpp
The easiest way to install a Metal-optimized version of llama.cpp on macOS is via Homebrew:
brew install llama.cpp

If you need more control, follow the build steps at the llama.cpp GitHub Repo
https://github.com/ggml-org/llama.cpp/blob/master/docs/build.md 


2. Download the Model
Download the quantized GGUF model files from HuggingFace. 
https://huggingface.co/ggml-org/models?search=qwen&p=0

# Create a directory for your models
mkdir -p ~/ai-models
cd ~/ai-models

# Download a 4-bit quantized version of the 32B model using wget or your browser
https://huggingface.co/ggml-org/Qwen3-4B-Thinking-2507-Q8_0-GGUF


3. Start the Inference Server
Run the llama-server command. We will tell it to listen on 0.0.0.0 (all network interfaces) and use port 11434 so it perfectly mimics the Ollama endpoint the vs code extensions are already looking for.

Since I downloaded and manually build the llamma.cpp repo, this command is run from within the built llamma.cpp directory at build/bin

./llama-server -m ~/Development/ai-models/qwen3-4b-thinking-2507-q8_0.gguf -c 32768 -ngl 99 --host 0.0.0.0 --port 11434


-c 32768: Context Window

Sets the context window to 32k tokens.  To put a 32k context window into perspective, that is roughly 24,000 words. You can easily highlight dozens of complex C# files, drop in documentation, and ask the model to architect a solution without it "forgetting" the beginning of your prompt.

-ngl 99: Number of GPU Layers.  

This flag tells llama.cpp how many layers of the AI model to offload to the GPU instead of the CPU.  The Apple M5 Pro chip uses a highly efficient "Unified Memory" architecture and a powerful Metal GPU. By setting this to a ridiculously high number like 99 (which is higher than the total layers in most models), you force llama.cpp to load 100% of the model into your blazing-fast GPU memory. 

--host 0.0.0.0:  Network Binding
By default, web servers (including llama-server) only listen to 127.0.0.1 (localhost). This means they will only talk to programs running on the exact same computer.

Setting the host to 0.0.0.0 tells the server to listen to all available network interfaces, including your Wi-Fi card. This is what allows another client to access the engine.

--port 11434: Port Assignment
This simply tells the server which network port to keep open for incoming HTTP requests.  11434 is the default port used by Ollama, so the client configuration from the Ollama iteration of this project can be used without changing the settings.


💻 Client Configuration (Windows 11)
Todo