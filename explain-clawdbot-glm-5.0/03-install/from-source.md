# Build OpenClaw from Source

## Table of contents (Explain OpenClaw - GLM-5.0 Edition)

- [Home (README)](../README.md)
- Plain English
  - [What is OpenClaw?](../01-plain-english/what-is-clawdbot.md)
  - [Glossary](../01-plain-english/glossary.md)
- Technical
  - [Architecture](../02-technical/architecture.md)
  - [Repo map](../02-technical/repo-map.md)
- Privacy + safety
  - [Threat model](../04-privacy-safety/threat-model.md)
  - [Hardening checklist](../04-privacy-safety/hardening-checklist.md)
  - [High privacy config example](../04-privacy-safety/high-privacy-config.example.json5.md)
- Installation
  - [Install and onboarding](./install-and-onboard.md)
  - [Build from source](./from-source.md)
- Deployment
  - [Standalone Mac mini](../05-use-cases/mac-mini-standalone.md)
  - [Isolated VPS](../05-use-cases/vps-isolated.md)

---

This guide explains how to build and install OpenClaw from source code.

Related official docs:
- https://docs.openclaw.ai/install
- https://docs.openclaw.ai/development

---

## Prerequisites

### System requirements

| Requirement | Minimum | Recommended |
|--------------|---------|-------------|
| Operating System | macOS 12+, Ubuntu 20.04+, Debian 11+ | Latest LTS |
| Node.js | 22.12.0 | Latest LTS |
| RAM | 2 GB | 4 GB+ |
| Disk | 10 GB free | 20 GB+ |
| Git | 2.0+ | Latest |

### Install Node.js

**macOS:**
```bash
brew install node
```

**Ubuntu/Debian:**
```bash
# Install Node.js 22.x LTS from NodeSource
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install -y nodejs

# Verify installation
node --version  # Should be v22.12.0 or later
```

### Install pnpm (required for building)

```bash
npm install -g pnpm
```

### Install Rust (optional, for native modules)

Some OpenClaw modules include native Rust code. If building from source:

```bash
# macOS
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Linux
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
```

---

## Step 1: Clone the repository

```bash
# Clone the main repository
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# Or your fork
git clone https://github.com/YOUR_USERNAME/openclaw.git
cd openclaw
```

---

## Step 2: Install dependencies

```bash
# Install all npm dependencies
pnpm install

# Or with npm (slower for this monorepo)
npm install
```

---

## Step 3: Build the project

```bash
# Build all packages
pnpm build

# Or build specific packages
pnpm build --filter=openclaw-gateway

# Build with production optimizations
NODE_ENV=production pnpm build
```

The build output will be in `packages/openclaw-dist/` by default.

---

## Step 4: Link for local development

For development work on OpenClaw itself:

```bash
# Link the built CLI globally
pnpm link --global

# Link specific packages
cd packages/openclaw-cli && pnpm link
cd packages/openclaw-gateway && pnpm link
```

---

## Step 5: Create a local configuration

```bash
# Create a local config directory
mkdir -p ~/.openclaw-local

# Set environment variable for local config
export OPENCLAW_STATE_DIR="$HOME/.openclaw-local"
```

---

## Step 6: Running from source

### Development mode (with file watching)

```bash
# From the CLI package directory
cd packages/openclaw-cli
pnpm dev

# This runs the CLI in watch mode with auto-restart on changes
```

### Production mode (from built binaries)

```bash
# Use the built Gateway binary
./packages/openclaw-dist/openclaw-gateway/unpacked/bin/openclaw gateway

# Or install globally from build output
npm install -g ./packages/openclaw-dist/openclaw-cli-*.tgz
```

---

## Adding Z.AI provider locally

When building from source, the Z.AI provider is included. To test GLM-5.0 integration:

1. Ensure your build includes `src/providers/` directory
2. The provider is registered in `src/agents/synthetic-models.ts` and related files
3. Model definitions are in `src/providers/together-models.ts`

### Testing GLM integration

```bash
# Run with debug output to see provider selection
OPENCLAW_DEBUG=1 openclaw gateway

# Check available models
openclaw models list
```

---

## Development workflow

### Making changes

1. Edit source files in your editor
2. The build/watch process (if running) will recompile
3. Restart the Gateway or CLI to test changes

### Running tests

```bash
# Run all tests
pnpm test

# Run tests for a specific package
pnpm test --filter=openclaw-gateway

# Run with coverage
pnpm test --coverage
```

### Linting

```bash
# Run ESLint
pnpm lint

# Auto-fix issues
pnpm lint --fix
```

---

## Building for production

```bash
# Create an optimized production build
NODE_ENV=production pnpm build

# The output will be in packages/openclaw-dist/
# Create a tarball for distribution
cd packages/openclaw-dist
tar czf openclaw-$(git describe --tags --abbrev=0).tar.gz *
```

---

## Installing from local build

After building, you can install the CLI globally:

```bash
# Install from tarball
npm install -g openclaw-*.tar.gz

# Or directly from the unpacked directory
npm install -g ./packages/openclaw-dist/openclaw-cli-*/
```

---

## Troubleshooting build issues

### "Module not found" errors

- Clear pnpm cache: `pnpm store prune`
- Reinstall dependencies: `rm -rf node_modules && pnpm install`

### Native module build failures

- Ensure Rust is installed: `rustc --version`
- Clean build artifacts: `pnpm clean`

### Type errors

- Regenerate types: `pnpm run postinstall`
- Ensure TypeScript version matches: `tsc --version`

---

## Contributing

When contributing to OpenClaw with GLM-5.0 improvements:

1. Follow the code style in existing files
2. Add tests for new features
3. Update documentation as needed
4. Submit PR with clear description of changes

**Contribution guide:** See `CONTRIBUTING.md` in the repository root for full guidelines.

---

Next: [Install and onboarding](./install-and-onboard.md) for the standard install path
