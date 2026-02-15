# CLI commands (plain English, beginner-first)

## Table of contents (Explain OpenClaw)

- [Home (README)](../README.md)
- Plain English
  - [What is OpenClaw?](./what-is-clawdbot.md)
  - [Glossary](./glossary.md)
  - [CLI commands](./cli-commands.md)
- Technical
  - [Architecture](../02-technical/architecture.md)
  - [Repo map](../02-technical/repo-map.md)
- Privacy + safety
  - [Threat model](../04-privacy-safety/threat-model.md)
  - [Hardening checklist](../04-privacy-safety/hardening-checklist.md)
- Deployment
  - [Standalone Mac mini](../03-deploy/standalone-mac-mini.md)
  - [Isolated VPS](../03-deploy/isolated-vps.md)
  - [Cloudflare Moltworker](../03-deploy/cloudflare-moltworker.md)
- Optimizations
  - [Cost + token optimization](../06-optimizations/cost-token-optimization.md)
- Reference
  - [Commands + troubleshooting](../99-reference/commands-and-troubleshooting.md)

---

## What this guide is

This is a **beginner-friendly walkthrough** of every `openclaw` CLI command. It's organized by *what you're trying to do* (not alphabetically), and each section explains:

- what the command does in plain English
- when you'd actually use it
- a practical example you can copy/paste
- a table of options (for the major commands)

**This is not a replacement for the canonical reference.** For terse, complete option lists, see:
- [CLI reference](../../docs/cli/index.md) (canonical, 1000+ lines)
- [Commands + troubleshooting](../99-reference/commands-and-troubleshooting.md) (copy/paste quick ref)

---

## Command tree (visual overview)

This is the full tree of everything `openclaw` can do. Don't worry about memorizing it; the rest of this guide walks through each group.

```
openclaw [--dev] [--profile <name>] <command>
  setup                              # first-time init
  onboard                            # interactive wizard
  configure                          # config wizard (models, channels, skills)
  config get|set|unset               # read/write config values
  doctor                             # health checks + quick fixes

  status                             # session health + recipients
  health                             # gateway health check
  sessions                           # list stored conversations
  dashboard                          # open web dashboard
  logs                               # tail gateway logs

  gateway run                        # run gateway (foreground)
  gateway install|uninstall          # service lifecycle
  gateway start|stop|restart         # service control
  gateway status|health|probe        # gateway diagnostics
  gateway usage-cost                 # usage cost summary
  gateway call|discover              # RPC helpers

  channels list|status|logs          # inspect channels
  channels add|remove                # manage channels
  channels login|logout              # channel auth (WhatsApp Web etc.)

  message send|poll|react|...        # outbound messaging + moderation
  message thread|emoji|sticker|...   # threads, reactions, stickers
  agent                              # single agent turn
  agents list|add|delete             # manage isolated agents
  acp                                # IDE bridge (ACP protocol)

  models status|list                 # view model config
  models set|set-image               # set default models
  models scan                        # discover available models
  models auth add|setup-token|...    # manage model provider auth
  models aliases|fallbacks|...       # aliases + fallback chains

  security audit                     # security scan
  approvals get|set|allowlist        # exec approval policy

  cron status|list|add|edit|rm|...   # scheduled jobs
  hooks list|info|check|enable|...   # hook management
  webhooks gmail setup|run           # Gmail Pub/Sub hooks

  browser status|start|stop          # browser lifecycle
  browser open|focus|close|tabs      # tab management
  browser screenshot|snapshot        # inspection
  browser navigate|click|type|...    # automation actions

  nodes status|list|invoke|run|...   # gateway-side node commands
  nodes camera|canvas|screen|...     # device capabilities
  node run|install|start|stop|...    # headless node host
  devices                            # device pairing + tokens
  pairing list|approve               # DM pairing approvals

  memory status|index|search         # vector memory
  docs                               # search live docs

  plugins list|install|enable|...    # plugin management
  skills list|info|check             # skill inspection

  sandbox list|recreate|explain      # sandbox management
  system event|heartbeat|presence    # system events (RPC)
  reset                              # reset config/state
  uninstall                          # remove gateway + data
  update                             # update CLI

  directory self|peers|groups         # directory lookups (contacts, groups)
  daemon install|start|stop|...      # gateway service (legacy alias)

  dns setup                          # discovery DNS helper
  tui                                # terminal UI
  completion                         # shell completion script
```

Plugins can add extra top-level commands (e.g., `openclaw voicecall`).

---

## 1. Getting started

These are the commands you run once (or rarely) when you first install OpenClaw or need to reconfigure from scratch.

### `openclaw setup`

**What it does:** Creates your config file and workspace directory. Think of it as "unboxing" — it sets up the folder structure OpenClaw needs before anything else can run.

**When would I use this?** Right after installing OpenClaw for the first time, or if you deleted your config and want a fresh start.

```bash
openclaw setup
```

| Option | What it does |
|--------|-------------|
| `--workspace <dir>` | Where the agent keeps working files (default `~/.openclaw/workspace`) |
| `--wizard` | Run the interactive onboarding wizard |
| `--non-interactive` | Skip all prompts (use defaults or flags) |
| `--mode <local\|remote>` | Local gateway vs. connecting to a remote one |
| `--remote-url <url>` | URL of a remote Gateway |
| `--remote-token <token>` | Auth token for a remote Gateway |

Docs: https://docs.openclaw.ai/start/getting-started

---

### `openclaw onboard`

**What it does:** The all-in-one interactive wizard. It walks you through setting up your gateway, workspace, model provider credentials, channels, and skills — in one go. This is the **recommended way to get started**.

**When would I use this?** First time setup. It's the "guided tour" that replaces running `setup`, `configure`, `models auth`, `channels add`, and `gateway install` one by one.

```bash
# The one command most people should run first:
openclaw onboard --install-daemon
```

The `--install-daemon` flag tells it to also install the Gateway as a background service (launchd on macOS, systemd on Linux) so it starts automatically.

<details>
<summary>Full options table (click to expand)</summary>

