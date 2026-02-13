# Glossary (beginner)

## Gateway
The always-on local server that:
- accepts inbound messages from channels
- runs the agent
- sends replies back

Repo: see `README.md` (“Gateway is the control plane”).

## Channel
A connector to a messaging surface (WhatsApp/Telegram/Slack/Discord/etc.).
Each channel has its own login/token story, but they all feed into the same Gateway.

Docs: `docs/channels/*`.

## Agent
The “AI brain” that decides how to respond.
In practice it’s:
- a model (Claude/OpenAI/etc.)
- plus system prompts / templates
- plus tool permissions
- plus session memory

Docs: https://docs.clawd.bot/concepts/agent

## Session
A conversation state: chat history + metadata.
Sessions are stored on disk so your assistant has continuity.

Docs: https://docs.clawd.bot/concepts/session

## Pairing
An explicit approval step so random people cannot DM your bot.
When DM pairing is enabled (default on many channels), unknown senders get a code and their message is ignored until you approve.

Docs: `docs/start/pairing.md`.

## Tool
A capability beyond “just text generation”, like:
- executing a command
- controlling a browser
- reading/writing files
- calling a paired node/device

Docs: https://docs.clawd.bot/tools

## Node (device)
A companion device connected to the Gateway (macOS/iOS/Android).
Nodes can perform device-local actions (camera, notifications, etc.).

Docs: https://docs.clawd.bot/nodes
