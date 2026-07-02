Local AI Distributed Coding Environment

This repository documents the infrastructure and configuration for a distributed, locally hosted Large Language Model (LLM) coding environment.

By offloading the AI inference to a dedicated high-RAM host (Apple M5 Pro, 48GB Unified Memory), this setup provides privacy-first, zero-latency AI code assistance to multiple client machines across the local area network.


🏗️ Architecture

Inference Host: MacBook Pro (M5 Pro, 48GB RAM) running Ollama.
Clients:
    MacBook Pro (Local VS Code)
    MS Surface with Windows 11 (Remote VS Code)
IDE Integration: Continue.dev extension for VS Code.
Models:
    Chat & Reasoning: qwen2.5-coder:32b
    Tab-Autocomplete (FIM): qwen2.5-coder:7b


⚙️ Host Setup (MacBook Pro)

1.  Install Ollama for macOS.
2.  Download Models:
        ollama pull qwen2.5-coder:32b
        ollama pull qwen2.5-coder:7b
3.  Expose to Local Network.  By default Ollama binds to localhost. To allow the Windows machine to connect, set the host environment variable to 0.0.0.0.
        #Run this in the Mac terminal, then restart the Ollama application
        launchctl setenv OLLAMA_HOST "0.0.0.0"


💻 Client Configurations

Both clients use the config.json file to dictate how the Continue.dev extension routes requests. Store a copy of your configuration files in this repository under /configs.

Mac Configuration (Local)File location: ~/.continue/config.json{
  "models": [
    {
      "title": "Qwen Coder 32B (Chat)",
      "provider": "ollama",
      "model": "qwen2.5-coder:32b",
      "apiBase": "[http://127.0.0.1:11434](http://127.0.0.1:11434)"
    }
  ],
  "tabAutocompleteModel": {
    "title": "Qwen Coder 7B (Autocomplete)",
    "provider": "ollama",
    "model": "qwen2.5-coder:7b",
    "apiBase": "[http://127.0.0.1:11434](http://127.0.0.1:11434)"
  }
}

Windows 11 Configuration (Networked)
File location: %USERPROFILE%\.continue\config.jsonNote: Replace <MAC_IP_ADDRESS> with the actual local IP of the MacBook Pro (e.g., 192.168.1.15).

{
  "models": [
    {
      "title": "Qwen Coder 32B (Remote Chat)",
      "provider": "ollama",
      "model": "qwen2.5-coder:32b",
      "apiBase": "http://<MAC_IP_ADDRESS>:11434"
    }
  ],
  "tabAutocompleteModel": {
    "title": "Qwen Coder 7B (Remote Autocomplete)",
    "provider": "ollama",
    "model": "qwen2.5-coder:7b",
    "apiBase": "http://<MAC_IP_ADDRESS>:11434"
  }
}


🔒 Security Note for Future Development

Because Ollama does not have built-in authentication, binding to 0.0.0.0 exposes the inference engine to anyone on the immediate local Wi-Fi network. Ensure your home network is secured, and do not use this binding configuration on public Wi-Fi networks.

A basic layer of security will be added in a future iteration.