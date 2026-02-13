# How Clawdbot Works

This guide explains the technical architecture in simple terms. You don't need to understand all of this to use Clawdbot, but it helps to know what's happening under the hood.

## The big picture

Clawdbot has three main parts:

```
┌─────────────────────────────────────────────────────────────────┐
│                         Messaging Apps                          │
│  WhatsApp │ Telegram │ Slack │ Discord │ Signal │ iMessage ...  │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                           The Gateway                           │
│                      (control plane)                            │
│                                                                 │
│  - Manages all connections                                      │
│  - Handles incoming/outgoing messages                           │
│  - Coordinates AI requests                                      │
│  - Stores sessions and history                                  │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                          Clients                                │
│                                                                 │
│  - macOS Menu Bar App                                           │
│  - CLI (command line)                                           │
│  - WebChat (browser interface)                                  │
│  - iOS/Android Apps                                             │
└─────────────────────────────────────────────────────────────────┘
```

## The Gateway: the heart of Clawdbot

The **Gateway** is the central component. Think of it as the "brain" that coordinates everything:

### What the Gateway does:

1. **Connects to messaging apps** — Maintains connections to WhatsApp, Telegram, Slack, etc.

2. **Receives your messages** — When you send a message, the Gateway receives it

3. **Talks to the AI** — Sends your message to the AI provider (Claude, GPT-4, etc.)

4. **Gets the response** — Receives the AI's reply

5. **Sends it back** — Routes the response back through the appropriate channel

6. **Remembers context** — Stores conversation history in sessions

### How it connects

The Gateway listens on **WebSocket port 18789** (by default):

- Clients connect to `ws://127.0.0.1:18789` locally
- Remote access via SSH tunnel or Tailscale
- All communication uses a structured protocol (JSON messages)

### Where data lives

```
~/.clawdbot/
├── clawdbot.json          # Your configuration
├── credentials/           # Stored API keys and tokens
├── sessions/              # Conversation history (per chat)
├── agents/                # Agent-specific data
├── pairing/               # Device pairing info
└── ...
```

Everything stays in this directory — nothing is sent elsewhere except your messages to the AI provider.

## The Agent system

The **Agent** is the AI "brain" that processes your messages. Clawdbot uses the **Pi agent framework**:

### How the agent thinks:

```
1. You send a message
        ↓
2. Gateway receives it and creates a "session"
        ↓
3. Agent analyzes your message
        ↓
4. Agent decides which tools to use
        ↓
5. Agent calls tools (browser, file read, command, etc.)
        ↓
6. Agent synthesizes the results
        ↓
7. Agent sends back a response
        ↓
8. Response delivered to your chat
```

### Available tools

The agent has access to various "tools" it can use:

| Tool | What it does |
|------|--------------|
| `bash` | Run terminal commands |
| `read` | Read files from disk |
| `write` | Write or edit files |
| `browser` | Control a web browser |
| `canvas` | Create visual content |
| `camera` | Take photos (on devices with cameras) |
| `location` | Get device location |
| `sessions_*` | Coordinate between different conversations |

### Session model

Each conversation has its own **session**:

- **Direct message** = One session (e.g., `agent:main:whatsapp:dm:+1234567890`)
- **Group chat** = One session (e.g., `agent:main:telegram:group:-100123456`)
- **CLI interaction** = One session (`agent:main:cli:main`)

Sessions:
- Store conversation history
- Maintain context across messages
- Can be reset with `/new` or `/reset`
- Can be compacted to save tokens

## Message flow: step by step

Here's what happens when you send "What's the weather?" in WhatsApp:

```
1. WhatsApp (your phone)
   │
   │ "What's the weather?"
   │
   ▼
2. WhatsApp servers
   │
   ▼
3. Clawdbot Gateway (Baileys connection)
   │
   │ Receives message
   │ Identifies sender
   │ Checks if sender is allowed
   │
   ▼
4. Gateway Router
   │
   │ Routes to appropriate agent
   │ Creates/updates session
   │
   ▼
5. Pi Agent
   │
   │ Analyzes message
   │ Decides to use browser tool
   │ Calls browser to search weather
   │
   ▼
6. AI Provider (Anthropic/OpenAI)
   │
   │ Processes request
   │ Returns response
   │
   ▼
7. Gateway
   │
   │ Formats response
   │
   ▼
8. WhatsApp (via Baileys)
   │
   ▼
9. Your phone
   │
   │ "It's currently 72°F and sunny..."
```

