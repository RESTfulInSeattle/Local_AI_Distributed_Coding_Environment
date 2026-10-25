# Visual Studio and VS Code Client Configuration

This guide details configuring your development environments on both Mac and Windows to seamlessly integrate with local LLMs (via llama.cpp) and cloud providers. It focuses on the native "Bring Your Own Key" (BYOK) approach and utilizing the Continue/Continued extensions.

## Table of Contents

1. [VS Code (Local Mac) \- Native BYOK & Extension](#vs-code-local-mac)  
2. [VS Code (Remote Windows) \- Native BYOK & Extension](#vs-code-remote-windows)  
3. [Visual Studio 2026 (Remote Windows) \- Third-Party Extension](#visual-studio-2026-remote-windows)  
4. [General Setup and Troubleshooting](#general-setup-and-troubleshooting)

---

## 1\. VS Code (Local Mac)

Since your Mac hosts the inference engine (e.g., `llama-server`), it connects locally via `127.0.0.1` (localhost).

### Option A: Native VS Code BYOK (Recommended)

This approach leverages VS Code's native Copilot UI, eliminating the need for third-party extensions and bloat.

1. Open **VS Code Settings** (`Cmd + ,`).  
2. Select **User Settings (JSON)** (click the file icon in the top right).  
3. Append the following configuration to your `settings.json` file. Ensure you use the correct model ID and endpoint:

  "chat.languageModels": \[

    {

      "name": "Local llama.cpp",

      "vendor": "customendpoint",

      "apiType": "openai",

      "models": \[

        {

          "id": "qwen3-4b-thinking",

          "name": "Local Qwen3 4B (llama.cpp)",

          "url": "http\://localhost:11434/v1/chat/completions",

          "toolCalling": true,

          "vision": false,

          "maxInputTokens": 32768,

          "maxOutputTokens": 4096

        }

      \]

    }

  \]

*(Crucial: Ensure `"apiType"` is set to `"openai"` when using `llama.cpp`.)*

4. **Save** the `settings.json` file.  
5. **Restart** VS Code.  
6. Open the native **Chat panel**. Click the model dropdown; your local model should be listed.

### Option B: Using Continue.dev or Continued Extension

1. Install the **Continue** (or **Continued**) extension.  
2. Access the extension's settings (often by clicking the gear icon in the extension's sidebar or opening `~/.continue/config.json`).  
3. Add or modify the provider block:  
   * **Provider:** `OpenAI` (or `OpenAI Compatible`)  
   * **Base URL:** `http://localhost:11434/v1`  
   * **Model:** `qwen3` (or the specific model ID)  
   * **API Key:** `dummy-key` (required by the extension, ignored by `llama-server`)

*(Note: If Continue uses `config.yaml` instead of `.json`, the formatting will change accordingly).*

---

## 2\. VS Code (Remote Windows)

Your Windows laptop acts as a client on the same local network, connecting to the Mac's IP address.

### Option A: Native VS Code BYOK (Recommended)

1. Open **VS Code Settings** (`Ctrl + ,`).  
2. Select **User Settings (JSON)**.  
3. Add the configuration block, substituting the URL with your Mac's IP address:

  "chat.languageModels": \[

    {

      "name": "Local llama.cpp",

      "vendor": "customendpoint",

      "apiType": "openai",

      "models": \[

        {

          "id": "qwen3-4b-thinking",

          "name": "Local Qwen3 4B (llama.cpp)",

          "url": "http\://\<mac\_ip\_address\>:11434/v1/chat/completions",

          "toolCalling": true,

          "vision": false,

          "maxInputTokens": 32768,

          "maxOutputTokens": 4096

        }

      \]

    }

  \]

4. **Save** and **Restart** VS Code. Select the model from the native Chat panel.

### Option B: Using Continue.dev or Continued Extension

1. Install the **Continue** extension.  
2. Configure the provider:  
   * **Provider:** `OpenAI` (or `OpenAI Compatible`)  
   * **Base URL:** `http://<mac_ip_address>:11434/v1`  
   * **Model:** `qwen3`  
   * **API Key:** `dummy-key`

---

## 3\. Visual Studio 2026 (Remote Windows)

Visual Studio 2026 currently restricts native "Custom Endpoint" additions to its built-in Copilot UI. To connect to your local `llama.cpp` server, you'll need a third-party extension.

### Using Codeium (Recommended over LocalPilot)

Codeium's "BYOM" (Bring Your Own Model) feature seamlessly accepts raw OpenAI endpoints without strict Ollama API validation checks.

1. Open **Visual Studio 2026**.  
2. Navigate to **Extensions \> Manage Extensions \> Online**.  
3. Search for and install **Codeium**. Restart Visual Studio.  
4. In Codeium's settings (or setup wizard), locate "Local Models" or "BYOM Endpoint".  
5. Enter your `llama.cpp` URL: `http://<mac_ip_address>:11434/v1`

*(Alternative: If using LocalPilot, configure the "Ollama Base URL" to `http://<mac_ip_address>:11434/v1` and manually type the model ID, e.g., `qwen3-omni-30b`, as the test connection will fail due to missing Ollama-specific API routes.)*

---

## 4\. General Setup and Troubleshooting

* **Ensure Network Visibility:** The inference engine (Mac) must be bound to `0.0.0.0` (e.g., `--host 0.0.0.0` with `llama-server`) to accept connections from the Windows client. Both machines must be on the same local network.  
* **Find Your Mac's IP:** On the Mac terminal, use `ipconfig getifaddr en0` (or `en1` for ethernet) to determine the IP address. Use this IP in your Windows configuration.  
* **500 Internal Server Errors (VS Code Native):** If you receive a 500 error in VS Code's native chat, `llama-server` likely rejected the request (often due to complex tool-calling prompts). Try setting `"toolCalling": false` in your `settings.json` block.  
* **Context Window Limits:** Ensure the `"maxInputTokens"` in your client configuration does not exceed the `-c` (context window) value used when starting `llama-server`.  
* **API Type Compatibility:** Always ensure `"apiType"` is `"openai"` when configuring VS Code natively to communicate with `llama.cpp`, as it uses the OpenAI API standard, not Anthropic's ("messages").