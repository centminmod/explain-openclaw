# Installing Clawdbot

This guide walks you through installing Clawdbot on your system.

## Before you begin

### Requirements

- **Node.js 22 or higher** — [Download from nodejs.org](https://nodejs.org)
- **macOS or Linux** — Windows works via WSL2
- **An AI provider account** — Anthropic (Claude) or OpenAI (GPT-4) recommended
- **Basic terminal comfort** — You'll need to run some commands

### Check if Node.js is installed

```bash
node --version
# Should show v22.x.x or higher

npm --version
# Should show 10.x.x or higher
```

If not installed, download from [nodejs.org](https://nodejs.org) or use a version manager like `nvm`.

## Installation methods

### Method 1: Quick install (recommended)

The fastest way to get started:

```bash
# Install globally via npm
npm install -g clawdbot@latest

# Or using pnpm
pnpm add -g clawdbot@latest

# Verify installation
clawdbot --version
```

### Method 2: Using the install script

For macOS and Linux:

```bash
curl -fsSL https://clawd.bot/install.sh | sh
```

This will:
- Install the latest version
- Set up the configuration directory
- Run the onboarding wizard

### Method 3: From source (for developers)

If you want to modify or contribute to Clawdbot:

```bash
# Clone the repository
git clone https://github.com/clawdbot/clawdbot.git
cd clawdbot

# Install dependencies
pnpm install

# Build the project
pnpm build

# Run from source
pnpm clawdbot --version
```

## Post-installation steps

### 1. Run the onboarding wizard

The wizard will guide you through initial setup:

```bash
clawdbot onboard --install-daemon
```

The wizard asks about:

1. **AI provider** — Anthropic (Claude) or OpenAI (GPT-4)
2. **API credentials** — Your API key
3. **Channels to enable** — WhatsApp, Telegram, etc.
4. **Gateway mode** — Local or remote

### 2. Set up your AI provider

#### Option A: Anthropic (Claude) — Recommended

1. Go to [console.anthropic.com](https://console.anthropic.com)
2. Create an account or sign in
3. Get your API key from the dashboard
4. Run: `clawdbot config set agent.model anthropic/claude-opus-4-5`
5. Set your key: `export ANTHROPIC_API_KEY=sk-ant-...`

#### Option B: OpenAI (GPT-4)

1. Go to [platform.openai.com](https://platform.openai.com)
2. Create an account or sign in
3. Get your API key from the dashboard
4. Run: `clawdbot config set agent.model openai/gpt-4o`
5. Set your key: `export OPENAI_API_KEY=sk-...`

### 3. Start the Gateway

```bash
# Start in foreground (to see logs)
clawdbot gateway --port 18789

# Or start as a background service
clawdbot service start
```

### 4. Verify it's working

```bash
# Check gateway status
clawdbot status

# Send a test message
clawdbot agent --message "Hello from Clawdbot!"
```

## Platform-specific notes

### macOS

1. Install Command Line Tools if prompted:
   ```bash
   xcode-select --install
   ```

2. Grant permissions when prompted:
   - Accessibility (for keyboard shortcuts)
   - Full Disk Access (for file access)

3. Optional: Download the macOS app for a menu bar interface

### Linux

1. Ensure you have build tools:
   ```bash
   sudo apt-get install build-essential  # Ubuntu/Debian
   # or
   sudo dnf install gcc-c++ make          # Fedora
   ```

2. Set up systemd service (optional):
   ```bash
   clawdbot service install
   clawdbot service start
   ```

### Windows (WSL2)

1. Install WSL2 following [Microsoft's guide](https://learn.microsoft.com/en-us/windows/wsl/install)

2. Install Ubuntu or another Linux distro

3. Follow the Linux instructions inside WSL2

## Verification checklist

After installation, verify:

- [ ] `clawdbot --version` works
- [ ] `clawdbot status` shows the Gateway is running
- [ ] Your API key is configured
- [ ] At least one channel is set up (or CLI works)
- [ ] You can send a test message

## Troubleshooting

### "command not found: clawdbot"

- Make sure you installed globally: `npm install -g clawdbot@latest`
- Check npm global bin directory: `npm config get prefix`
- Add it to your PATH if needed

### "Cannot connect to Gateway"

- Make sure the Gateway is running: `clawdbot status`
- Check the port: `clawdbot config get gateway.port`
- Try specifying port explicitly: `clawdbot gateway --port 18789`

### API errors

- Verify your API key is set correctly
- Check your API key has credits/billing set up
- Test with a simple message first

### Permission errors (macOS)

- Run `clawdbot doctor` to diagnose
- Grant requested permissions in System Settings
- Restart the Gateway after granting permissions

## Upgrading

To upgrade to the latest version:

```bash
# Via npm
npm update -g clawdbot

# Via pnpm
pnpm update -g clawdbot

# From source
git pull
pnpm install
pnpm build
```

## Uninstalling

```bash
# Stop the service
clawdbot service stop

# Uninstall
npm uninstall -g clawdbot

# Remove configuration (optional, deletes all data)
rm -rf ~/.clawdbot
```

## Next steps

After installing:

- [Configure it](./configuration.md) — Customize settings
- [Set up channels](./reference.md#channels) — Connect WhatsApp, Telegram, etc.
- Choose your setup: [Mac Mini](./usage-mac-mini.md) or [VPS](./usage-vps.md)

For detailed installation guides for specific platforms, see the [official docs](https://docs.clawd.bot/install).
