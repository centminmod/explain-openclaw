# Where Clawdbot stores state on disk

Clawdbot persists sessions and credentials under `~/.clawdbot/`.
This is the main privacy boundary.

The repo’s security doc lists a credential map:

- WhatsApp: `~/.clawdbot/credentials/whatsapp/<accountId>/creds.json`
- Pairing allowlists: `~/.clawdbot/credentials/<channel>-allowFrom.json`
- Model auth profiles: `~/.clawdbot/agents/<agentId>/agent/auth-profiles.json`
- Session transcripts: `~/.clawdbot/agents/<agentId>/sessions/*.jsonl`
- Legacy OAuth import: `~/.clawdbot/credentials/oauth.json`

Source: `docs/gateway/security.md`.

Practical tips:
- Keep `~/.clawdbot` permissions tight (audit can set `700` / `600`).
- Do not sync this directory to iCloud/Dropbox/etc.
- Back up only what you understand, encrypted.