| Option | What it does |
|--------|-------------|
| `--workspace <dir>` | Agent workspace path |
| `--reset` | Wipe config + credentials + sessions before starting |
| `--non-interactive` | Skip prompts; requires explicit flags |
| `--mode <local\|remote>` | Local gateway or connect to remote |
| `--flow <quickstart\|advanced\|manual>` | How detailed the wizard is (`manual` = alias for `advanced`) |
| `--auth-choice <provider>` | Which model provider to set up (e.g., `setup-token`, `openai-api-key`, `gemini-api-key`, `custom-api-key`) |
| `--token-provider <id>` | Provider id for `--auth-choice token` |
| `--token <token>` | Token value for `--auth-choice token` |
| `--anthropic-api-key <key>` | Anthropic API key |
| `--openai-api-key <key>` | OpenAI API key |
| `--openrouter-api-key <key>` | OpenRouter API key |
| `--gemini-api-key <key>` | Gemini API key |
| `--zai-api-key <key>` | z.ai API key |
| `--moonshot-api-key <key>` | Moonshot API key |
| `--kimi-code-api-key <key>` | Kimi Code API key |
| `--minimax-api-key <key>` | MiniMax API key |
| `--opencode-zen-api-key <key>` | OpenCode Zen API key |
| `--ai-gateway-api-key <key>` | AI Gateway API key |
| `--custom-base-url <url>` | Custom provider base URL |
| `--custom-model-id <id>` | Custom provider model ID |
| `--custom-api-key <key>` | Custom provider API key (or set `CUSTOM_API_KEY` env) |
| `--custom-provider-id <id>` | Custom provider identifier |
| `--custom-compatibility <openai\|anthropic>` | API compatibility mode (default `openai`) |
| `--gateway-port <port>` | Gateway port (default 18789) |
| `--gateway-bind <mode>` | `loopback\|lan\|tailnet\|auto\|custom` |
| `--gateway-auth <mode>` | `token\|password` |
| `--gateway-token <token>` | Gateway auth token |
| `--gateway-password <password>` | Gateway auth password |
| `--remote-url <url>` | Remote Gateway URL |
| `--remote-token <token>` | Remote Gateway token |
| `--tailscale <mode>` | `off\|serve\|funnel` |
| `--install-daemon` | Install Gateway as background service |
| `--no-install-daemon` | Skip daemon install (alias: `--skip-daemon`) |
| `--daemon-runtime <node\|bun>` | Runtime for the daemon |
| `--skip-channels` | Skip channel setup |
| `--skip-skills` | Skip skill setup |
| `--skip-health` | Skip health check |
| `--skip-ui` | Skip UI setup |
| `--node-manager <npm\|pnpm\|bun>` | Package manager (pnpm recommended) |
| `--json` | Machine-readable output |

</details>

Non-interactive example (custom provider):

```bash
export CUSTOM_API_KEY="your-api-key-here"
openclaw onboard --non-interactive --install-daemon \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "my-model" \
  --custom-compatibility openai
```

Docs: https://docs.openclaw.ai/start/wizard

---

### `openclaw configure`

**What it does:** An interactive wizard for changing your config *after* initial setup. Covers models, channels, skills, and gateway settings.

**When would I use this?** You're already running OpenClaw and want to add a new channel, swap model providers, or tweak gateway settings — without hand-editing JSON.

```bash
openclaw configure
```

---

### `openclaw doctor`

**What it does:** Runs health checks on your config, gateway, and legacy services, and offers quick fixes. Think of it as a "check engine light" that also knows how to fix common problems.

**When would I use this?** Something isn't working and you're not sure why. Or after an upgrade to make sure everything migrated properly.

```bash
openclaw doctor
```

| Option | What it does |
|--------|-------------|
| `--no-workspace-suggestions` | Skip workspace memory hints |
| `--yes` | Accept defaults without prompting |
| `--non-interactive` | Skip prompts; apply safe migrations only |
| `--deep` | Scan system services for extra gateway installs |

---

## 2. Day-to-day status

These commands tell you what's happening right now. Think of them as your dashboard instruments.

### `openclaw status`

**What it does:** Shows your linked session health, recent recipients, and (optionally) model provider usage. This is your go-to "is everything okay?" command.

**When would I use this?** Any time you want a quick pulse check, or when debugging why messages aren't going through.

```bash
openclaw status          # quick overview
openclaw status --all    # full diagnosis (pasteable for support)
openclaw status --deep   # also probe channel health
openclaw status --usage  # show model provider usage/quota
```

| Option | What it does |
|--------|-------------|
| `--json` | Machine-readable output |
| `--all` | Full diagnosis; read-only, pasteable |
| `--deep` | Probe channels |
| `--usage` | Show model provider usage/quota |
| `--timeout <ms>` | Probe timeout |
| `--verbose` / `--debug` | Extra detail |

---

### `openclaw health`

**What it does:** Fetches health info from the running Gateway. A lighter check than `status` — just "is the gateway alive?"

```bash
openclaw health
openclaw health --json
```

| Option | What it does |
|--------|-------------|
| `--json` | Machine-readable output |
| `--timeout <ms>` | Probe timeout |
| `--verbose` | Extra detail |

---

### `openclaw sessions`

**What it does:** Lists stored conversation sessions. Each session is a conversation thread with history and metadata.

**When would I use this?** To see which conversations the bot has been having, or to find a specific session ID for debugging.

```bash
openclaw sessions
openclaw sessions --active 60   # only sessions active in last 60 minutes
```

| Option | What it does |
|--------|-------------|
| `--json` | Machine-readable output |
| `--verbose` | Extra detail |
| `--store <path>` | Custom session store path |
| `--active <minutes>` | Filter to recently active sessions |

---

### `openclaw dashboard`

**What it does:** Opens the web-based control UI in your browser. Prints a tokenized URL so you don't have to manually add auth.

**When would I use this?** When you want the visual dashboard instead of the CLI.

```bash
openclaw dashboard
```

---

### `openclaw logs`

**What it does:** Tails the Gateway's log file via RPC. In a terminal you get colorized, structured output; pipe it and you get plain text.

**When would I use this?** Debugging what the Gateway is doing in real time — watching messages flow, seeing errors, tracing agent turns.

```bash
openclaw logs --follow     # live tail
openclaw logs --limit 200  # last 200 lines
openclaw logs --json       # line-delimited JSON
openclaw logs --plain      # no colors
```

---

## 3. Configuration

### `openclaw config`

**What it does:** Read and write individual config values without opening the JSON file. Running `openclaw config` alone launches the interactive wizard (same as `configure`).

**When would I use this?** Quick one-liner changes, or scripting config updates.

```bash
# Read a value
openclaw config get gateway.port

# Set a value (JSON5 or raw string)
openclaw config set gateway.bind "loopback"

# Remove a value
openclaw config unset channels.telegram.default.token
```

| Subcommand | What it does |
|------------|-------------|
| `config get <path>` | Print a config value (dot/bracket path) |
| `config set <path> <value>` | Set a value |
| `config unset <path>` | Remove a value |

---

## 4. Gateway management

The Gateway is the always-on process that makes everything work. These commands let you run it, manage it as a service, and poke at it for diagnostics.

### `openclaw gateway` / `openclaw gateway run`

**What it does:** Runs the Gateway **in the foreground** in your terminal. You'll see log output directly. This is useful for development or debugging; for production use, install it as a service instead.

**When would I use this?** Testing, debugging, or when you want to watch the Gateway's output live.

```bash
openclaw gateway
openclaw gateway run --verbose
```

| Option | What it does |
|--------|-------------|
| `--port <port>` | Override the port (default 18789) |
| `--bind <mode>` | `loopback\|tailnet\|lan\|auto\|custom` |
| `--token <token>` | Auth token |
| `--auth <mode>` | `token\|password` |
| `--password <password>` | Auth password |
| `--tailscale <mode>` | `off\|serve\|funnel` |
| `--allow-unconfigured` | Start even without full config |
| `--force` | Kill existing listener on the port |
| `--verbose` | Extra logging |
| `--ws-log <auto\|full\|compact>` | WebSocket log verbosity |
| `--raw-stream` | Dump raw stream to stdout |
| `--raw-stream-path <path>` | Save raw stream output to file |
| `--reset` | Reset dev config + credentials + sessions + workspace (dev mode only) |
| `--claude-cli-logs` | Filter logs to agent/claude-cli only |
| `--compact` | Alias for `--ws-log compact` |

