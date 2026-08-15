Local AI Distributed Coding Environment
(llama.cpp Branch)

This branch documents the infrastructure and configuration for a distributed, locally hosted Large Language Model (LLM) coding environment, utilizing llama.cpp as the bare-metal inference engine.

By offloading the AI inference to a dedicated high-RAM host (Apple M5 Pro, 48GB Unified Memory), this setup provides privacy-first, zero-latency AI code assistance to multiple client machines across the local area network.


🏗️ Architecture

Inference Host: MacBook Pro (M5 Pro, 48GB RAM) running the llama.cpp HTTP server.
Clients: Windows 11 Laptop (Remote VS Code)
IDE Integration: Continued (or Continue.dev) extension for VS Code.
Models (GGUF Format):Chat & Reasoning: qwen2.5-coder-32b-instruct-q4_K_M.gguf


⚙️ Host Setup (MacBook Pro)

Unlike Ollama, llama.cpp requires you to download model weights manually and run the server via the command line.

1. Install llama.cppThe easiest way to install a Metal-optimized version of llama.cpp on macOS is via Homebrew:
brew install llama.cpp

2. Download the Model
Download the quantized GGUF model files from HuggingFace. A highly recommended repository is bartowski/Qwen2.5-Coder-32B-Instruct-GGUF.
# Create a directory for your models
mkdir -p ~/ai-models
cd ~/ai-models

# Download a 4-bit quantized version of the 32B model using wget or your browser
wget https://huggingface.co/bartowski/Qwen2.5-Coder-32B-Instruct-GGUF/resolve/main/Qwen2.5-Coder-32B-Instruct-Q4_K_M.gguf

3. Start the Inference Server
Run the llama-server command. We will tell it to listen on 0.0.0.0 (all network interfaces) and use port 11434 so it perfectly mimics the Ollama endpoint your extensions are already looking for.
llama-server -m ~/ai-models/Qwen2.5-Coder-32B-Instruct-Q4_K_M.gguf \
  --host 0.0.0.0 \
  --port 11434 \
  -c 8192 \
  -ngl 99
-c 8192: Sets the context window to 8k tokens (adjust based on memory needs).-ngl 99: Offloads all layers to the M5 GPU (Metal) for maximum speed.


💻 Client Configuration (Windows 11)
Todo