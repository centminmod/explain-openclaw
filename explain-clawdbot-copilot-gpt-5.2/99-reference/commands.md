# Useful commands (copy/paste)

## Install + onboard

```bash
curl -fsSL https://clawd.bot/install.sh | bash
clawdbot onboard --install-daemon
```

## Run / status

```bash
clawdbot gateway status
clawdbot gateway --port 18789 --verbose
clawdbot status --all
clawdbot health
```

## Security

```bash
clawdbot security audit
clawdbot security audit --deep
clawdbot security audit --fix
```

## Pairing

```bash
clawdbot pairing list telegram
clawdbot pairing approve telegram <CODE>

clawdbot devices list
clawdbot devices approve <requestId>
```

## Remote access (SSH)

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

Docs: https://docs.clawd.bot/gateway/remote
