# Client Configuration (OpenAI-Compatible API)

This guide details configuring your development environments on both Mac and Windows to seamlessly integrate with your local AI gateway. Because the inference infrastructure is abstracted behind a standard OpenAI-compatible API proxy, client configuration relies entirely on the native "Bring Your Own Key" (BYOK) capabilities of modern IDEs.

## Table of Contents

1. [VS Code (Local Mac) - Native Copilot Integration](#vs-code-local-mac)
2. [VS Code (Remote Windows) - Native Copilot Integration](#vs-code-remote-windows)
3. [Visual Studio 2026 (Remote Windows) - Third-Party Extension](#visual-studio-2026-remote-windows)
4. [General Setup and Troubleshooting](#general-setup-and-troubleshooting)

---

## 1. VS Code (Local Mac) - Native Copilot Integration

Since your Mac hosts the API gateway, it connects locally via `127.0.0.1` (localhost). This approach leverages VS Code's native Copilot UI, integrating your local models directly into the deepest workspace context without third-party extension bloat.

1. Open the native **Copilot Chat panel** on the left sidebar.
2. Click the model dropdown menu at the top of the chat window and select **Manage Models** (or Add Custom Endpoint).
3. In the UI configuration window that opens, insert the following JSON block to define your local API:

```json
{
    "name": "Local AI Gateway",
    "vendor": "customendpoint",
    "apiType": "openai",
    "models": [
        {
            "id": "qwen3-4b-thinking",
            "name": "Local Qwen3 4B (Gateway)",
            "url": "http://127.0.0.1:11434/v1/chat/completions",
            "toolCalling": true,
            "vision": false,
            "maxInputTokens": 32768,
            "maxOutputTokens": 4096
        }
    ]
}
```

4. **Save** the configuration.
5. You can now select **Local Qwen3 4B (Gateway)** directly from the native Copilot dropdown.

---

## 2. VS Code (Remote Windows) - Native Copilot Integration

Your Windows laptop acts as a client on the same local network, connecting to the Mac's API gateway over your LAN.

1. Open the native **Copilot Chat panel** in VS Code.
2. Click the model dropdown and select **Manage Models**.
3. Add the configuration block, substituting the URL with your Mac's LAN IP address:

```json
{
    "name": "Local AI Gateway",
    "vendor": "customendpoint",
    "apiType": "openai",
    "models": [
        {
            "id": "qwen3-4b-thinking",
            "name": "Local Qwen3 4B (Gateway)",
            "url": "http://<mac_ip_address>:11434/v1/chat/completions",
            "toolCalling": true,
            "vision": false,
            "maxInputTokens": 32768,
            "maxOutputTokens": 4096
        }
    ]
}
```

4. **Save** the configuration and select the model from the native Chat panel.

---

## 3. Visual Studio 2026 (Remote Windows)

Visual Studio 2026 currently restricts native "Custom Endpoint" additions to its built-in Copilot UI. To connect to your local OpenAI-compatible server, you will need a third-party extension that supports BYOK (Bring Your Own Key).

### Using Codeium

Codeium's "BYOM" (Bring Your Own Model) feature seamlessly accepts raw OpenAI endpoints without strict proprietary API validation checks.

1. Open **Visual Studio 2026**.
2. Navigate to **Extensions > Manage Extensions > Online**.
3. Search for and install **Codeium**. Restart Visual Studio.
4. In Codeium's settings (or setup wizard), locate "Local Models" or "BYOM Endpoint".
5. Enter your API Gateway URL: `http://<mac_ip_address>:11434/v1`

---

## 4. General Setup and Troubleshooting

* **Ensure Network Visibility:** Your API router (e.g., LiteLLM proxy) must be bound to `0.0.0.0` (e.g., `--host 0.0.0.0`) to accept connections from the Windows client. Both machines must be on the same local network.
* **Find Your Mac's IP:** On the Mac terminal, use `ipconfig getifaddr en0` (or `en1` for ethernet) to determine the IP address. Use this IP in your Windows configuration.
* **API Type Compatibility:** Always ensure `"apiType"` is set to `"openai"` in your VS Code settings, which ensures proper JSON schema formatting for tool calling and context payloads. 
* **Utility Models (Background Features):** If Copilot features like commit message generation or chat title summarization fail, open VS Code Settings (`Cmd + ,` or `Ctrl + ,`), search for `chat.utilityModel`, and set both `chat.utilityModel` and `chat.utilitySmallModel` to your new local gateway profile.
* **Context Window Constraints:** Ensure the `"maxInputTokens"` in your client configuration matches the maximum sequence length configured on your backend inference engine to prevent context-length rejection errors.