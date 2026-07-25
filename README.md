# 🛡️ Human-in-the-Loop AI Security Gateway

An event-driven, privacy-first middleware built to secure LLM communications. This gateway intercepts outgoing prompts, detects and masks sensitive data (PII, API keys, internal tokens), and enforces a manual Human-in-the-Loop (HITL) approval process before routing requests to external cloud AI providers.

## 🚀 Architecture & Core Features

*   **Zero-Trust Data Masking:** Dynamically scans payloads using Regex-based rules to vault sensitive information (e.g., `sk-` keys, emails, MedusaJS environment variables) and replaces them with sterile placeholders (`[SECRET_1]`).
*   **Asynchronous HITL Approval:** Pauses the execution state and sends an interactive webhook-based alert via Telegram. Cloud execution is strictly blocked until an authorized admin explicitly clicks `Approve`.
*   **Seamless Data Unmasking:** Once the cloud LLM (e.g., DeepSeek, Gemma via OpenRouter) returns the generated content, the gateway re-injects the original vaulted secrets into the final response, keeping the cloud provider completely blind to the actual sensitive data.
*   **Microservice Ready:** Built as a standalone API endpoint capable of servicing autonomous backend processes (such as automated product description generation for headless storefronts).

## 🛠️ Tech Stack
*   **Workflow Engine:** n8n (Self-hosted)
*   **Scripting:** JavaScript / Node.js
*   **AI Integration:** OpenRouter API (DeepSeek, etc.)
*   **Notifications:** Telegram Bot API
*   **Infrastructure:** Tested in an isolated local environment (Google Antigravity / OrbStack on macOS)

## 🔄 Workflow Diagram

![Core Gateway Architecture](https://github.com/Artfarreltuta/n8n-ai-security-gateway/raw/main/%5BCore%5D%20AI%20Security%20Gateway.png)

![Telegram Listener Architecture](https://github.com/Artfarreltuta/n8n-ai-security-gateway/raw/main/%5BHelper%5D%20Telegram%20Callback%20Listener.png)

1. `POST Request` ➡️ **Webhook** (Receives raw prompt from external system)
2. **Data Masking Engine** (Vaults secrets, sanitizes payload)
3. **Smart Router** ➡️ **Telegram Bot** (Sends approval request to Admin)
4. **Wait Node** (Suspends execution until webhook callback)
5. **Decision Switch** (Verifies `approve` or `block` status)
6. **OpenRouter Cloud** (Sends sanitized prompt to LLM)
7. **Data Unmasking Engine** (Restores vaulted data into the final output)
8. `200 OK` ➡️ Returns secure generated text to the requesting service.

## ⚙️ Installation & Setup

1. **Import Workflows:** Download the exported `.json` files from this repository and import them into your n8n instance.
2. **Configure Credentials:**
   *   Set up your OpenRouter API Key in the HTTP Request node.
   *   Add your Telegram Bot Token.
3. **Expose Webhooks:** Ensure your n8n instance can receive external callbacks (using Pinggy, Ngrok, or a reverse proxy if hosted locally).
4. **Customize RegEx:** Modify the `Data Masking Engine` node to add custom patterns for your specific infrastructure secrets.

## 💡 Use Case Example
When an automated script attempts to vibe-code or generate descriptions using a prompt like:
`"Generate a summary for my store. My admin key is sk-admin-123456789"`

The external LLM only ever receives:
`"Generate a summary for my store. My admin key is [SECRET_1]"`

Protecting your infrastructure from prompt leakage and unauthorized model training.