---

### Gateway service lifecycle

Think of these as "install/start/stop the background daemon."

```bash
openclaw gateway install    # register as a system service
openclaw gateway uninstall  # remove the system service
openclaw gateway start      # start the service
openclaw gateway stop       # stop the service
openclaw gateway restart    # restart the service
openclaw gateway status     # check if it's running
```

**When would I use these?** After `onboard` sets up the service, you'll mostly use `restart` (after config changes) and `status` (to check health).

`gateway install` options:

| Option | What it does |
|--------|-------------|
| `--port <port>` | Port to bind |
| `--runtime <node\|bun>` | Runtime (Node recommended; bun has known bugs) |
| `--token <token>` | Auth token |
| `--force` | Overwrite existing service |
| `--json` | Machine-readable output |

`gateway status` options:

| Option | What it does |
|--------|-------------|
| `--no-probe` | Skip the live RPC probe |
| `--deep` | System-level scans for extra installs |
| `--json` | Machine-readable output |

All service commands support `--json` for scripting.

---

### `openclaw gateway usage-cost`

**What it does:** Fetches a usage cost summary from session logs. Shows how much you've spent on model API calls over a given period.

**When would I use this?** To check your AI provider spending, audit costs, or track usage trends.

```bash
openclaw gateway usage-cost              # last 30 days (default)
openclaw gateway usage-cost --days 7     # last 7 days
openclaw gateway usage-cost --json       # machine-readable output
```

| Option | What it does |
|--------|-------------|
| `--days <days>` | Number of days to include (default 30) |
| `--json` | Machine-readable output |

All gateway RPC options also apply: `--url`, `--token`, `--timeout`, `--expect-final`.

---

### Gateway RPC helpers

These are lower-level commands for talking to the Gateway's internal RPC interface. Most users won't need them day-to-day.

```bash
openclaw gateway call <method> --params '{"key": "value"}'
openclaw gateway health
openclaw gateway probe
openclaw gateway discover
```

**When would I use these?** Scripting, automation, or advanced debugging.

Common RPCs you might use with `gateway call`:
- `config.apply` — validate + write config + restart
- `config.patch` — merge a partial update + restart
- `update.run` — run update + restart

All RPC commands accept: `--url`, `--token`, `--password`, `--timeout`, `--expect-final`.

When you pass `--url`, the CLI does *not* auto-apply config credentials — you must include `--token` or `--password` explicitly.

---

## 5. Channels

Channels are OpenClaw's "phone lines" — the connections between the Gateway and messaging platforms like WhatsApp, Telegram, Discord, Slack, Google Chat, Signal, iMessage, and MS Teams (plus plugins for Mattermost and more).

### `openclaw channels list`

**What it does:** Shows all configured channels and their auth profiles.

```bash
openclaw channels list
openclaw channels list --json
openclaw channels list --no-usage   # skip model provider usage snapshots
```

---

### `openclaw channels status`

**What it does:** Checks if each channel is reachable and healthy. Prints warnings with suggested fixes when it detects common misconfigurations.

```bash
openclaw channels status
openclaw channels status --probe   # run extra checks
```

Tip: For gateway-level health, use `openclaw health` or `openclaw status --deep` instead.

---

### `openclaw channels add`

**What it does:** Connect a new messaging channel. Without flags it runs a wizard; with flags it goes straight to non-interactive mode.

**When would I use this?** Adding a Telegram bot, Discord bot, Slack workspace, etc.

```bash
# Interactive:
openclaw channels add

# Non-interactive:
openclaw channels add --channel telegram --account alerts \
  --name "Alerts Bot" --token $TELEGRAM_BOT_TOKEN

openclaw channels add --channel discord --account work \
  --name "Work Bot" --token $DISCORD_BOT_TOKEN
```

---

### `openclaw channels remove`

**What it does:** Disable a channel (or fully remove its config entries).

```bash
openclaw channels remove --channel discord --account work           # disable
openclaw channels remove --channel discord --account work --delete  # remove config
```

---

### `openclaw channels login` / `logout`

**What it does:** Interactive login for channels that require it (WhatsApp Web), and logout to end a session.

```bash
openclaw channels login                              # defaults to WhatsApp
openclaw channels login --channel whatsapp --verbose
openclaw channels logout --channel whatsapp
```

---

### `openclaw channels logs`

**What it does:** Show recent channel-specific logs from the Gateway log file.

```bash
openclaw channels logs                        # all channels
openclaw channels logs --channel telegram     # just Telegram
openclaw channels logs --lines 500            # more history
```

---

### Channel common options

These options apply across most `channels` subcommands:

| Option | What it does |
|--------|-------------|
| `--channel <name>` | Core: `whatsapp\|telegram\|discord\|irc\|googlechat\|slack\|signal\|imessage` — Plugins: `msteams\|mattermost` |
| `--account <id>` | Channel account id (default `default`) |
| `--name <label>` | Display name for the account |

---

### `openclaw directory`

**What it does:** Directory lookups (self, peers, groups) for channels that support it. Not all channels implement directory features — if yours doesn't, the command will tell you.

**When would I use this?** To look up contacts, list groups, or see group members on a specific channel.

```bash
openclaw directory self --channel telegram       # show your account info
openclaw directory peers list --channel telegram  # list contacts
openclaw directory groups list                    # list groups
openclaw directory groups members --group-id 123  # list group members
```

| Subcommand | What it does |
|------------|-------------|
| `self` | Show the current account user |
| `peers list` | List peers/contacts (`--query`, `--limit`) |
| `groups list` | List groups (`--query`, `--limit`) |
| `groups members` | List group members (`--group-id <id>`, `--limit`) |

Common options: `--channel <name>`, `--account <id>`, `--json`.

---

### `openclaw daemon` (legacy alias)

**What it does:** Same as `openclaw gateway service` commands (`install`, `uninstall`, `start`, `stop`, `restart`, `status`). This exists as a **legacy alias** from before these were moved under `gateway`.

**When would I use this?** If you have existing scripts using `openclaw daemon`. For new work, prefer `openclaw gateway install/start/stop/...` instead.

```bash
openclaw daemon install    # same as: openclaw gateway install
openclaw daemon start      # same as: openclaw gateway start
openclaw daemon status     # same as: openclaw gateway status
```

---

## 6. Messaging + agent

This is where you send messages and run agent turns from the command line.

### `openclaw message`

**What it does:** Unified outbound messaging. You can send messages, create polls, add reactions, manage threads, upload stickers, moderate users, and more — all from the CLI.

**When would I use this?** Sending messages from scripts/cron jobs, testing channel delivery, or managing chat features.

