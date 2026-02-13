# Repo map (where things live)

This is a “what folder do I open?” guide.

## Root
- `README.md`: product overview + install + quickstart.
- `package.json`: runtime requirements, CLI entrypoint, scripts.
- `docs/`: the canonical documentation (Mintlify).
- `src/`: TypeScript source for the CLI + gateway + channels + agent runtime.
- `dist/`: built output (what `npm install -g clawdbot` runs).
- `extensions/`: optional channel plugins.
- `skills/`: bundled skills (treated as trusted code).

## `src/` (high-level)
- `src/cli/`: CLI program and command surfaces.
- `src/commands/`: CLI subcommands (onboarding/wizard, doctor, pairing/devices, etc.).
- `src/gateway/`: the gateway server, auth, protocol, routing, methods.
- `src/*channel*/`: per-channel adapters (telegram/discord/slack/etc.).
- `src/config/`: config types/schema + loading + migration.
- `src/security/`: security audit and related checks.

Tip: if you’re new, read these in order:
1) `README.md`
2) `docs/start/getting-started.md`
3) `docs/gateway/security.md`
4) `docs/gateway/remote.md`
