# Local AI Distributed Coding Environment

This repo documents the infrastructure and configuration for a distributed, locally hosted Large Language Model (LLM) coding environment.  By offloading the AI inference to a dedicated high-RAM host (Apple M5 Pro, 48GB Unified Memory), this setup provides privacy-first, low-latency AI code assistance to multiple client machines across a small local area network.

The first iteration of this project used Ollama (available in the ollama branch), and the second used bare-metal llama.cpp (llama-cpp branch).

Current Architecture: vLLM + Apple MLX
This iteration migrates the inference engine to vLLM utilizing the Apple MLX framework to solve the concurrency bottlenecks of earlier iterations. Key upgrades include:

Continuous Batching: Dynamically batches incoming requests from multiple LAN clients, preventing one user's long generation from blocking the rest of the network.

Prefix Caching: Reuses the compute for shared system prompts and large codebases, drastically reducing Time-To-First-Token (TTFT) for repetitive coding queries.

Native Silicon Optimization: Leverages the MLX backend to fully exploit the M5 Pro's unified memory architecture for maximum throughput.


## 🏗️ Architecture (Three-Tier)

This environment uses a three-tier architecture to support high concurrency, dynamic model routing, and secure local data retrieval:

* **Inference Layer: `vllm-mlx` on the M5 Pro host.** Leverages continuous batching and MLX-backed Metal kernels for maximum throughput on Apple Silicon.  
* **Gateway Layer: LiteLLM Proxy.** Acts as an AI gateway, standardizing APIs across models so Visual Studio 2026 and VS Code interact with local models exactly as they would with OpenAI.  
* **Context Layer: Telemetry MCP Server.** A local Model Context Protocol (MCP) server that securely exposes system health and logs directly into the context window, preventing proprietary data from leaking to frontier cloud APIs.

### Hardware & Models

* **Inference Host:** MacBook Pro (M5 Pro, 48GB RAM)  
* **Clients:** VS Code run locally on the MacBook Pro; Windows 11 Laptop with VS Code and Visual Studio Professional 2026  
* **Models:**  
  * Chat & Reasoning/Tooling: `Qwen3-Omni-30B-A3B-Thinking`  
  * Chat & Reasoning: `qwen3-4b-thinking`  
  * Autocomplete: `Qwen2.5-Coder-7B-Instruct`

---

## ⚙️ Tier 1 & 2: Environment & Infrastructure Setup

To prevent Python namespace collisions and Pydantic validation crashes on startup, both the `vllm-mlx` inference engine and the `litellm` proxy must be installed securely into the exact same isolated `uv` virtual environment.

### 1\. Build the Unified Environment

Ensure you have a Python 3.10+ environment, then install the Apple Silicon-specific vLLM package alongside the LiteLLM proxy extras:

\# Create a dedicated environment

uv venv vllm-env

source vllm-env/bin/activate

\# Install the MLX-optimized vLLM package AND the LiteLLM proxy together

uv pip install vllm-mlx "litellm\[proxy\]"

### 2\. Start the Inference Server (Tier 1\)

Instead of using GGUF files, `vllm-mlx` works best with MLX-optimized Safetensors downloaded from the HuggingFace `mlx-community`. It will automatically download the weights on first run.

Open **Terminal 1**:

source vllm-env/bin/activate

\# Start the 7B coder model on port 8000 using the vllm-mlx binary

uv run vllm-mlx serve mlx-community/Qwen2.5-Coder-7B-Instruct-4bit \\

    \--port 8000 \\

    \--continuous-batching

*(Note: Run your background autocomplete models on separate ports, e.g., 8001, if hosting concurrently).*

**Command Explained:**

* **`uv run` (Managed Execution):** Automatically detects your project's isolated environment and securely executes the command inside it.  
* **`vllm-mlx serve`:** Spins up the OpenAI-compatible REST API server utilizing Apple's Metal GPU and unified memory.  
* **`--continuous-batching` (Performance Boost):** Explicitly enables the high-throughput, multi-client batching engine natively engineered for Apple Silicon.

### 3\. Configure and Start the Gateway Router (Tier 2\)

LiteLLM intercepts OpenAI-compatible API calls, acting as a centralized AI gateway. It solves the Visual Studio 2026 compatibility issue by presenting a unified endpoint while routing traffic to your local vLLM instances.

**Create `litellm_config.yaml`:** Create this YAML file in your project root. The `general_settings` block securely locks down your server so it doesn't boot exposed to the public network without a master key.

model\_list:

  \- model\_name: qwen2.5-coder-7b

    litellm\_params:

      model: openai/Qwen/Qwen2.5-Coder-7B-Instruct

      api\_base: "http\://localhost:8000/v1"

      api\_key: "sk-local"

  \- model\_name: qwen3-omni-30b

    litellm\_params:

      model: openai/Qwen3-Omni-30B-A3B-Thinking

      api\_base: "http\://localhost:8001/v1"

      api\_key: "sk-local"

general\_settings:

  master\_key: "sk-local"

**Start the Gateway:** Open **Terminal 2**:

source vllm-env/bin/activate

\# Start LiteLLM proxy listening on all network interfaces, telemetry disabled for fast boot

uv run litellm \--config litellm\_config.yaml \--host 0.0.0.0 \--port 11434 \--telemetry=False

*(We intentionally use port 11434 to mimic Ollama. This ensures that any existing client configurations do not need to be updated).*

---

## 🔑 Hugging Face Authentication (Fixing Unauthenticated Errors)

If you receive an "unauthenticated requests" warning or hit a rate limit while pulling models from Hugging Face, you need to authenticate your terminal.

1. Create a free account at [Hugging Face](https://huggingface.co/).  
2. Navigate to **Settings \> Access Tokens** and generate a new **Read** token.  
3. Export the token into your Mac's environment before downloading or running the server.

export HF\_TOKEN="hf\_your\_token\_string\_here"

## 🧪 Testing the Servers

To verify `vllm-mlx` is processing requests natively (bypassing the router):

curl http\://localhost:8000/v1/chat/completions \\

  \-H "Content-Type: application/json" \\

  \-d '{

    "model": "mlx-community/Qwen2.5-Coder-7B-Instruct-4bit",

    "messages": \[

      {"role": "user", "content": "Write a Hello World in Python."}

    \]

  }'

---

🔌 Tier 3: Context Layer (Telemetry MCP Server)
To allow the local LLM to read local system logs and troubleshoot infrastructure without giving it raw bash access or sending trade secrets to the cloud, we bind it to our Custom MCP Server.

Because the Telemetry MCP server communicates over standard input/output (stdio), you do not need to start it manually. Your MCP-compatible client will automatically spawn and manage the server process in the background.

MCP Client Configuration:
To bind the MCP server to your local environment, add the execution command to your client's configuration file (e.g., Claude Desktop or your preferred MCP-enabled agent). Ensure mcpServers is formatted as an array:

JSON
{
  "mcpServers": [
    {
      "name": "TelemetryInspector",
      "command": "uv",
      "args": [
        "run",
        "--directory",
        "/Users/YOUR_MAC_USERNAME/Development/Telemetry_MCP_Server",
        "python",
        "src/telemetry_mcp_server/server.py"
      ]
    }
  ]
}
💻 Client Configurations
Because our API Gateway (LiteLLM) translates everything into the standard OpenAI format, client configuration is vastly simplified and can often be handled natively by the IDE.

For comprehensive instructions on connecting your specific environments (VS Code on Mac, VS Code on Windows, and Visual Studio 2026) using native integrations, please refer to the CLIENTS.md guide.