```bash
# Send a text message
openclaw message send --target +15555550123 --message "Hi there"

# Create a poll in Discord
openclaw message poll --channel discord --target channel:123 \
  --poll-question "Snack?" --poll-option Pizza --poll-option Sushi

# React to a message
openclaw message react --channel telegram --target chat:456 \
  --message-id 789 --emoji thumbsup
```

Full subcommands:

| Subcommand | What it does |
|------------|-------------|
| `send` | Send a text message |
| `poll` | Create a poll |
| `react` | Add a reaction |
| `reactions` | List reactions |
| `read` | Mark as read |
| `edit` | Edit a sent message |
| `delete` | Delete a message |
| `pin` / `unpin` | Pin/unpin messages |
| `pins` | List pinned messages |
| `permissions` | Check permissions |
| `search` | Search messages |
| `timeout` / `kick` / `ban` | Moderation actions |
| `thread create\|list\|reply` | Thread management |
| `emoji list\|upload` | Custom emoji |
| `sticker send\|upload` | Stickers |
| `role info\|add\|remove` | Role management |
| `channel info\|list` | Channel info |
| `member info` | Member info |
| `voice status` | Voice channel status |
| `event list\|create` | Calendar events |

---

### `openclaw agent`

**What it does:** Run a single agent turn via the Gateway (or locally with `--local`). This sends a message to the agent, gets a response, and exits. Useful for scripting.

**When would I use this?** Running the agent from a script, testing prompts, or automating single-turn interactions without a chat channel.

```bash
openclaw agent --message "What's the weather in Tokyo?"
openclaw agent --message "Summarize this doc" --local   # embedded, no gateway
openclaw agent --message "Hello" --to +15555550123 --deliver  # deliver via channel
```

| Option | What it does |
|--------|-------------|
| `--message <text>` | **Required.** The message to send |
| `--to <dest>` | Session key and optional delivery target |
| `--session-id <id>` | Explicit session id |
| `--thinking <level>` | `off\|minimal\|low\|medium\|high\|xhigh` (GPT-5.2 + Codex only) |
| `--verbose <on\|full\|off>` | Output verbosity |
| `--channel <name>` | Target channel for delivery |
| `--local` | Run embedded (no gateway needed) |
| `--deliver` | Actually deliver the response via channel |
| `--json` | Machine-readable output |
| `--timeout <seconds>` | Agent turn timeout |

---

### `openclaw agents`

**What it does:** Manage isolated agents. Each agent has its own workspace, auth, model config, and channel bindings.

**When would I use this?** Multi-agent setups where different agents serve different purposes (e.g., one for work Slack, one for personal Telegram).

```bash
openclaw agents list
openclaw agents list --bindings    # show channel bindings
openclaw agents add "work-agent" --workspace ~/work-ai --bind slack
openclaw agents delete work-agent --force
```

| Subcommand | What it does |
|------------|-------------|
| `list` | List all agents (`--json`, `--bindings`) |
| `add [name]` | Add a new agent (wizard or `--non-interactive` with flags) |
| `delete <id>` | Remove an agent and its workspace (`--force`, `--json`) |

`agents add` options: `--workspace <dir>`, `--model <id>`, `--agent-dir <dir>`, `--bind <channel[:accountId]>` (repeatable), `--non-interactive`, `--json`.

---

### `openclaw acp`

**What it does:** Runs the ACP (Agent Control Protocol) bridge that connects IDEs to the Gateway. This is how editor extensions talk to your OpenClaw agent.

**When would I use this?** If you use an IDE extension that integrates with OpenClaw (e.g., VS Code, JetBrains).

```bash
openclaw acp
```

See the full options at https://docs.openclaw.ai/cli/acp.

---

## 7. Models

Models are the "brain" behind your assistant. These commands manage which AI models you use, how you authenticate with providers, and what happens when your primary model is unavailable.

### `openclaw models status`

**What it does:** Show your current model configuration, auth profiles, and OAuth expiry status. Running `openclaw models` alone is the same as `models status`.

```bash
openclaw models               # alias for models status
openclaw models status
openclaw models status --probe # live-test configured auth (may consume tokens)
```

| Option | What it does |
|--------|-------------|
| `--json` / `--plain` | Output format |
| `--check` | Exit code: 1 = expired/missing, 2 = expiring |
| `--probe` | Live probe of auth profiles (uses tokens!) |
| `--probe-provider <name>` | Probe specific provider |
| `--probe-profile <id>` | Probe specific profile(s) |
| `--probe-timeout <ms>` | Probe timeout |
| `--probe-concurrency <n>` | Parallel probes |

---

### `openclaw models list`

**What it does:** List available models.

```bash
openclaw models list
openclaw models list --all       # include all known models
openclaw models list --local     # only local models
openclaw models list --provider anthropic
```

---

### `openclaw models set` / `set-image`

**What it does:** Set the default primary model (or image model).

```bash
openclaw models set claude-sonnet-4-5-20250929
openclaw models set-image dall-e-3
```

---

### `openclaw models scan`

**What it does:** Discovers what models are available from your configured providers. Can auto-set defaults.

**When would I use this?** After adding a new provider, or to see what's available. It probes provider APIs, so it may consume a small amount of tokens.

```bash
openclaw models scan
openclaw models scan --provider anthropic --set-default
openclaw models scan --no-probe --yes   # non-interactive, skip live probes
```

| Option | What it does |
|--------|-------------|
| `--min-params <b>` | Minimum parameter count (billions) |
| `--max-age-days <days>` | Max model age |
| `--provider <name>` | Only scan one provider |
| `--no-probe` | Skip live probes |
| `--yes` / `--no-input` | Non-interactive |
| `--set-default` | Auto-set the primary model |
| `--set-image` | Auto-set the image model |
| `--json` | Machine-readable output |

---

### `openclaw models auth`

**What it does:** Manage authentication with model providers.

```bash
# Interactive auth setup:
openclaw models auth add

# Set up Anthropic token (preferred method):
claude setup-token
openclaw models auth setup-token --provider anthropic

# Paste an existing token:
openclaw models auth paste-token --provider openai --expires-in 365d
```

| Subcommand | What it does |
|------------|-------------|
| `auth add` | Interactive auth helper |
| `auth setup-token` | Provider-specific token setup (`--provider <name>`, `--yes`) |
| `auth paste-token` | Paste an existing token (`--provider`, `--profile-id`, `--expires-in`) |

---

### `openclaw models aliases`

**What it does:** Manage model name aliases (e.g., map `"fast"` to `"claude-haiku-4-5-20251001"`).

```bash
openclaw models aliases list
openclaw models aliases add fast claude-haiku-4-5-20251001
openclaw models aliases remove fast
```

---

### `openclaw models fallbacks` / `image-fallbacks`

**What it does:** Configure fallback model chains. If your primary model is down, OpenClaw tries the next one in the list.

```bash
openclaw models fallbacks list
openclaw models fallbacks add gpt-4o
openclaw models fallbacks remove gpt-4o
openclaw models fallbacks clear

# Same for image models:
openclaw models image-fallbacks list
openclaw models image-fallbacks add dall-e-3
```

