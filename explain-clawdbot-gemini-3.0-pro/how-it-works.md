# How Clawdbot Works: A Technical Overview

At its core, Clawdbot is a **Gateway**—a WebSocket server and control plane written in TypeScript/Node.js. It acts as a central hub connecting three main components: **Channels**, **Agents**, and **Tools**.

## The Architecture

```mermaid
graph TD
    User[User on WhatsApp/Telegram] -->|Message| Channel[Channel Adapter]
    Channel -->|Event| Gateway[Clawdbot Gateway]
    Gateway -->|Context| Agent[AI Agent (e.g., Pi)]
    Agent -->|Reasoning| LLM[LLM (Claude/GPT)]
    Agent -->|Tool Call| Gateway
    Gateway -->|Execution| Tools[Tools & Nodes]
    Tools -->|Result| Gateway
    Gateway -->|Response| Channel
    Channel -->|Reply| User
```

### 1. The Gateway (Control Plane)
The Gateway (`src/gateway/`) is the heart of the system. It:
*   Manages **Sessions**: Keeps track of conversation history and context for each user.
*   Handles **Routing**: Decides which agent should handle a message based on configuration.
*   Manages **Security**: Enforces permissions (e.g., "Can this user run shell commands?").

### 2. Channels (Inputs)
Clawdbot has built-in adapters (`src/channels/`) for various platforms.
*   **WhatsApp**: Uses `Baileys` to emulate a WhatsApp Web client.
*   **Telegram/Discord/Slack**: Connects via their respective Bot APIs.
*   **iMessage**: Interacts with the local `imsg` CLI on macOS.

### 3. The Agent (The Brain)
The default agent is **Pi** (Personal Intelligence). When it receives a message, it:
1.  Constructs a prompt with the conversation history and available tools.
2.  Sends this to an LLM (Large Language Model) like Anthropic's Claude or OpenAI's GPT-4.
3.  Decides whether to reply with text or **call a tool**.

### 4. Tools & Nodes (Capabilities)
This is where Clawdbot shines. It exposes "tools" to the agent:
*   **Local Tools**: `bash` (run commands), `fs` (file system), `browser` (puppeteer/chrome).
*   **Nodes**: Companion apps running on iOS/Android/macOS that expose device sensors (camera, location, screen recording) back to the Gateway via WebSocket.

### 5. The Plugin System
Located in `extensions/`, plugins allow dynamic loading of extra functionality. They can add new channels (like Matrix) or new capabilities without modifying the core codebase.

## Key Technical Concepts

*   **Sandboxing**: For safety, tool execution (like running code) for non-owner users happens inside **Docker containers**. This prevents a random user in a group chat from deleting files on your host machine.
*   **Tailscale Integration**: Clawdbot has native support for Tailscale `serve` and `funnel`, allowing you to securely expose the Gateway UI and API to the internet without opening public firewall ports.
*   **State Management**: It uses a local database (often SQLite or simple JSON stores) to persist sessions, allowing the bot to "remember" context across restarts.
