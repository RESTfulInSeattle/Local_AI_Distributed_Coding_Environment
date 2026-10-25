Local AI Distributed Coding Environment
This repo documents the infrastructure and configuration for a distributed, locally hosted Large Language Model (LLM) coding environment.

By offloading the AI inference to a dedicated high-RAM host (Apple M5 Pro, 48GB Unified Memory), this setup provides privacy-first, low-latency AI code assistance to multiple client machines across a small local area network.

The first iteration of this project used Ollama (available in the Ollama branch), and the second used bare-metal llama.cpp (available in the llama-cpp branch).

🏗️ Architecture (Three-Tier)
This environment uses a production-grade three-tier architecture to support high concurrency, dynamic model routing, and secure local data retrieval:

Inference Layer: vllm-mlx on the M5 Pro host. 
Leverages continuous batching and MLX-backed Metal kernels for maximum throughput on Apple Silicon.

Gateway Layer: LiteLLM Proxy. 
Acts as an AI gateway, standardizing APIs across models so Visual Studio 2026 and VS Code interact with local models exactly as they would with OpenAI.

Context Layer: Telemetry MCP Server. https://github.com/RESTfulInSeattle/Telemetry_MCP_Server 
A local Model Context Protocol (MCP) server that securely exposes system health and logs directly into the context window, preventing proprietary data from leaking to frontier cloud APIs.

Inference Host: MacBook Pro (M5 Pro, 48GB RAM)


Clients:

VS Code run locally on the MacBook Pro

Windows 11 Laptop with VS Code and Visual Studio Professional 2026


Models:

Chat & Reasoning/Tooling: Qwen3-Omni-30B-A3B-Thinking

Chat & Reasoning: qwen3-4b-thinking

Autocomplete: Qwen2.5-Coder-7B-Instruct


⚙️ Tier 1: Inference Host Setup (vllm-mlx)
Unlike the previous llama.cpp configuration, vllm utilizing the Apple MLX framework optimizes batching and prefix caching for multi-client workflows.

### 1. Install vLLM with MLX Support
Ensure you have a Python 3.10+ environment, then install the Apple Silicon-specific vllm package:

```bash
# Create a dedicated environment
uv venv vllm-env
source vllm-env/bin/activate

# Install the MLX-optimized vLLM package
uv pip install vllm-mlx

### 2. Start the Inference Server
Instead of using GGUF files, `vllm-mlx` works best with MLX-optimized Safetensors downloaded from the HuggingFace `mlx-community`. It will automatically download the weights on first run.

```bash
# Start the 7B coder model on port 8000 using the vllm-mlx binary
uv run vllm-mlx serve mlx-community/Qwen2.5-Coder-7B-Instruct-4bit \
    --port 8000 \
    --continuous-batching

Note: Run your background autocomplete models on separate ports (e.g., 8001) if hosting concurrently.

### Command Explained

* **`uv run` (Managed Execution):**
  Instead of manually activating a virtual environment (e.g., `source .venv/bin/activate`), `uv run` automatically detects your project's isolated environment and securely executes the command inside it.

* **`vllm-mlx serve' (The Inference Engine):
This is the specialized command to spin up the OpenAI-compatible REST API server utilizing Apple's Metal GPU and unified memory.

* **'mlx-community/Qwen2.5-Coder-7B-Instruct-4bit' (The Target Model):
This points vllm-mlx to a HuggingFace repository specifically optimized for Apple Silicon. If you haven't run this command before, it will automatically download and cache the MLX Safetensors directly into your environment.

* **`--port 8000` (Port Assignment):**
  Tells the server to listen for local API requests on port `8000`. This is the endpoint your VS Code extensions (like Continue or Cline) will point to.

* **'--continuous-batching' (Performance Boost):
This flag explicitly enables the high-throughput, multi-client batching engine and paged KV cache natively engineered for Apple Silicon. This ensures the engine doesn't queue requests sequentially, keeping inference speeds high even when VS Code autocomplete and chat windows are firing requests simultaneously.


 ### 🔑 Hugging Face Authentication (Fixing Unauthenticated Errors)

If you receive an "unauthenticated requests" warning or hit a rate limit while pulling models from Hugging Face, you need to authenticate your terminal. 

1. Create a free account at [Hugging Face](https://huggingface.co/).
2. Navigate to **Settings > Access Tokens** and generate a new **Read** token.
3. Export the token into your Mac's environment before downloading or running the server. 

Run this command in your terminal (replacing the placeholder with your actual token):
```bash
export HF_TOKEN="hf_your_token_string_here"


# To Test the Server

curl http://127.0.0.1:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "mlx-community/Qwen2.5-Coder-7B-Instruct-4bit",
    "messages": [
      {"role": "user", "content": "Write a Hello World in Python."}
    ]
  }'


🚦 Tier 2: Gateway Router (LiteLLM Proxy)
LiteLLM solves the Visual Studio 2026 compatibility issue by presenting a standard OpenAI-compatible API to all IDEs while routing traffic to your local vllm instances.

1. Install LiteLLM
Bash
uv tool install litellm
2. Configure the Router (litellm_config.yaml)
Create a configuration file to map your internal IDE model names to your local vllm endpoints.

YAML
model_list:
  - model_name: qwen3-4b-thinking
    litellm_params:
      model: openai/Qwen/Qwen2.5-Coder-7B-Instruct
      api_base: "http://127.0.0.1:8000/v1"
      api_key: "sk-local"
  - model_name: qwen3-omni-30b
    litellm_params:
      model: openai/Qwen/Qwen3-Omni-30B-A3B-Thinking
      api_base: "http://127.0.0.1:8001/v1"
      api_key: "sk-local"
3. Start the Gateway
Bash
# Start LiteLLM proxy listening on all network interfaces
litellm --config litellm_config.yaml --host 0.0.0.0 --port 11434
(We use port 11434 so previous client configurations do not need to be updated).

🔌 Tier 3: Context Layer (Telemetry MCP Server)
To allow the local LLM to read local system logs and troubleshoot infrastructure without giving it raw bash access or sending trade secrets to the cloud, we bind it to our Custom MCP Server.

1. Start the MCP Server
Ensure your Telemetry MCP server is configured and running per the instructions in the Telemetry_MCP_Server repository.

2. VS Code Configuration (~/.continue/config.json)
Bind the MCP server to your local Qwen profile:

JSON
{
  "models": [
    {
      "title": "Local Secure (Telemetry & Trade Secrets)",
      "provider": "openai",
      "model": "qwen3-4b-thinking",
      "apiBase": "http://<mac_ip_address>:11434/v1",
      "apiKey": "sk-local"
    }
  ],
  "mcpServers": {
    "TelemetryInspector": {
      "command": "uv",
      "args": [
        "run",
        "--directory",
        "/Users/YOUR_MAC_USERNAME/Development/Telemetry_MCP_Server",
        "python",
        "src/telemetry_mcp_server/server.py"
      ]
    }
  }
}
💻 Client Configurations
Because LiteLLM is translating everything into the OpenAI standard, client configuration is vastly simplified.

VS Code (Mac & Windows Clients)
Open Copilot settings or Continue.dev.

Add a Custom OpenAI Endpoint:

URL/Base URL: http://<mac_ip_address>:11434/v1

Model: qwen3-4b-thinking or qwen3-omni-30b

API Key: sk-local

Visual Studio 2026 Professional (Windows 11 Client)
With LiteLLM Proxy acting as the API gateway, Visual Studio 2026 can now connect to your local models by targeting the proxy as a standard OpenAI endpoint or custom Ollama URL route, completely bypassing previous connection failures.