---

### `openclaw models auth order`

**What it does:** Control the priority order of auth profiles for a provider.

```bash
openclaw models auth order get --provider anthropic
openclaw models auth order set --provider anthropic profile1 profile2
openclaw models auth order clear --provider anthropic
```

Options: `--agent <id>` to scope to a specific agent, `--json` for output.

---

## 8. Security

### `openclaw security audit`

**What it does:** Scans your config and local state for security issues. Think of it as a security health check — it looks at 50+ checks across 12 categories (file permissions, channel policies, model hygiene, plugin trust, network exposure, and more).

**When would I use this?** After initial setup, after config changes, or periodically to make sure nothing has drifted. If you only do one security thing, run `openclaw security audit --fix`.

```bash
openclaw security audit          # read-only scan
openclaw security audit --deep   # + live WebSocket probe of running gateway
openclaw security audit --fix    # apply safe fixes, then show remaining issues
```

| Flag | What it adds | Modifies system? |
|------|-------------|-----------------|
| *(none)* | Config, filesystem, channel policies, model hygiene, plugin trust, attack surface | No (read-only) |
| `--deep` | All base checks + live WebSocket probe | No (read-only probe) |
| `--fix` | `chmod 600/700` on state/config, flip open policies to allowlist, enable redaction | Yes (safe defaults only) |

`--fix` runs fixes *before* the audit, so the report shows the hardened state.

See the [full audit command reference](../08-security-analysis/security-audit-command-reference.md).

---

### `openclaw approvals`

**What it does:** Manage the exec approval policy — controls whether the agent needs explicit approval before running shell commands.

```bash
openclaw approvals get
openclaw approvals set --policy ask
openclaw approvals allowlist add "git status"
openclaw approvals allowlist remove "rm -rf"
```

| Subcommand | What it does |
|------------|-------------|
| `get` | Show current approval policy |
| `set` | Set the policy |
| `allowlist add <command>` | Add a command to the auto-approve list |
| `allowlist remove <command>` | Remove from auto-approve list |

---

## 9. Automation

### `openclaw cron`

**What it does:** Manage scheduled jobs that run on the Gateway. Jobs can trigger system events or send messages on a schedule.

**When would I use this?** Automating recurring tasks — daily summaries, periodic health checks, scheduled reminders.

```bash
openclaw cron list
openclaw cron status
openclaw cron add --name "daily-summary" --every 24h \
  --system-event "Generate daily summary"
openclaw cron runs --id <job-id> --limit 5   # view recent runs
openclaw cron run <job-id> --force            # trigger now
openclaw cron disable <job-id>
openclaw cron rm <job-id>
```

| Subcommand | What it does |
|------------|-------------|
| `status` | Show scheduler status |
| `list` | List all jobs (`--all`, `--json`) |
| `add` | Create a job (needs `--name` + schedule + payload) |
| `edit <id>` | Patch job fields |
| `rm <id>` | Delete a job (aliases: `remove`, `delete`) |
| `enable <id>` / `disable <id>` | Toggle a job |
| `runs --id <id>` | View recent run history (`--limit <n>`) |
| `run <id>` | Trigger a job now (`--force`) |

Schedule options for `cron add`: exactly one of `--at <time>`, `--every <interval>`, or `--cron <expression>`.
Payload: exactly one of `--system-event <text>` or `--message <text>`.

All cron commands accept: `--url`, `--token`, `--timeout`, `--expect-final`.

---

### `openclaw hooks`

**What it does:** Manage event hooks — scripts that run in response to Gateway events.

```bash
openclaw hooks list
openclaw hooks info <hook-name>
openclaw hooks check              # verify hooks are valid
openclaw hooks enable <name>
openclaw hooks disable <name>
openclaw hooks install <path>
openclaw hooks update <name>
```

---

### `openclaw webhooks gmail`

**What it does:** Set up and run a Gmail Pub/Sub webhook — so your assistant can react to incoming emails.

**When would I use this?** Email-triggered automation (e.g., "summarize new emails", "flag urgent messages").

```bash
openclaw webhooks gmail setup --account user@gmail.com
openclaw webhooks gmail run
```

Options include: `--project`, `--topic`, `--subscription`, `--label`, `--hook-url`, `--hook-token`, `--bind`, `--port`, `--tailscale`, and more.

Docs: https://docs.openclaw.ai/automation/gmail-pubsub

---

## 10. Browser automation

OpenClaw can control a dedicated browser (Chrome/Brave/Edge/Chromium) for web automation tasks.

### Browser lifecycle

```bash
openclaw browser status          # is the browser running?
openclaw browser start           # start the managed browser
openclaw browser stop            # stop it
openclaw browser reset-profile   # reset browser profile
```

### Tab management

```bash
openclaw browser tabs                  # list open tabs
openclaw browser open https://example.com  # open a URL
openclaw browser focus <targetId>      # switch to a tab
openclaw browser close <targetId>      # close a tab
```

### Browser profiles

```bash
openclaw browser profiles
openclaw browser create-profile --name "research" --color "#FF5A2D"
openclaw browser delete-profile --name "research"
```

### Inspection

```bash
openclaw browser screenshot              # capture the page
openclaw browser screenshot --full-page  # full page, not just viewport
openclaw browser snapshot                # text snapshot (a11y tree)
openclaw browser snapshot --format ai    # AI-friendly format
openclaw browser console                 # view console messages
```

### Automation actions

```bash
openclaw browser navigate https://example.com
openclaw browser click <ref>
openclaw browser type <ref> "Hello world" --submit
openclaw browser press Enter
openclaw browser hover <ref>
openclaw browser drag <startRef> <endRef>
openclaw browser select <ref> "option1" "option2"
openclaw browser upload ./file.pdf --ref <ref>
openclaw browser fill --fields '{"#name": "Alice", "#email": "alice@example.com"}'
openclaw browser dialog --accept
openclaw browser wait --text "Loading complete"
openclaw browser evaluate --fn "() => document.title"
openclaw browser pdf    # save page as PDF
openclaw browser resize 1920 1080
```

Common options across browser commands: `--url`, `--token`, `--timeout`, `--json`, `--browser-profile <name>`, `--target-id <id>`.

---

## 11. Nodes + devices

Nodes are companion devices (macOS/iOS/Android/headless) that connect to the Gateway and offer device-local capabilities — cameras, screens, location, remote execution.

### `openclaw nodes` (gateway-side)

These commands talk to the Gateway and target *paired* nodes.

```bash
openclaw nodes status              # connected nodes
openclaw nodes list                # all known nodes
openclaw nodes describe --node myMac
openclaw nodes pending             # unapproved pairing requests
openclaw nodes approve <requestId>
openclaw nodes reject <requestId>
openclaw nodes rename --node myMac --name "Office Mac"
```

Remote execution:

```bash
# Invoke a registered command on a node
openclaw nodes invoke --node myMac --command "screenshot" --params '{}'

# Run a shell command on a node
openclaw nodes run --node myMac -- ls -la ~/Desktop
```

Camera:

