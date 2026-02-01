# Usage Examples

> **Last Updated:** January 2026  
> **Applies to:** Moltbot 2026.1.27-beta.1+

This guide provides practical examples for common Moltbot use cases.

---

## Table of Contents

1. [Getting Started Commands](#getting-started-commands)
2. [Channel Management](#channel-management)
3. [AI Agent Interaction](#ai-agent-interaction)
4. [File & System Operations](#file--system-operations)
5. [Automation & Cron](#automation--cron)
6. [Multi-Agent Workflows](#multi-agent-workflows)
7. [Troubleshooting Commands](#troubleshooting-commands)

---

## Getting Started Commands

### Check System Status

```bash
# Overall health check
moltbot health

# Detailed status with all components
moltbot status --all

# Deep status with live probes
moltbot status --deep
```

### Configuration Management

```bash
# View current configuration
moltbot config get

# View specific section
moltbot config get gateway
moltbot config get channels.whatsapp

# Set a configuration value
moltbot config set agent.model "anthropic/claude-opus-4-5"
moltbot config set channels.whatsapp.dmPolicy "pairing"

# Reset configuration to default
moltbot config reset

# Validate configuration
moltbot doctor --check-config
```

### Gateway Control

```bash
# Start Gateway (foreground)
moltbot gateway --port 18789 --verbose

# Start Gateway (background with logs)
nohup moltbot gateway --port 18789 > /tmp/moltbot.log 2>&1 &

# Check Gateway status
moltbot gateway status

# Stop Gateway
moltbot gateway stop

# Restart Gateway
moltbot gateway restart
```

---

## Channel Management

### WhatsApp

```bash
# Login (scan QR code)
moltbot channels login whatsapp

# Check connection status
moltbot channels status whatsapp

# Logout
moltbot channels logout whatsapp

# Send a message
moltbot message send --to "+1234567890" --message "Hello from Moltbot!"

# Send with media
moltbot message send --to "+1234567890" --message "Check this out" --media ./image.jpg
```

### Telegram

```bash
# Configure bot token
moltbot config set channels.telegram.botToken "123456:ABC-DEF1234"

# Check status
moltbot channels status telegram

# Set webhook (for production)
moltbot config set channels.telegram.webhookUrl "https://your-domain.com/webhook"
```

### Discord

```bash
# Configure bot token
moltbot config set channels.discord.token "your-bot-token"

# Configure allowed guilds
moltbot config set channels.discord.guilds '{"guild-id": {"requireMention": true}}'

# Check bot status
moltbot channels status discord
```

### Slack

```bash
# Configure tokens
moltbot config set channels.slack.botToken "xoxb-your-bot-token"
moltbot config set channels.slack.appToken "xapp-your-app-level-token"

# Configure channels
moltbot config set channels.slack.channels '{"general": {"requireMention": false}}'
```

---

## AI Agent Interaction

### Basic Agent Invocation

```bash
# Send a message to the agent
moltbot agent --message "What is the weather today?"

# With specific thinking level
moltbot agent --message "Analyze this code" --thinking high

# With specific model
moltbot agent --message "Explain quantum computing" --model "anthropic/claude-opus-4-5"

# Read from file
moltbot agent --file ./code.py --message "Review this code"

# With context from URL
moltbot agent --message "Summarize this article" --context-url "https://example.com/article"
```

### Using Tools

```bash
# Execute bash command (if enabled)
moltbot agent --message "Run: ls -la"

# Read file
moltbot agent --message "Read the contents of README.md"

# Search web
moltbot agent --message "Search for latest Node.js version"

# Browser control
moltbot agent --message "Open example.com and find the contact page"

# Multi-step task
moltbot agent --message "Search for Python async tutorials, then create a summary"
```

### Session Management

```bash
# List active sessions
moltbot sessions list

# Show session details
moltbot sessions show main

# Reset session (clear context)
moltbot sessions reset main

# Compact session (summarize old messages)
moltbot sessions compact main

# Delete session
moltbot sessions delete session-id
```

### In-Chat Commands

When messaging the bot directly, use these commands:

```
/status          - Show current model, tokens, cost
/new or /reset   - Reset the session
/compact         - Compact session context
/think <level>   - Set thinking level (off|minimal|low|medium|high|xhigh)
/verbose on|off  - Toggle verbose output
/usage <mode>    - Show usage footer (off|tokens|full)
/activation      - Toggle group activation mode
```

---

## File & System Operations

### File Management

```bash
# Read file (via agent)
moltbot agent --message "Read /path/to/file.txt"

# Write file
moltbot agent --message "Create a file at ~/notes.txt with content: Hello World"

# Edit file
moltbot agent --message "Edit ~/config.json to add debug: true"

# Apply patch
moltbot agent --file ./changes.patch --message "Apply this patch"
```

### System Commands

```bash
# Run system command (requires elevated permissions)
moltbot agent --message "Run system command: df -h"

# Check system info
moltbot agent --message "What's the current CPU and memory usage?"

# Notification
moltbot agent --message "Send notification: Build completed"
```

### Browser Control

```bash
# Open browser
moltbot agent --message "Open browser to https://example.com"

# Take screenshot
moltbot agent --message "Take a screenshot of the current page"

# Navigate and interact
moltbot agent --message "Go to example.com, click on 'About', extract the team list"

# Form filling
moltbot agent --message "Fill out the contact form with name 'John', email 'john@example.com'"
```

---

## Automation & Cron

### Scheduled Tasks

```bash
# List all cron jobs
moltbot cron list

# Create a daily summary job
moltbot cron create --name "daily-summary" \
  --schedule "0 9 * * *" \
  --message "Send me a summary of today's calendar and tasks"

# Create a health check job
moltbot cron create --name "health-check" \
  --schedule "*/30 * * * *" \
  --message "Check disk space and notify if below 10%"

# Enable/disable a job
moltbot cron disable daily-summary
moltbot cron enable daily-summary

# Delete a job
moltbot cron delete daily-summary

# View job logs
moltbot cron logs daily-summary --tail 50
```

### Webhooks

```bash
# Create a webhook endpoint
moltbot webhook create --name "github-events" \
  --url "/webhooks/github" \
  --secret "webhook-secret"

# List webhooks
moltbot webhook list

# Test webhook
moltbot webhook test github-events \
  --payload '{"event": "push", "ref": "main"}'

# Delete webhook
moltbot webhook delete github-events
```

### Gmail Integration

```bash
# Setup Gmail Pub/Sub
moltbot config set automation.gmail.enabled true
moltbot config set automation.gmail.projectId "your-gcp-project"

# Create filter for specific emails
moltbot automation gmail create-filter \
  --from "alerts@company.com" \
  --subject "URGENT" \
  --action "notify"
```

---

## Multi-Agent Workflows

### Session-to-Session Communication

```bash
# List all sessions
moltbot sessions list

# Send message to another session
moltbot sessions send --to "research-agent" --message "Find info on X"

# Get history from another session
moltbot sessions history --session "research-agent" --limit 10
```

### Agent Spawning

```bash
# Spawn a new agent for specific task
moltbot agent spawn --name "code-reviewer" \
  --workspace "~/projects/reviews" \
  --tools "read,apply_patch" \
  --message "Review the PR at ~/projects/app/pull-request-123"

# List spawned agents
moltbot agents list

# Terminate specific agent
moltbot agents terminate code-reviewer
```

### Workspace Management

```bash
# List workspaces
moltbot workspace list

# Create new workspace
moltbot workspace create --name "personal" --path "~/clawd-personal"

# Switch workspace
moltbot config set agents.defaults.workspace "~/clawd-personal"

# Sync workspace skills
moltbot skills sync --workspace "~/clawd-personal"
```

---

## Node Management

### Mobile Nodes (iOS/Android)

```bash
# List paired nodes
moltbot nodes list

# Pair a new node
moltbot nodes pair

# Check node status
moltbot nodes status <node-id>

# Invoke node command
moltbot nodes invoke <node-id> --command "camera.snap"

# Get location from mobile node
moltbot nodes invoke <node-id> --command "location.get"

# Start screen recording
moltbot nodes invoke <node-id> --command "screen.record" --duration 30
```

### macOS Node

```bash
# Use macOS-specific capabilities
moltbot nodes invoke macbook-pro --command "system.notify" \
  --args '{"title": "Alert", "body": "Task completed"}'

# Run local script
moltbot nodes invoke macbook-pro --command "system.run" \
  --args '{"cmd": "osascript -e \'display notification \"Done\"\'"}'
```

---

## Troubleshooting Commands

### Diagnostic Commands

```bash
# Run comprehensive diagnostics
moltbot doctor

# Fix common issues
moltbot doctor --fix

# Check Gateway health
moltbot health

# View logs
moltbot logs --follow
moltbot logs --since "1 hour ago"

# Check for updates
moltbot update --check

# Update to latest version
moltbot update --channel stable
```

### Security Audit

```bash
# Basic security check
moltbot security audit

# Deep audit with live probe
moltbot security audit --deep

# Auto-fix safe issues
moltbot security audit --fix

# Generate report
moltbot security audit --format json --output audit-report.json
```

### Debug Information

```bash
# Show full system information
moltbot status --all

# Export debug bundle
moltbot doctor --export-bundle ./debug-bundle.zip

# Check specific component
moltbot doctor --check-channels
moltbot doctor --check-auth
moltbot doctor --check-permissions
```

---

## Advanced Examples

### Complex Multi-Step Workflow

```bash
#!/bin/bash
# deploy-check.sh - Automated deployment verification

# 1. Check pre-deployment conditions
echo "Checking pre-deployment conditions..."
moltbot agent --message "Check GitHub for any open blockers for repo myapp"

# 2. Run tests
moltbot agent --message "Run the test suite in ~/projects/myapp and report results"

# 3. Deploy if tests pass
moltbot agent --message "If tests passed, deploy to staging with: cd ~/projects/myapp && make deploy-staging"

# 4. Verify deployment
moltbot agent --message "Check if staging is healthy at https://staging.myapp.com/health"

# 5. Notify team
moltbot message send --to "#deployments" --message "Deployment complete. Check #deployments channel for details."
```

### Research Assistant Workflow

```bash
# 1. Create dedicated research session
RESEARCH_SESSION=$(moltbot agent spawn --name "research-$(date +%s)" --workspace "~/research")

# 2. Conduct research
moltbot sessions send --to "$RESEARCH_SESSION" --message ""

# 3. Compile findings
moltbot sessions history --session "$RESEARCH_SESSION" > research-notes.md

# 4. Clean up
moltbot agents terminate "$RESEARCH_SESSION"
```

### Daily Standup Automation

```bash
# daily-standup.sh

# Collect yesterday's activities
moltbot agent --message "Summarize my calendar events from yesterday"

# Get today's agenda
moltbot agent --message "What's on my calendar for today?"

# Check pending tasks
moltbot agent --message "List any overdue or high-priority tasks"

# Generate standup message
STANDUP=$(moltbot agent --message "Generate a standup update from today's info")

# Send to team channel
moltbot message send --to "#standup" --message "$STANDUP"
```

---

## Tips & Best Practices

### Performance

```bash
# Use appropriate thinking level
moltbot agent --message "Quick question" --thinking minimal
moltbot agent --message "Complex analysis" --thinking high

# Compact sessions regularly to reduce token usage
moltbot sessions compact main

# Prune old sessions
moltbot sessions prune --older-than 30d
```

### Security

```bash
# Always approve unknown DMs
moltbot pairing list whatsapp
moltbot pairing approve whatsapp CODE123

# Use sandbox for untrusted agents
moltbot config set agents.defaults.sandbox.mode "non-main"

# Regular security audits
moltbot security audit --deep --fix
```

### Organization

```bash
# Use descriptive session names
moltbot agent --session "project-alpha-discussion" --message "..."

# Tag important messages
moltbot agent --message "#important Deploy at 3pm today"

# Archive completed work
moltbot sessions archive old-session-id
```

---

## Related Documentation

- [Installation & Setup](./installation-and-setup.md)
- [Configuration Reference](./configuration-reference.md)
- [Security Analysis](./security-analysis.md)
- [Official CLI Docs](https://docs.molt.bot/cli)