# 🧠 Local AI Model Directory

This document tracks the Large Language Models (LLMs) configured for use in this distributed coding environment.

All models are run via `llama.cpp` using the HuggingFace GGUF format to take advantage of the Apple M5 Pro's Metal GPU and Unified Memory architecture.


## 🚀 1. The Heavyweight (Deep Reasoning & Architecture)

**Model:** `Qwen3-Omni-30B-A3B-Thinking-Q8_0.gguf`

* **Size/Type:** 30 Billion Parameters (MoE: Active 3B), 8-bit Quantization.
* **RAM Footprint:** ~32 GB.
* **Best For:** Complex architectural questions, finding obscure bugs, writing unit tests for undocumented code, and deep logical reasoning.
* **Notes:** Because of its size, the context window is limited to 16k to prevent memory swapping on a 48GB machine.

**Start Command:**

./llama-server -m ~/Development/ai-models/Qwen3-Omni-30B-A3B-Thinking-Q8_0.gguf -c 16384 -ngl 99 -t 8 --host 0.0.0.0 --port 11434

JSON
{
    "name": "Local llama.cpp",
    "vendor": "customendpoint",
    "apiType": "openai",
    "models": [
        {
            "id": "qwen3-omni-30b",
            "name": "Local Qwen3 30B (llama.cpp)",
            "url": "[http://<mac ip address>:11434/v1/chat/completions](http://192.168.1.136:11434/v1/chat/completions)",
            "toolCalling": true,
            "vision": false,
            "maxInputTokens": 16384,
            "maxOutputTokens": 4096
        }
    ]
}

## ⚡ 2. The Speedy Assistant (Fast Chat & Medium Context)

**Model:** `qwen3-4b-thinking-2507-q8_0.gguf`

* **Size/Type:** 4 Billion Parameters, 8-bit Quantization.
* **RAM Footprint:** ~4.5 GB.
* **Best For:** Quick Q&A, battery-saving mode, or tasks requiring massive context windows.
* **Notes:** Leaves over 40GB of RAM free, allowing for massive 32k+ context windows to process dozens of files at once without slowing down.

**Start Command:**

./llama-server -m ~/Development/ai-models/qwen3-4b-thinking-2507-q8_0.gguf -c 32768 -ngl 99 --host 0.0.0.0 --port 11434

JSON
{
    "name": "Local llama.cpp",
    "vendor": "customendpoint",
    "apiType": "openai",
    "models": [
        {
            "id": "qwen3-4b-thinking",
            "name": "Local Qwen3 4B (llama.cpp)",
            "url": "[http://<mac ip address>:11434/v1/chat/completions](http://192.168.1.136:11434/v1/chat/completions)",
            "toolCalling": true,
            "vision": false,
            "maxInputTokens": 32768,
            "maxOutputTokens": 4096
        }
    ]
}


## 💻 3. The Autocomplete Specialist (Background IDE Usage)

**Model:** `Qwen2.5-Coder-7B-Instruct-Q4_K_M.gguf` *(Recommended)*

* **Size/Type:** 7 Billion Parameters, 4-bit Quantization.
* **RAM Footprint:** ~4.5 GB.
* **Best For:** Fill-In-the-Middle (FIM) tab-autocomplete as you type in VS Code or Visual Studio.
* **Notes:** Coder variants are highly specialized for syntax generation rather than conversational chat. Run this with a smaller context window for maximum millisecond response times.

**Start Command:**

./llama-server -m ~/Development/ai-models/Qwen2.5-Coder-7B-Instruct-Q4_K_M.gguf -c 4096 -ngl 99 -t 8 --host 0.0.0.0 --port 11434

JSON
{
    "name": "Local llama.cpp",
    "vendor": "customendpoint",
    "apiType": "openai",
    "models": [
        {
            "id": "qwen2.5-coder-7b",
            "name": "Local Qwen Coder 7B (llama.cpp)",
            "url": "[http://<mac ip address>:11434/v1/chat/completions](http://192.168.1.136:11434/v1/chat/completions)",
            "toolCalling": false,
            "vision": false,
            "maxInputTokens": 4096,
            "maxOutputTokens": 1024
        }
    ]
}