```bash
openclaw nodes camera list --node myPhone
openclaw nodes camera snap --node myPhone --facing front
openclaw nodes camera clip --node myPhone --duration 10s
```

Canvas + screen:

```bash
openclaw nodes canvas snapshot --node myMac
openclaw nodes canvas present --node myMac --target https://example.com
openclaw nodes canvas hide --node myMac
openclaw nodes canvas navigate https://example.com --node myMac
openclaw nodes canvas eval "document.title" --node myMac
openclaw nodes screen record --node myMac --duration 10s
```

Location:

```bash
openclaw nodes location get --node myPhone --accuracy precise
```

Common options: `--url`, `--token`, `--timeout`, `--json`.

---

### `openclaw node` (headless node host)

These commands manage the node host *on the device itself* — running it, installing it as a service, etc.

```bash
openclaw node run --host gateway.local --port 18789
openclaw node status
openclaw node install --host gateway.local --port 18789
openclaw node uninstall
openclaw node start
openclaw node stop
openclaw node restart
```

`node install` options: `--host`, `--port`, `--tls`, `--tls-fingerprint`, `--node-id`, `--display-name`, `--runtime <node|bun>`, `--force`.

---

### `openclaw devices`

**What it does:** Device pairing and token management.

**When would I use this?** Managing which devices are allowed to connect to the Gateway.

---

### `openclaw pairing`

**What it does:** Approve DM pairing requests across channels. When someone first messages the bot, they need to be "paired" (approved by you).

```bash
openclaw pairing list telegram
openclaw pairing list telegram --json
openclaw pairing approve telegram ABC123 --notify
```

Docs: https://docs.openclaw.ai/start/pairing

---

## 12. Memory + docs search

Memory is like a personal notebook the bot keeps about you and your project. OpenClaw reads your markdown files (`MEMORY.md` and everything in the `memory/` folder), breaks them into small overlapping chunks, converts each chunk into a "meaning vector" (an embedding — a list of numbers that captures what the text *means*), and stores everything in a searchable SQLite database. When you or the agent search, it finds the most relevant chunks by *meaning*, not just keyword matching. This is called **semantic search**, and it's why searching for "deploy instructions" can find a paragraph titled "How to push to production" even though the words don't overlap.

### `openclaw memory status`

**What it does:** Shows the health of the memory search index — which embedding provider is active, how many files and chunks are indexed, whether the index is out of date ("dirty"), and whether the vector store (sqlite-vec) is available.

**When would I use this?** After setting up memory for the first time, after changing your embedding provider, or when memory search results seem stale or missing. The `--deep` flag probes the embedding provider to confirm it can actually generate embeddings (useful when an API key might be expired).

```bash
openclaw memory status                # quick index overview
openclaw memory status --deep         # also probe embedding provider availability
openclaw memory status --index        # reindex if dirty, then show status (implies --deep)
openclaw memory status --index --force  # force full reindex even if not dirty
openclaw memory status --json         # machine-readable JSON output
openclaw memory status --agent mybot  # check a specific agent (not the default)
```

| Option | What it does |
|--------|-------------|
| `--agent <id>` | Target a specific agent instead of the default |
| `--json` | Print results as JSON (for scripts) |
| `--deep` | Probe embedding provider availability and vector store readiness |
| `--index` | Reindex if dirty before showing status (implies `--deep`) |
| `--force` | Force full reindex even if nothing changed (use with `--index`) |
| `--verbose` | Show detailed progress during indexing |

### `openclaw memory index`

**What it does:** Manually triggers a reindex of all memory files. OpenClaw scans your files, re-chunks them, generates fresh embeddings, and updates the SQLite database. A progress bar shows elapsed time and ETA.

**When would I use this?** After you've edited `MEMORY.md` or files in `memory/`, after changing your embedding provider, or if `openclaw memory status` shows the index is dirty. Normally OpenClaw reindexes automatically (on session start, on file changes if watching is enabled), but this command forces it immediately.

```bash
openclaw memory index                 # incremental reindex (only changed files)
openclaw memory index --force         # full reindex from scratch
openclaw memory index --verbose       # show provider info + detailed progress
openclaw memory index --agent mybot   # reindex a specific agent
```

| Option | What it does |
|--------|-------------|
| `--agent <id>` | Target a specific agent instead of the default |
| `--force` | Full reindex — re-embeds every chunk even if the file hasn't changed |
| `--verbose` | Show provider details and per-file progress with elapsed/ETA |

### `openclaw memory search`

**What it does:** Runs a semantic search across your memory files and prints scored results with file paths, line ranges, and text snippets. This is the same search the agent uses internally, but exposed as a CLI command so you can test it yourself.

**When would I use this?** To check what the agent would "remember" about a topic, to verify that important information is indexed, or to find which memory file contains a specific piece of knowledge.

```bash
openclaw memory search "how to deploy"
openclaw memory search "telegram setup" --max-results 10
openclaw memory search "API keys" --min-score 0.5
openclaw memory search "deployment" --json
openclaw memory search "project goals" --agent mybot
```

| Option | What it does |
|--------|-------------|
| `--agent <id>` | Target a specific agent instead of the default |
| `--max-results <n>` | Maximum number of results to return (default: 6) |
| `--min-score <n>` | Minimum similarity score between 0 and 1 (default: 0.35) |
| `--json` | Print results as JSON (for scripts) |

Each result shows a **score** (0.000 to 1.000 — higher is more relevant), the **file path with line range**, and a **text snippet**.

### How memory files are organized

Think of `MEMORY.md` as the table of contents and the `memory/` folder as the chapters:

| Location | Purpose |
|----------|---------|
| `MEMORY.md` | Your main memory file — long-term curated knowledge, preferences, key decisions |
| `memory/*.md` | Topic files (e.g., `memory/debugging.md`, `memory/patterns.md`) — linked from MEMORY.md |
| Session transcripts | (Experimental) Past conversation logs — opt in via `sources: ["memory", "sessions"]` |
| Extra paths | Additional files/directories via `extraPaths` config — useful for shared team docs |

OpenClaw scans all of these locations (based on your config) when building the search index. If a file doesn't exist yet, that's fine — just create it and run `openclaw memory index`.

> **Security note:** Memory files (`MEMORY.md`, `memory/*.md`) are loaded as **trusted context** — they're injected directly into the agent's system prompt without untrusted-content markers. Any process or user with write access to your workspace can plant persistent prompt injection that survives across sessions. Restrict write access to the workspace directory and periodically audit memory file contents. See [Threat model, Section 7: Persistent memory files](../04-privacy-safety/threat-model.md) for full details.

### How search works (plain English)

Memory search happens in three stages:

1. **Chunking** — Each file is split into overlapping pieces of roughly 400 tokens (~300 words). The chunks overlap by 80 tokens so that a sentence sitting on a boundary isn't lost. Think of it like cutting a book into pages that each repeat the last few lines of the previous page.

2. **Embedding** — Each chunk is converted into a "meaning vector" — a list of numbers (typically 256-1536 dimensions) that captures the semantic meaning of the text. Two chunks about the same topic will have similar vectors even if they use completely different words. This is done by an embedding provider (see below).

