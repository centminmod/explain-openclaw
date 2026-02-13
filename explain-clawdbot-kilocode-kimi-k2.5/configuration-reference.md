# Configuration Reference

> **Last Updated:** January 2026  
> **Applies to:** Moltbot 2026.1.27-beta.1+  
> **Config Location:** `~/.moltbot/moltbot.json`

Complete reference for all Moltbot configuration options.

---

## Table of Contents

1. [Configuration File Structure](#configuration-file-structure)
2. [Gateway Settings](#gateway-settings)
3. [Channel Configuration](#channel-configuration)
4. [Agent Configuration](#agent-configuration)
5. [Security Settings](#security-settings)
6. [Provider Configuration](#provider-configuration)
7. [Tool Configuration](#tool-configuration)
8. [Logging Configuration](#logging-configuration)
9. [Example Configurations](#example-configurations)

---

## Configuration File Structure

### Location

| Platform | Path |
|----------|------|
| macOS/Linux | `~/.moltbot/moltbot.json` |
| Windows | `%USERPROFILE%\.moltbot\moltbot.json` |
| Custom | `$CLAWDBOT_STATE_DIR/moltbot.json` |

### Format

Moltbot uses **JSON5** format (JSON with comments and relaxed syntax):

```json5
{
  // This is a comment
  gateway: {
    port: 18789,  // Trailing commas allowed
  },
}
```

### Schema Validation

Configuration is validated on startup. Use `moltbot doctor --check-config` to validate.

---

## Gateway Settings

### Core Gateway Configuration

```json5
{
  gateway: {
    // Operating mode
    // Options: "local", "remote"
    mode: "local",
    
    // Network binding
    // Options: "loopback" (127.0.0.1), "lan" (0.0.0.0), "tailnet", "custom"
    bind: "loopback",
    
    // Custom bind address (when bind: "custom")
    bindAddress: "192.168.1.100",
    
    // Port to listen on
    port: 18789,
    
    // HTTPS/TLS configuration
    tls: {
      enabled: false,
      cert: "/path/to/cert.pem",
      key: "/path/to/key.pem"
    },
    
    // Control UI settings
    controlUi: {
      enabled: true,
      // Allow insecure auth (localhost only recommended)
      allowInsecureAuth: false,
      // DANGER: Disables device auth entirely
      dangerouslyDisableDeviceAuth: false
    }
  }
}
```

### Authentication

```json5
{
  gateway: {
    auth: {
      // Authentication mode
      // Options: "token", "password", "none" (NOT RECOMMENDED)
      mode: "token",
      
      // Token auth
      token: "your-secure-random-token",
      
      // Password auth (alternative to token)
      // password: "${CLAWDBOT_GATEWAY_PASSWORD}",
      
      // Allow Tailscale identity headers
      allowTailscale: true,
      
      // CORS origins
      allowedOrigins: ["https://app.molt.bot"]
    }
  }
}
```

### Remote Access

```json5
{
  gateway: {
    remote: {
      // Remote Gateway connection (for CLI on different machine)
      host: "gateway.example.com",
      port: 18789,
      
      // Auth for remote connection
      token: "remote-gateway-token",
      // password: "remote-password",
      
      // TLS fingerprint pinning (security)
      tlsFingerprint: "sha256/abc123...",
      
      // SSH tunnel settings
      ssh: {
        enabled: false,
        host: "gateway.example.com",
        user: "moltbot",
        keyFile: "~/.ssh/moltbot_gateway"
      }
    }
  }
}
```

### Tailscale Integration

```json5
{
  gateway: {
    tailscale: {
      // Tailscale mode
      // Options: "off", "serve" (tailnet only), "funnel" (public)
      mode: "serve",
      
      // Reset on exit
      resetOnExit: true,
      
      // Funnel configuration (when mode: "funnel")
      funnel: {
        domain: "moltbot.your-domain.ts.net"
      }
    }
  }
}
```

### Discovery & mDNS

```json5
{
  discovery: {
    mdns: {
      // mDNS broadcast mode
      // Options: "off", "minimal" (recommended), "full"
      mode: "minimal",
      
      // Custom service name
      serviceName: "moltbot-gateway"
    },
    
    // Trusted proxy addresses (for X-Forwarded-For)
    trustedProxies: ["127.0.0.1", "10.0.0.1"]
  }
}
```

---

## Channel Configuration

### WhatsApp

```json5
{
  channels: {
    whatsapp: {
      enabled: true,
      
      // DM Policy
      // Options: "pairing" (recommended), "allowlist", "open", "disabled"
      dmPolicy: "pairing",
      
      // DM allowlist (phone numbers)
      allowFrom: ["+1234567890", "+0987654321"],
      
      // Group configuration
      groups: {
        // Per-group settings
        "group-id-1": {
          requireMention: true,
          mentionPatterns: ["@clawd", "@bot"]
        },
        // Default for all groups
        "*": {
          requireMention: true
        }
      },
      
      // Media size limits
      mediaMaxMb: 50,
      
      // Auto-reply settings
      autoReply: {
        enabled: true,
        delayMs: 1000
      }
    }
  }
}
```

### Telegram

```json5
{
  channels: {
    telegram: {
      enabled: true,
      
      // Bot token
      botToken: "${TELEGRAM_BOT_TOKEN}",
      // Or use tokenFile: "/path/to/token"
      
      // DM Policy
      dmPolicy: "pairing",
      allowFrom: [],
      
      // Group configuration
      groups: {
        "@mychannel": {
          requireMention: false
        }
      },
      
      // Webhook (recommended for production)
      webhookUrl: "https://your-domain.com/webhook/telegram",
      
      // Polling (fallback if no webhook)
      polling: {
        enabled: false,
        interval: 1000
      }
    }
  }
}
```

### Discord

```json5
{
  channels: {
    discord: {
      enabled: true,
      
      // Bot token
      token: "${DISCORD_BOT_TOKEN}",
      
      // DM Policy
      dmPolicy: "pairing",
      allowFrom: [],
      
      // Guild (server) configuration
      guilds: {
        "guild-id-1": {
          // Channel-specific settings
          channels: {
            "general": { requireMention: false },
            "bot-commands": { requireMention: false }
          }
        }
      },
      
      // Slash commands
      commands: {
        native: true,      // Register Discord slash commands
        text: true,        // Allow text commands
        useAccessGroups: true
      },
      
      // Media limits
      mediaMaxMb: 25
    }
  }
}
```

### Slack

```json5
{
  channels: {
    slack: {
      enabled: true,
      
      // Tokens
      botToken: "${SLACK_BOT_TOKEN}",
      appToken: "${SLACK_APP_TOKEN}",
      
      // Socket mode (for workspaces without public URL)
      socketMode: false,
      
      // Channel configuration
      channels: {
        "#general": { requireMention: false },
        "#random": { requireMention: true }
      },
      
      // DM settings
      dm: {
        policy: "pairing",
        allowFrom: []
      }
    }
  }
}
```

### iMessage (macOS only)

```json5
{
  channels: {
    imessage: {
      enabled: true,
      
      // Group configuration
      groups: {
        "*": {
          requireMention: true
        }
      }
    }
  }
}
```

### Signal

```json5
{
  channels: {
    signal: {
      enabled: true,
      
      // signal-cli configuration
      cliPath: "/usr/local/bin/signal-cli",
      
      // Account settings
      account: "+1234567890",
      
      // DM Policy
      dmPolicy: "pairing"
    }
  }
}
```

---

## Agent Configuration

### Default Agent Settings

```json5
{
  agents: {
    defaults: {
      // Default AI model
      model: "anthropic/claude-opus-4-5",
      
      // Workspace directory
      workspace: "~/clawd",
      
      // Thinking level
      // Options: "off", "minimal", "low", "medium", "high", "xhigh"
      thinking: "low",
      
      // Session settings
      session: {
        // DM session isolation
        // Options: "main" (all DMs to main), 
        //          "per-channel-peer" (isolate by contact),
        //          "per-account-channel-peer"
        dmScope: "per-channel-peer",
        
        // Group chat isolation
        groupScope: "per-channel",
        
        // Session key for main session
        mainKey: "main"
      },
      
      // Identity linking (same person across channels)
      identityLinks: {
        "user@example.com": ["whatsapp:+1234567890", "telegram:@username"]
      }
    }
  }
}
```

### Multi-Agent Configuration

```json5
{
  agents: {
    // Default settings (applied to all agents)
    defaults: {
      model: "anthropic/claude-opus-4-5",
      workspace: "~/clawd"
    },
    
    // Specific agent configurations
    list: [
      {
        id: "main",
        model: "anthropic/claude-opus-4-5",
        workspace: "~/clawd-personal",
        sandbox: { mode: "off" },  // Full access
        tools: {
          allow: ["*"],  // All tools
          deny: []
        }
      },
      {
        id: "family",
        model: "anthropic/claude-sonnet-4",
        workspace: "~/clawd-family",
        sandbox: {
          mode: "all",
          scope: "agent",
          workspaceAccess: "ro"  // Read-only
        },
        tools: {
          allow: ["read", "sessions_list"],
          deny: ["exec", "write", "browser"]
        }
      },
      {
        id: "public",
        model: "openai/gpt-4o-mini",
        workspace: "~/clawd-public",
        sandbox: {
          mode: "all",
          scope: "session",
          workspaceAccess: "none"
        },
        tools: {
          allow: ["sessions_list", "whatsapp"],
          deny: ["exec", "read", "write", "browser"]
        }
      }
    ]
  }
}
```

### Sandbox Configuration

```json5
{
  agents: {
    defaults: {
      sandbox: {
        // Sandbox mode
        // Options: "off" (no sandbox), "non-main" (sandbox groups),
        //          "all" (sandbox everything)
        mode: "non-main",
        
        // Sandbox scope
        // Options: "shared", "agent", "session"
        scope: "agent",
        
        // Workspace access in sandbox
        // Options: "none", "ro" (read-only), "rw" (read-write)
        workspaceAccess: "none",
        
        // Docker-specific settings
        docker: {
          image: "moltbot/sandbox:latest",
          network: "none",  // Disable network in sandbox
          memory: "512m",
          cpu: "1.0"
        },
        
        // Tool allowlist in sandbox
        allowTools: ["read", "bash"],
        denyTools: ["browser", "web_fetch"]
      }
    }
  }
}
```

---

## Security Settings

### Tool Authorization

```json5
{
  tools: {
    // Elevated mode (host-level access)
    elevated: {
      // Who can use elevated tools
      allowFrom: ["whatsapp:+1234567890"],
      
      // Require approval
      requireApproval: true,
      
      // Approval timeout
      approvalTimeout: 300  // seconds
    },
    
    // Per-tool settings
    browser: {
      enabled: true,
      // Browser profile
      profile: "clawd",
      // Headless mode
      headless: true,
      // Color for browser UI
      color: "#FF4500"
    },
    
    // Web tools
    web: {
      enabled: true,
      search: {
        provider: "brave",
        apiKey: "${BRAVE_API_KEY}"
      }
    }
  }
}
```

### Pairing Configuration

```json5
{
  pairing: {
    // Maximum pending requests per channel
    maxPending: 3,
    
    // Code expiry time
    codeExpiry: 3600,  // seconds
    
    // Auto-approve local connections
    autoApproveLocal: true,
    
    // Require signature for non-local
    requireSignature: true
  }
}
```

---

## Provider Configuration

### Anthropic (Claude)

```json5
{
  providers: {
    anthropic: {
      // API key (or use env var ANTHROPIC_API_KEY)
      apiKey: "${ANTHROPIC_API_KEY}",
      
      // Default model
      defaultModel: "claude-opus-4-5",
      
      // Rate limiting
      rateLimit: {
        requestsPerMinute: 60
      }
    }
  }
}
```

### OpenAI

```json5
{
  providers: {
    openai: {
      apiKey: "${OPENAI_API_KEY}",
      
      // Organization ID (if applicable)
      organization: "org-...",
      
      defaultModel: "gpt-4o",
      
      // Custom base URL (for proxies)
      baseUrl: "https://api.openai.com/v1"
    }
  }
}
```

### OpenRouter

```json5
{
  providers: {
    openrouter: {
      apiKey: "${OPENROUTER_API_KEY}",
      defaultModel: "anthropic/claude-3.5-sonnet"
    }
  }
}
```

### Local Models (Ollama)

```json5
{
  providers: {
    ollama: {
      enabled: true,
      baseUrl: "http://localhost:11434",
      defaultModel: "llama3.2",
      
      // Model mappings
      models: {
        "local-llama": "llama3.2",
        "local-mistral": "mistral"
      }
    }
  }
}
```

### Multiple Provider Fallback

```json5
{
  agents: {
    defaults: {
      // Primary model
      model: "anthropic/claude-opus-4-5",
      
      // Fallback chain
      fallbackModels: [
        "openai/gpt-4o",
        "openrouter/anthropic/claude-3.5-sonnet"
      ]
    }
  }
}
```

---

## Tool Configuration

### Browser Tool

```json5
{
  browser: {
    enabled: true,
    
    // Chrome/Chromium path
    executablePath: "/usr/bin/google-chrome",
    
    // User data directory
    userDataDir: "~/.moltbot/browser-profiles/clawd",
    
    // Headless mode
    headless: true,
    
    // Viewport size
    viewport: {
      width: 1280,
      height: 720
    },
    
    // Extensions
    extensions: [],
    
    // Additional arguments
    args: [
      "--disable-gpu",
      "--no-sandbox"
    ],
    
    // Proxy settings
    proxy: {
      server: "http://proxy.example.com:8080"
    }
  }
}
```

### Cron Tool

```json5
{
  cron: {
    enabled: true,
    
    // Timezone for schedules
    timezone: "America/New_York",
    
    // Maximum concurrent jobs
    maxConcurrent: 5,
    
    // Job timeout
    timeout: 300  // seconds
  }
}
```

### Web Search Tool

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        
        // Provider: "brave", "serpapi", "duckduckgo"
        provider: "brave",
        apiKey: "${BRAVE_API_KEY}",
        
        // Max results per query
        maxResults: 10,
        
        // Safe search
        safeSearch: true
      },
      
      fetch: {
        enabled: true,
        
        // Request timeout
        timeout: 30000,
        
        // Max content size
        maxSize: "10MB",
        
        // Allowed content types
        allowedTypes: ["text/html", "text/plain", "application/json"],
        
        // Blocked domains
        blockedDomains: ["malicious.example.com"]
      }
    }
  }
}
```

---

## Logging Configuration

```json5
{
  logging: {
    // Log level
    // Options: "debug", "info", "warn", "error"
    level: "info",
    
    // Log file path
    file: "/tmp/moltbot/moltbot.log",
    
    // Console output
    console: true,
    
    // Sensitive data redaction
    redactSensitive: "tools",  // Options: "off", "tools", "all"
    
    // Custom redaction patterns
    redactPatterns: [
      "password[=:]\\s*\\S+",
      "token[=:]\\s*\\S+",
      "api[_-]?key[=:]\\s*\\S+"
    ],
    
    // Rotation settings
    rotation: {
      enabled: true,
      maxSize: "10MB",
      maxFiles: 5
    }
  }
}
```

---

## Example Configurations

### Minimal Personal Setup

```json5
// ~/.moltbot/moltbot.json
{
  gateway: {
    bind: "loopback",
    port: 18789,
    auth: {
      mode: "token",
      token: "your-token-here"
    }
  },
  
  agents: {
    defaults: {
      model: "anthropic/claude-opus-4-5",
      workspace: "~/clawd"
    }
  },
  
  channels: {
    whatsapp: {
      dmPolicy: "pairing"
    }
  }
}
```

### High-Security Setup

```json5
{
  gateway: {
    bind: "loopback",
    auth: {
      mode: "token",
      token: "${CLAWDBOT_GATEWAY_TOKEN}"
    },
    controlUi: {
      allowInsecureAuth: false
    }
  },
  
  agents: {
    defaults: {
      model: "anthropic/claude-opus-4-5",
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none"
      },
      tools: {
        allow: ["read", "sessions_list"],
        deny: ["exec", "write", "browser", "config_patch"]
      }
    }
  },
  
  channels: {
    whatsapp: {
      dmPolicy: "pairing",
      allowFrom: [],
      groups: {
        "*": { requireMention: true }
      }
    }
  },
  
  logging: {
    level: "info",
    redactSensitive: "all"
  }
}
```

### Multi-User Team Setup

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: {
      mode: "serve"
    }
  },
  
  agents: {
    defaults: {
      model: "anthropic/claude-sonnet-4",
      sandbox: {
        mode: "all",
        scope: "agent"
      }
    },
    list: [
      {
        id: "dev-team",
        workspace: "/shared/clawd-dev",
        routing: {
          channels: ["slack:#dev-team"]
        }
      },
      {
        id: "ops-team",
        workspace: "/shared/clawd-ops",
        routing: {
          channels: ["slack:#ops-team"]
        }
      }
    ]
  },
  
  channels: {
    slack: {
      botToken: "${SLACK_BOT_TOKEN}",
      appToken: "${SLACK_APP_TOKEN}",
      socketMode: true
    }
  }
}
```

### Cloud/VPS Setup

```json5
{
  gateway: {
    bind: "loopback",
    port: 18789,
    auth: {
      mode: "password",
      password: "${CLAWDBOT_GATEWAY_PASSWORD}"
    },
    remote: {
      token: "${CLAWDBOT_REMOTE_TOKEN}"
    }
  },
  
  discovery: {
    mdns: { mode: "off" }
  },
  
  agents: {
    defaults: {
      model: "anthropic/claude-opus-4-5",
      sandbox: {
        mode: "all",
        docker: {
          enabled: true,
          network: "none"
        }
      }
    }
  }
}
```

---

## Environment Variables

Many configuration options can be set via environment variables:

| Variable | Description |
|----------|-------------|
| `CLAWDBOT_STATE_DIR` | Custom state directory |
| `CLAWDBOT_GATEWAY_TOKEN` | Gateway auth token |
| `CLAWDBOT_GATEWAY_PASSWORD` | Gateway auth password |
| `CLAWDBOT_REMOTE_TOKEN` | Remote gateway token |
| `ANTHROPIC_API_KEY` | Anthropic API key |
| `OPENAI_API_KEY` | OpenAI API key |
| `TELEGRAM_BOT_TOKEN` | Telegram bot token |
| `DISCORD_BOT_TOKEN` | Discord bot token |
| `SLACK_BOT_TOKEN` | Slack bot token |
| `SLACK_APP_TOKEN` | Slack app-level token |

---

## Related Documentation

- [Installation & Setup](./installation-and-setup.md)
- [Usage Examples](./usage-examples.md)
- [Security Analysis](./security-analysis.md)
- [Official Config Reference](https://docs.molt.bot/gateway/configuration)