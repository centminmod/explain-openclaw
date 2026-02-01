# Privacy & Security

Clawdbot is designed for high-privacy and high-safety environments. Here is how it protects you and your data.

## 1. Data Locality
*   **Where is data stored?** All conversation history, sessions, and credentials are stored **locally** on the machine running the Gateway (in `~/.clawdbot/state/`).
*   **Cloud Interaction:** The only data that leaves your machine is the specific text/context sent to the LLM provider (Anthropic/OpenAI) for generation. Your WhatsApp keys, Telegram tokens, and local files **never** leave your server.

## 2. DM Pairing (Access Control)
By default, Clawdbot treats all incoming Direct Messages (DMs) as **untrusted**.

*   **The Problem:** If you connect a Telegram bot, *anyone* can find it and message it.
*   **The Solution**: Clawdbot's "Pairing Mode".
    1.  Stranger sends "Hi".
    2.  Clawdbot ignores the message or replies with a unique 6-digit code (if configured).
    3.  You, the owner, must run `clawdbot pairing approve <channel> <code>` on your server to allow that user.
    4.  Only then can they interact with the AI.

## 3. Sandboxing (Code Execution Safety)
The most powerful feature—allowing the AI to write and run code—is also the most dangerous.

*   **Owner Mode (Mac Mini)**: By default, the `main` session (you) runs code directly on the host. This is powerful (access to your files) but requires trust in the model.
*   **Sandbox Mode (VPS/Groups)**: You can configure `sandbox: "non-main"` or `sandbox: "all"`.
    *   **How it works**: When the agent wants to run a shell command, Clawdbot creates a **Docker container**.
    *   **Isolation**: The code runs inside this disposable container. It cannot see your host files, SSH keys, or other secrets unless you explicitly mount them.
    *   **Network**: You can even restrict the container's network access.

## 4. Input Sanitization
Clawdbot includes middleware to sanitize inputs before they reach the shell, but the Docker sandbox is the primary line of defense against "jailbreaks" where an LLM might be tricked into running malicious commands.

## 5. Tailscale "Hidden" Gateway
By using **Tailscale Serve**, your Gateway's admin dashboard is never exposed to the public internet. It is only accessible to devices on your private Tailscale network (Tailnet). This eliminates the risk of brute-force attacks on your admin panel.