All of this happens in seconds.

## Remote access

You can run the Gateway on one machine and control it from another:

### SSH Tunnel (simplest)

```bash
# On your laptop, forward the gateway port
ssh -N -L 18789:127.0.0.1:18789 user@server
```

Now your laptop can connect to `ws://127.0.0.1:18789` and it actually reaches the remote server.

### Tailscale (recommended)

Tailscale creates a private network between your devices:

- Enable Tailscale Serve on the server
- Access the Gateway from anywhere on your tailnet
- Optional Funnel for public HTTPS (with password)

## Nodes (iOS, Android, macOS)

Clawdbot can pair with **mobile devices** as "nodes":

- iOS app connects as a node
- Android app connects as a node
- macOS app can connect in "node mode"

Nodes can do things the main Gateway can't:

- Take photos with the phone camera
- Get GPS location
- Record screen
- Show a visual "Canvas"
- Receive notifications

The Gateway can invoke these capabilities via the `node.invoke` protocol.

## Key files and what they do

```
clawdbot/
├── dist/
│   ├── gateway/           # The Gateway server
│   ├── cli/               # Command-line interface
│   ├── whatsapp/          # WhatsApp (Baileys) integration
│   ├── telegram/          # Telegram (grammY) integration
│   ├── slack/             # Slack integration
│   ├── discord/           # Discord integration
│   ├── signal/            # Signal integration
│   ├── imessage/          # iMessage integration (macOS)
│   ├── agents/            # Pi agent framework
│   ├── sessions/          # Session management
│   ├── browser/           # Browser control
│   ├── canvas/            # Canvas/A2UI host
│   ├── config/            # Configuration handling
│   ├── security/          # Security & auditing
│   └── ...
├── apps/
│   ├── macos/             # macOS menu bar app
│   ├── ios/               # iOS node app
│   └── android/           # Android node app
└── extensions/            # Channel plugins (Teams, Matrix, etc.)
```

## WebSocket protocol (simplified)

Communication with the Gateway uses WebSocket:

### Connection

```json
{
  "type": "req",
  "id": "1",
  "method": "connect",
  "params": {
    "role": "client",
    "version": "2026.1.25"
  }
}
```

### Sending a message

```json
{
  "type": "req",
  "id": "2",
  "method": "send",
  "params": {
    "message": "Hello!",
    "sessionKey": "agent:main:whatsapp:dm:+1234567890"
  }
}
```

### Agent request

```json
{
  "type": "req",
  "id": "3",
  "method": "agent",
  "params": {
    "message": "Summarize this file",
    "sessionKey": "agent:main:cli:main"
  }
}
```

### Server events

```json
{
  "type": "event",
  "event": "agent",
  "payload": {
    "type": "content",
    "content": "Here's the summary..."
  }
}
```

## Security model

### Local connections

- Same machine (127.0.0.1) → trusted by default
- Can require password if desired

### Remote connections

- Must go through tunnel or VPN
- Device pairing required
- Optional password authentication

### Channel security

- Each channel (WhatsApp, Telegram, etc.) has an allowlist
- Default: DM pairing required (unknown senders get a pairing code)
- Groups: mention-gated by default

### Tool security

- Main session (you): Full tool access
- Group sessions: Restricted by sandbox settings
- Docker sandboxing available for untrusted contexts

## Next steps

Now that you understand how Clawdbot works:

- [Install it](./installation.md) — Get it running on your machine
- [Configure it](./configuration.md) — Set it up your way
- Learn about [privacy/security](./privacy-security.md)
- Choose your setup: [Mac Mini](./usage-mac-mini.md) or [VPS](./usage-vps.md)