3. **Hybrid search** — When you search, OpenClaw runs *two* searches in parallel and blends the results:
   - **Vector similarity** (70% weight by default) — finds chunks whose meaning is closest to your query, using cosine similarity between embedding vectors
   - **Keyword matching / BM25** (30% weight by default) — traditional text search via SQLite FTS5, which boosts chunks that contain your exact words

   The final score for each chunk is `0.7 * vectorScore + 0.3 * keywordScore`. This hybrid approach catches both "meaning matches" and "exact word matches" that pure vector search might miss.

**sqlite-vec** is a native SQLite extension that accelerates vector search using hardware-optimized operations. When available, vector lookups are a fast indexed query. When it's *not* available, OpenClaw falls back to computing cosine similarity for *every* chunk in memory (an O(n) full scan) — which works but gets slow as your memory grows.

> **Resource cost cross-reference:** Local embeddings are CPU-heavy (see [resource-usage.md, CPU #3](../06-optimizations/resource-usage.md) — "Very High" impact). The cosine fallback without sqlite-vec is also costly per query ([CPU #5](../06-optimizations/resource-usage.md)). SQLite memory databases don't auto-VACUUM ([Disk section](../06-optimizations/resource-usage.md) — WAL files can bloat over time).

### Embedding providers

An embedding provider is the service that converts text into meaning vectors. OpenClaw supports four:

| Provider | Default model | Token limit | Where it runs | Best for |
|----------|--------------|-------------|---------------|----------|
| **OpenAI** | `text-embedding-3-small` | 8,192 | OpenAI API (remote) | Most users — fast, cheap, good quality |
| **Gemini** | `gemini-embedding-001` | 2,048 | Google API (remote) | If you already have a Google API key |
| **Voyage** | `voyage-4-large` | 32,000 | Voyage AI API (remote) | Long documents, code-heavy memory files |
| **Local** | `embeddinggemma-300M` (GGUF) | varies | Your machine (node-llama-cpp) | Offline/air-gapped setups, privacy-sensitive |

**Auto-selection** (the default): When `provider` is set to `"auto"`, OpenClaw tries providers in this order: local (if a model file exists) > OpenAI > Gemini > Voyage. The first one that works wins.

**Practical advice:** If your machine is low-powered (e.g., a small VPS or older laptop), use an API provider instead of local. Local embedding inference demands serious CPU — see [resource-usage.md, CPU #3](../06-optimizations/resource-usage.md) for details.

### How the agent uses memory

During conversations, the agent has two memory tools it can call automatically:

| Tool | What it does |
|------|-------------|
| `memory_search` | Semantically searches memory files — same as `openclaw memory search` but called by the agent mid-conversation. Returns scored snippets with file paths and line numbers. |
| `memory_get` | Reads a specific section of a memory file (by path and line range). Used after `memory_search` to pull in just the relevant lines without loading entire files into context. |

The agent's system prompt tells it: *"Before answering questions about prior work, decisions, dates, people, preferences, or todos — search memory first."* This means the agent will proactively search your memory files when you ask about something it might have noted before, without you having to tell it to.

### Key configuration

All memory settings live under `agents.defaults.memorySearch` in your config. The most important ones:

| Config key | What it controls | Default |
|-----------|-----------------|---------|
| `provider` | Embedding provider (`"openai"`, `"gemini"`, `"voyage"`, `"local"`, `"auto"`) | `"auto"` |
| `query.hybrid.vectorWeight` | How much to weight vector similarity (0-1) | `0.7` |
| `query.hybrid.textWeight` | How much to weight keyword/BM25 matching (0-1) | `0.3` |
| `chunking.tokens` | Chunk size in tokens | `400` |
| `chunking.overlap` | Overlap between chunks in tokens | `80` |
| `query.maxResults` | Default number of search results | `6` |
| `query.minScore` | Minimum score threshold (0-1) | `0.35` |
| `sync.onSessionStart` | Reindex when a new session starts | `true` |
| `sync.onSearch` | Reindex before searching if dirty | `true` |
| `sync.watch` | Watch memory files for changes and auto-reindex | `true` |
| `store.vector.enabled` | Use sqlite-vec for accelerated vector search | `true` |
| `sources` | Which sources to index (`["memory"]` or `["memory", "sessions"]`) | `["memory"]` |

Run `openclaw configure` for an interactive setup wizard that walks you through these options.

> **Resource impact:** Local embeddings are CPU-heavy ([resource-usage.md, CPU #3](../06-optimizations/resource-usage.md)). API providers offload that work to the cloud. The sqlite-vec extension avoids the O(n) cosine fallback ([CPU #5](../06-optimizations/resource-usage.md)). SQLite databases don't auto-VACUUM, so WAL files can bloat — run `sqlite3 ~/.openclaw/memory/*.sqlite VACUUM` periodically ([Disk section](../06-optimizations/resource-usage.md)).

### `openclaw docs`

**What it does:** Searches the **live OpenClaw documentation site** (docs.openclaw.ai) and prints matching pages with titles, links, and snippets. This is completely separate from `memory search` — it searches the *official docs*, not your local memory files.

**How is this different from `memory search`?** `memory search` looks in *your* notes (MEMORY.md, memory/*.md). `docs` looks in the *project's official documentation*. Use `docs` when you want to know how a feature works; use `memory search` when you want to recall something you or the agent previously noted.

```bash
openclaw docs "how do I set up Telegram?"
openclaw docs "security audit"
openclaw docs "memory search configuration"
```

Results show the page **title**, a clickable **link** to the docs site, and a short **snippet** of matching content.

---

## 13. Plugins + skills

### `openclaw plugins`

**What it does:** Manage extensions that add channels, tools, or features to the Gateway. Plugins run **in-process** — treat installing one like running arbitrary code.

```bash
openclaw plugins list                  # discover plugins
openclaw plugins list --json
openclaw plugins info <id>             # plugin details
openclaw plugins install <path|.tgz|npm-spec>   # install a plugin
openclaw plugins enable <id>           # turn on
openclaw plugins disable <id>          # turn off
openclaw plugins doctor                # report load errors
```

Most plugin changes require a Gateway restart.

---

### `openclaw skills`

**What it does:** List and inspect available skills (pre-built capabilities like web search, image generation, etc.) and whether their requirements are met.

```bash
openclaw skills list                  # all skills
openclaw skills list --eligible       # only ready-to-use skills
openclaw skills info <name>           # details for one skill
openclaw skills check                 # readiness summary
```

| Option | What it does |
|--------|-------------|
| `--eligible` | Only show ready skills |
| `--json` | Machine-readable output |
| `-v` / `--verbose` | Include missing requirements detail |

Tip: use `npx clawhub` to search, install, and sync skills from the marketplace.

---

## 14. System + maintenance

### `openclaw system event`

**What it does:** Enqueue a system event and optionally trigger a heartbeat (via Gateway RPC).

```bash
openclaw system event --text "Deployment complete" --mode now
```

| Option | What it does |
|--------|-------------|
| `--text <text>` | **Required.** Event text |
| `--mode <now\|next-heartbeat>` | When to fire |
| `--json` | Machine-readable output |

---

### `openclaw system heartbeat`

**What it does:** Control the heartbeat timer (the periodic "still alive" signal).

```bash
openclaw system heartbeat last      # when was the last heartbeat?
openclaw system heartbeat enable
openclaw system heartbeat disable
```

---

### `openclaw system presence`

**What it does:** List system presence entries (which components are online).

```bash
openclaw system presence
openclaw system presence --json
```

All `system` commands accept: `--url`, `--token`, `--timeout`, `--expect-final`, `--json`.

---

### `openclaw sandbox`

**What it does:** Manage agent sandboxes (isolated execution environments).

```bash
openclaw sandbox list
openclaw sandbox recreate       # rebuild sandboxes
openclaw sandbox explain        # explain sandbox config
```

---

### `openclaw reset`

**What it does:** Reset local config and state. The CLI stays installed; this just wipes your settings.

**When would I use this?** Starting fresh without fully uninstalling.

```bash
openclaw reset --scope config                     # config only
openclaw reset --scope config+creds+sessions      # config + credentials + sessions
openclaw reset --scope full                        # everything
openclaw reset --scope full --dry-run              # preview what would be deleted
```

| Option | What it does |
|--------|-------------|
| `--scope <level>` | `config\|config+creds+sessions\|full` |
| `--yes` | Skip confirmation |
| `--non-interactive` | Requires `--scope` and `--yes` |
| `--dry-run` | Show what would be deleted |

---

### `openclaw uninstall`

**What it does:** Remove the gateway service and/or local data. The CLI binary itself remains.

```bash
openclaw uninstall --all --yes         # remove everything
openclaw uninstall --service           # just the service
openclaw uninstall --state             # just state data
openclaw uninstall --all --dry-run     # preview
```

| Option | What it does |
|--------|-------------|
| `--service` | Remove the gateway service |
| `--state` | Remove state directory |
| `--workspace` | Remove workspace |
| `--app` | Remove the app |
| `--all` | Remove everything |
| `--yes` | Skip confirmation |
| `--non-interactive` | Requires `--yes` and scopes (or `--all`) |
| `--dry-run` | Show what would be removed |

---

### `openclaw update`

**What it does:** Update the CLI to the latest version (source installs only).

```bash
openclaw update
```

You can also use `openclaw --update` as a shorthand.

---

## 15. Terminal UI + shell completion

### `openclaw tui`

**What it does:** Open an interactive terminal UI connected to the Gateway. A text-based chat interface in your terminal.

**When would I use this?** When you want a chat experience without opening a browser or messaging app.

```bash
openclaw tui
openclaw tui --message "Hello"  # start with a message
```

| Option | What it does |
|--------|-------------|
| `--url <url>` | Gateway URL |
| `--token <token>` | Auth token |
| `--password <password>` | Auth password |
| `--session <key>` | Session key |
| `--deliver` | Deliver responses via channel |
| `--thinking <level>` | Thinking level |
| `--message <text>` | Initial message |
| `--timeout-ms <ms>` | Turn timeout |
| `--history-limit <n>` | History length |

---

### `openclaw completion`

**What it does:** Generate a shell completion script so you get tab-completion for `openclaw` commands.

```bash
openclaw completion          # prints the script
openclaw completion >> ~/.bashrc   # add to your shell
```

---

### `openclaw dns setup`

**What it does:** Wide-area discovery DNS helper (CoreDNS + Tailscale).

**When would I use this?** Advanced setups where you want DNS-based gateway discovery.

```bash
openclaw dns setup
openclaw dns setup --apply   # install/update CoreDNS config (macOS, needs sudo)
```

---

## Global flags

These work with any `openclaw` command:

| Flag | What it does |
|------|-------------|
| `--dev` | Isolate state under `~/.openclaw-dev` and shift default ports |
| `--profile <name>` | Isolate state under `~/.openclaw-<name>` |
| `--no-color` | Disable ANSI colors |
| `--update` | Shorthand for `openclaw update` (source installs only) |
| `-V` / `--version` / `-v` | Print version and exit |

### Output styling

- ANSI colors and progress indicators only render in TTY sessions (i.e., when you're in a real terminal, not piping to a file).
- `--json` (and `--plain` where supported) disables styling for clean, parseable output.
- `--no-color` disables ANSI styling; `NO_COLOR=1` environment variable is also respected.
- Long-running commands show a progress indicator (uses OSC 9;4 in supported terminals).

---

## Environment variables

These override config-file settings. Useful for scripting, CI/CD, or running multiple instances.

| Variable | What it controls | Default |
|----------|-----------------|---------|
| `OPENCLAW_STATE_DIR` | State directory (sessions, logs, caches) | `~/.openclaw` |
| `OPENCLAW_CONFIG_PATH` | Config file location | `$OPENCLAW_STATE_DIR/openclaw.json` |
| `OPENCLAW_GATEWAY_PORT` | Gateway port | `18789` |
| `OPENCLAW_GATEWAY_TOKEN` | Gateway auth token | *(from config)* |
| `OPENCLAW_PROFILE` | Profile name (isolates state under `~/.openclaw-<name>`) | *(none)* |
| `OPENCLAW_AGENT_DIR` | Agent directory | `$OPENCLAW_STATE_DIR/agent` |
| `OPENCLAW_OAUTH_DIR` | OAuth credentials directory | `$OPENCLAW_STATE_DIR/credentials` |
| `OPENCLAW_SKIP_CHANNELS` | Skip channel initialization (`1` = skip) | *(off)* |
| `OPENCLAW_HOME` | Override home directory for state resolution | `$HOME` |
| `OPENCLAW_NIX_MODE` | Nix mode (`1` = no auto-install flows, Nix-specific errors) | *(off)* |
| `OPENCLAW_DISABLE_LAZY_SUBCOMMANDS` | Eagerly register all subcommands (`1` = eager) | *(off)* |
| `CUSTOM_API_KEY` | API key for custom providers (used with `--auth-choice custom-api-key`) | *(none)* |
| `NO_COLOR` | Disable ANSI colors (`1` = no color) | *(off)* |

Legacy aliases (`CLAWDBOT_STATE_DIR`, `CLAWDBOT_CONFIG_PATH`, `CLAWDBOT_GATEWAY_PORT`) still work for backward compatibility.

---

## Where to go next

- **Quick copy/paste reference:** [Commands + troubleshooting](../99-reference/commands-and-troubleshooting.md)
- **Canonical technical reference:** [CLI reference](../../docs/cli/index.md)
- **What is OpenClaw?** [Plain English introduction](./what-is-clawdbot.md)
- **Glossary:** [Terms you'll see in the CLI](./glossary.md)
- **Security audit details:** [Security audit command reference](../08-security-analysis/security-audit-command-reference.md)
- **Official docs:** https://docs.openclaw.ai

---

Previous: [Glossary](./glossary.md) | Up: [Home](../README.md)
