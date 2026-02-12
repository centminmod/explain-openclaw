# Resource Usage Analysis (CPU, Memory, Disk)

> **Verified by:** 3 Explore agents, LSP symbol checks, direct file reads, and [`/consult-codex` dual-AI consultation](https://github.com/centminmod/my-claude-code-setup/tree/master/.claude/skills/consult-codex/SKILL.md) (Codex GPT-5.3 + Code-Searcher). All findings independently confirmed with high agreement.

Users report OpenClaw can be resource-intensive. This guide documents every resource-consuming subsystem with verified source references, plain-English explanations, and practical optimization strategies.

---

## A. CPU-Intensive Operations

*Think of CPU usage like a chef in a kitchen. Most of the time they're waiting for orders, but some tasks — like hand-rolling 42 different sushi pieces just to pick the smallest one — keep them frantically busy. These are the operations that make your fans spin and your machine feel sluggish.*

### Ranked by impact

| # | Operation | Source | Impact | Plain English |
|---|-----------|--------|--------|---------------|
| 1 | **Screenshot normalization** — nested loop of up to 7 sizes x 6 qualities = 42 sharp resize ops per screenshot | `src/browser/screenshot.ts:34-51` | Very High | Like resizing a photo 42 different ways to find which version fits in an envelope — each resize takes real effort |
| 2 | **PNG image optimization** — grid of 5 sizes x 4 compression levels = 20 sharp ops (mozjpeg is CPU-heavy) | `src/media/image-ops.ts:400-457` | Very High | Like printing the same photo in 20 different quality settings to find the smallest file — each print job takes CPU time |
| 3 | **Local embedding inference** — on-device GGUF model via node-llama-cpp, `Promise.all` over all texts | `src/memory/embeddings.ts:82-128` | Very High (when local) | Like running a mini-ChatGPT on your own machine to understand your notes — powerful but demands serious CPU |
| 4 | **Plugin loading via jiti** — synchronous TypeScript transpilation per plugin at startup | `src/plugins/loader.ts:211-296` | High (startup) | Like compiling a recipe book from scratch every time you open the kitchen, instead of using a pre-printed copy |
| 5 | **Cosine similarity fallback** — O(n) full-scan vector comparison when sqlite-vec unavailable | `src/memory/manager-search.ts:71-93` | High (per query) | Like comparing a new photo to every single photo in your album one-by-one, instead of using a smart index |
| 6 | **PDF-to-image rendering** — per-page canvas creation + PNG encoding via `@napi-rs/canvas` | `src/media/input-files.ts:197-254` | High (per PDF) | Like photocopying each page of a PDF into a separate image file — each page takes a rendering pass |
| 7 | **Full AX tree traversal** — `Accessibility.getFullAXTree` on complex browser pages | `src/browser/cdp.ts:251-264` | Medium-High | Like reading every element on a web page aloud for accessibility — hundreds of elements on complex pages |
| 8 | **Image resize via sips** — macOS-specific process spawning for each HEIC conversion/resize | `src/media/image-ops.ts:136-274` | Medium | Like opening a separate program for each photo conversion — the per-process overhead adds up |
| 9 | **Media understanding** — sending media to AI providers (Whisper/Gemini/OpenAI) for transcription | `src/media-understanding/runner.ts:802-984` | Medium | CPU cost is mostly on the provider side, but local buffering and encoding still takes cycles |
| 10 | **Ed25519 keypair generation** — asymmetric crypto on first run / device identity creation | `src/infra/device-identity.ts:57` | Low (one-time) | Like generating a strong password — intensive but happens only once |
| 11 | **Memory sync** — file hashing + markdown chunking + embedding + SQLite FTS5/vec indexing | `src/memory/manager.ts:391+` | Medium (periodic) | Like re-indexing a library catalog — scanning, categorizing, and filing every document |
| 12 | **TTS generation** — ElevenLabs/OpenAI/Edge TTS API calls + audio buffer handling | `src/tts/tts.ts:1043-1229` | Medium | API calls are remote but audio buffer conversion is local CPU work |
| 13 | **Agent execution loop** — continuous model response processing | `src/auto-reply/reply/agent-runner-execution.ts:101` | Medium (continuous) | The main "brain" loop — always running while the bot is responding |
| 14 | **Cron timer loop** — infinite loop for scheduled job processing | `src/cron/service/timer.ts:422` | Low (idle) | Like a clock ticking in the background — minimal CPU unless jobs are firing |

### Other CPU consumers

**Polling and reconnection loops:**
- Signal SSE event stream — long-lived HTTP connection with reconnection logic
- Telegram long-polling — continuous `getUpdates` calls to Telegram API
- WhatsApp monitor — persistent connection with keepalive

**Browser extension WebSocket pings:**
- `src/browser/extension-relay.ts:503` — periodic ping/pong to keep connections alive

**Crypto operations:**
- HMAC signature verification for webhooks
- SHA-256 hashing for content deduplication and caching

**Child process stdout/stderr accumulation:**
- `src/process/exec.ts:114-141` — unbounded string concatenation of process output
- `src/memory/qmd-manager.ts:553-575` — same pattern for QMD processing

**Media fetch buffering:**
- `src/media/fetch.ts:131-133` — full response body buffered into memory before processing

---

## B. Memory-Intensive Operations

*Memory (RAM) is like your desk space. The more papers, sticky notes, and browser tabs you keep open, the more crowded it gets. Some parts of OpenClaw are tidy — they clean up after themselves. Others keep piling up papers and never throw anything away, eventually leaving no room to work.*

### In-memory caches and their bounds

| Cache | Location | Bound | Risk |
|-------|----------|-------|------|
| Session store cache | `src/config/sessions/store.ts:35` | 45s TTL, `structuredClone` per read | Medium — each entry holds all 500 sessions |
| Discord presence cache | `src/discord/monitor/presence-cache.ts:9` | 5000/account LRU | Low |
| Telegram sent message cache | `src/telegram/sent-message-cache.ts:13` | 24h TTL, 100/chat | Low-Medium |
| History map | `src/auto-reply/reply/history.ts:7` | 1000 keys LRU | Well bounded |
| Inbound dedupe | `src/auto-reply/reply/inbound-dedupe.ts:8` | 5000 max, 20min TTL | Well bounded |
| Gateway dedupe | `src/gateway/server-constants.ts:33-34` | 1000 max, 5min TTL | Well bounded |
| Browser roleRefs | `src/browser/pw-session.ts:96-97` | 50 max LRU | Well bounded |
| Followup queues | `src/auto-reply/reply/queue/state.ts:20` | 20/queue, no queue count cap | **Unbounded queues** |
| Agent event seqByRun | `src/infra/agent-events.ts:21` | **No cleanup** | **Leak risk** |
| Agent run sequence | `src/gateway/server-runtime-state.ts:180` | **No pruning** (maintenance timer skips it) | **Leak risk** |
| WhatsApp group histories | `src/web/auto-reply/monitor.ts:103` | Helper has 1000-key cap, but web direct writes bypass it | **Partial leak** |
| WhatsApp group member names | `src/web/auto-reply/monitor.ts:113` | **No eviction at all** | **Leak risk** |
| Cost usage cache | `src/gateway/server-methods/usage.ts:47` | 30s TTL per entry, **no max entry count** | Low-Medium |
| Warned contexts | `src/infra/session-maintenance-warning.ts:14` | **Never pruned** | Low |
| Announce queues | `src/agents/subagent-announce-queue.ts:45` | Per-queue cap, **no queue count cap** | Low |
| Telegram sent msgs outer map | `src/telegram/sent-message-cache.ts:13` | Per-chat TTL, **outer map never evicts dead chat keys** | Low-Medium |

> *Session store cache:* Like photocopying an entire filing cabinet every time you check one folder — works, but wastes desk space.

> *Unbounded Maps (seqByRun, agentRunSeq):* Like a guest book that records every visitor but never tears out old pages — after months, it's a thick ledger eating memory for no reason.

### Browser memory

- **Chromium instance** (Playwright CDP): `src/browser/pw-session.ts:103` — singleton, but Chromium itself can consume **200MB to 2GB+**
  > *Like having a full web browser running invisibly in the background — it alone can use more memory than everything else combined.*
- Per-page state caps: console (500), errors (200), network requests (500) — `src/browser/pw-session.ts:99-101`
- WeakMaps used for page/context state (GC-friendly): `src/browser/pw-session.ts:89-92`

### Model context accumulation

The conversation with the AI model grows with every message. Left unchecked, this would eat unlimited memory.

> *Like a conversation transcript that keeps growing — OpenClaw has a "summarizer" that periodically condenses old messages, like a meeting secretary writing "minutes" instead of keeping the full recording.*

OpenClaw has several defenses:
- **Compaction system:** `src/agents/compaction.ts` — multi-stage summarization to prevent unbounded context growth
- **History turn limiting:** `src/agents/pi-embedded-runner/history.ts:15-36`
- **Context pruning extension:** `src/agents/pi-extensions/context-pruning/extension.ts`
- **WebSocket payload limits:** 512KB/frame, 1.5MB/connection buffer, 6MB history — `src/gateway/server-constants.ts:1-4`

### Plugin memory

Modules loaded via jiti persist for process lifetime. Each plugin's tools, commands, and hooks are registered in global maps and never unloaded.

---

## C. Disk-Intensive Operations

*Disk usage is like storage boxes in your garage. Some boxes are neatly labeled and rotated out seasonally (managed). Others just keep growing — like never deleting old text messages — until one day your phone says "Storage Full." OpenClaw has both kinds, and the unbounded ones are the ones to watch.*

### Unbounded growth risks (no rotation/pruning)

| Resource | Location | Risk |
|----------|----------|------|
| Transcript `.jsonl` files | `src/config/sessions/transcript.ts:60-151` | **No rotation, no size limit** — grows forever per session |
| Command logger | `src/hooks/bundled/command-logger/handler.ts:42-58` | **No rotation** — `commands.log` grows unbounded |
| Telegram sticker cache | `src/telegram/sticker-cache.ts:35-67` | **No eviction** — JSON grows with unique stickers |
| Browser user-data profiles | `src/browser/chrome.ts:62-64` | Full Chromium profile — can reach GBs |
| SQLite databases | `src/memory/manager.ts:704-713` | **No VACUUM** — WAL files can bloat |
| Per-day log file size | `src/logging/logger.ts:102-109` | No cap on individual file size |
| Voice-call `calls.jsonl` | `extensions/voice-call/src/manager/store.ts:7-10` | **Append-only, no rotation** + full-file reads on load |

> *Transcript JSONL files:* Like a chat log that records every message forever but never archives or deletes old conversations — a busy bot can accumulate gigabytes over months.

> *Browser user-data profiles:* Like a real web browser's cache, cookies, and history — it grows the more pages the bot visits, just like your own browser.

> *commands.log:* Like a security camera that records 24/7 but never overwrites old footage — eventually the DVR fills up.

### Bounded/managed resources

| Resource | Limit | Location |
|----------|-------|----------|
| Media files | 2min TTL auto-cleanup | `src/media/store.ts:15,67-83` |
| Rolling logs | 24h age pruning | `src/logging/logger.ts:18,227-251` |
| Session store | 500 entries, 30d prune, 10MB rotation, 3 backups | `src/config/sessions/store.ts:208-210` |
| Cron run logs | 2MB/2000 lines self-pruning | `src/cron/run-log.ts:26-57` |
| TTS temp files | 5min delayed cleanup | `src/tts/tts.ts:42,989-998` |
| Pairing requests | 3/channel, 1h TTL | `src/pairing/pairing-store.ts:14-15` |

### Size limits on inbound data

| Type | Limit | Location |
|------|-------|----------|
| Images | 6MB (10MB input files) | `src/media/constants.ts:1`, `src/media/input-files.ts:101` |
| Audio | 16MB | `src/media/constants.ts:2` |
| Video | 16MB | `src/media/constants.ts:3` |
| Documents | 100MB | `src/media/constants.ts:4` |
| WS frame | 512KB | `src/gateway/server-constants.ts:1` |
| WS buffer | 1.5MB/connection | `src/gateway/server-constants.ts:2` |
| Browser screenshot | 5MB | `src/browser/screenshot.ts:4` |

### No disk-space checks

There are **zero** free-space checks (`statvfs`/disk usage) anywhere in the codebase. OpenClaw will keep writing until the disk is completely full, with no warning.

> *Like filling a storage unit without ever checking how much room is left — OpenClaw will keep writing until the disk is completely full.*

---

## D. Optimization Strategies for Users

*You don't need to understand the code to keep your OpenClaw instance running lean. Think of these tips like maintaining a car: you don't need to be a mechanic, but changing the oil and checking tire pressure goes a long way. Here's the equivalent for OpenClaw.*

### CPU optimizations

1. **Disable browser tools if not needed** — saves Chromium CPU overhead entirely
2. **Use API-based embedding providers** instead of local GGUF models — offloads inference to the cloud
3. **Minimize plugins** to reduce jiti transpilation at startup
4. **Avoid frequent PDF processing / large screenshots** — each triggers heavy image processing loops
5. **Use the sqlite-vec extension** — avoids the O(n) cosine similarity fallback on every memory query

### Memory optimizations

1. **Configure `dmHistoryLimit`** to cap conversation context per session
2. **Limit active browser profiles** / close unused pages — each Chromium page holds state
3. **Disable unused channels** — each channel adapter holds connection state and caches
4. **Monitor WhatsApp group count** if using auto-reply — group history Maps are unbounded
5. **Restart Gateway periodically** if running for weeks — clears accumulated `seqByRun` / run sequence leaks

### Disk optimizations

1. **Periodically clean transcript files:**
   ```bash
   # List largest transcripts
   du -sh ~/.openclaw/agents/*/sessions/*.jsonl | sort -rh | head -20
   ```
   Transcripts are never auto-pruned.

2. **Periodically clean browser profiles:**
   ```bash
   du -sh ~/.openclaw/browser/*/user-data/
   ```
   Chromium profiles grow indefinitely.

3. **Run SQLite VACUUM periodically:**
   ```bash
   sqlite3 ~/.openclaw/memory/*.sqlite VACUUM
   ```
   WAL files can bloat without periodic compaction.

4. **Monitor `commands.log` size** — it grows unbounded:
   ```bash
   ls -lh ~/.openclaw/logs/commands.log
   ```

5. **Clean orphaned temp directories:**
   ```bash
   ls -lh /tmp/tts-* /tmp/openclaw-* 2>/dev/null
   ```

6. **Set `logging.level` to `warn` in production** to reduce log volume

7. **Use session maintenance config:**
   - `sessions.pruneAfter` — auto-prune old sessions
   - `sessions.maxEntries` — cap total session count
   - `sessions.rotateBytes` — rotate session store file at size threshold

---

## E. Monitoring & Profiling Guide

*Sections A-C told you **what** consumes CPU, memory, and disk. This section tells you **how to see it happening** on your actual machine. Think of it like the difference between a thermometer (one quick reading) and a thermograph (a chart that records temperature all day). You need both: quick checks to see what's happening right now, and historical tools to spot trends before they become problems.*

### Quick process identification

Before monitoring anything, find the OpenClaw Gateway process:

```bash
# Find the Gateway PID (both platforms)
pgrep -f "openclaw"

# See full process details (both platforms)
ps aux | grep openclaw

# macOS — also check Activity Monitor (search "openclaw" or "node")
# The Gateway runs as a Node.js process, so look for "node" if "openclaw" doesn't appear
```

> *Tip:* Save the PID in a variable for repeated use: `OC_PID=$(pgrep -f "openclaw")`

### Real-time monitoring (point-in-time)

*These tools are like glancing at your car's dashboard — they show you what's happening right now, but don't record history.*

**top / htop** (both platforms)

```bash
# Filter top for the OpenClaw process
# Linux:
top -p $(pgrep -f "openclaw")
# macOS (different flag):
top -pid $(pgrep -f "openclaw")

# htop — a friendlier version with color-coded bars (both platforms, same flag)
# Install: brew install htop (macOS) / apt install htop (Linux)
htop -p $(pgrep -f "openclaw")
```

**Per-process one-liner** (both platforms)

```bash
# Snapshot of CPU% and memory for the Gateway
ps -p $(pgrep -f "openclaw") -o pid,%cpu,%mem,rss,vsz,command
```

Column reference: `%cpu` = CPU percentage, `%mem` = RAM percentage, `rss` = resident set size (actual RAM in KB), `vsz` = virtual size (allocated, not all physical).

**macOS Activity Monitor:**
Open Activity Monitor from Spotlight (`Cmd+Space` → "Activity Monitor"), then search for `openclaw` or `node`. The Memory tab shows Real Memory (same as RSS) and the CPU tab shows per-process usage.

### sysstat tutorial (Linux VPS)

*`top` and `htop` show you what's happening right now — like glancing at a speedometer. But what if you want to know how fast you were going at 3 AM when nobody was watching? That's what sysstat does. It's a suite of tools that automatically records system performance every 10 minutes and lets you replay the data later. Think of it as a dashcam for your server.*

#### What's included

| Tool | Purpose |
|------|---------|
| `sar` | **S**ystem **A**ctivity **R**eporter — replays historical data (CPU, memory, disk, network) |
| `pidstat` | Per-process stats (like `top`, but with timestamps and logging) |
| `iostat` | Disk I/O throughput and latency |
| `mpstat` | Per-CPU core breakdown |

#### Install

```bash
# Debian/Ubuntu
sudo apt install sysstat

# RHEL/CentOS/Fedora
sudo dnf install sysstat    # or: sudo yum install sysstat

# Enable automatic data collection (records every 10 min by default)
sudo systemctl enable --now sysstat
```

After enabling, sysstat stores daily data files in `/var/log/sa/` (or `/var/log/sysstat/`). You can replay any day's data — even from last week.

#### sar — system-wide historical replay

```bash
# CPU usage over time (today) — shows %user, %system, %idle
sar -u

# CPU usage for a specific date (e.g., February 10)
sar -u -f /var/log/sa/sa10

# CPU usage sampled every 5 seconds, 12 times (1 minute of live data)
sar -u 5 12

# Memory usage over time — shows kbmemfree, kbmemused, %memused
sar -r

# Disk I/O — shows tps (transfers per second), read/write throughput
sar -b          # block device summary
sar -d          # per-device breakdown (useful to identify which disk)

# Network interface stats — shows rxkB/s, txkB/s per interface
sar -n DEV
```

> *Reading sar output:* Each row is a timestamp. Look for `%idle` dropping below 20 (CPU saturated), `%memused` above 90 (memory pressure), or sudden spikes in `tps` (disk I/O storms). If your Gateway was sluggish at 3 AM, run `sar -u -f /var/log/sa/sa$(date -d yesterday +%d)` to see yesterday's CPU.

#### pidstat — per-process monitoring

*`pidstat` is like `top`, but it prints timestamped lines you can log to a file. Perfect for watching OpenClaw specifically without the noise of other processes.*

```bash
# Per-process CPU usage for the Gateway
pidstat -p $(pgrep -f "openclaw") 1
# Prints a line every 1 second showing %usr, %system, %CPU

# Per-process memory (RSS, VSZ, %MEM)
pidstat -r -p $(pgrep -f "openclaw") 5
# Prints memory stats every 5 seconds

# Per-process disk I/O (kB read/written per second)
pidstat -d -p $(pgrep -f "openclaw") 5

# Combined: CPU + memory + disk, every 5 seconds for 1 minute (12 samples)
pidstat -urd -p $(pgrep -f "openclaw") 5 12

# Monitor ALL Node.js processes (useful if OpenClaw spawns children)
pidstat -urd -C "node" 5
```

> *Pro tip:* Pipe `pidstat` output to a file for later analysis:
> ```bash
> pidstat -urd -p $(pgrep -f "openclaw") 60 1440 > ~/openclaw-24h-stats.log &
> ```
> This logs CPU/memory/disk every 60 seconds for 24 hours (1440 samples).

#### macOS note

sysstat is **not available on macOS**. Use these alternatives instead:

```bash
# Memory pressure (macOS equivalent of sar -r)
vm_stat 5
# Columns: free, active, inactive, wired — multiply by page size (16384 on Apple Silicon)

# Disk I/O (macOS equivalent of sar -b / iostat)
iostat -w 5
# Shows KB/t, tps, MB/s per disk

# Per-process stats — use ps in a loop (macOS equivalent of pidstat)
while true; do
  date; ps -p $(pgrep -f "openclaw") -o %cpu,%mem,rss 2>/dev/null
  sleep 5
done
```

### Disk I/O monitoring

*If your OpenClaw instance is slow but CPU and memory look fine, the bottleneck might be disk. These tools tell you who's reading/writing and how fast.*

```bash
# iostat — disk throughput snapshot
# Linux (extended stats):
iostat -x 5       # every 5 seconds; key columns: r/s, w/s, %util
# macOS (different flags — no -x support):
iostat -w 5       # every 5 seconds; shows KB/t, tps, MB/s per disk

# iotop — who's doing the I/O (Linux, requires root)
sudo iotop -o     # only show processes with active I/O
# Look for "node" or "openclaw" — heavy writes may be transcript/log accumulation

# macOS — use fs_usage to trace file system calls
sudo fs_usage -f filesys node   # trace all filesystem calls by Node.js processes
```

### Node.js profiling (OpenClaw-specific)

*When you've confirmed that the Gateway is using too much CPU or memory, these tools let you look **inside** the Node.js process to find exactly which function is responsible.*

#### Chrome DevTools (CPU profile + heap snapshot)

```bash
# Start the Gateway with the inspector enabled
openclaw --inspect=0.0.0.0:9229

# Or attach to an already-running Gateway (send SIGUSR1)
kill -USR1 $(pgrep -f "openclaw")
```

Then open `chrome://inspect` in Chrome/Chromium, click the OpenClaw target, and:
- **CPU Profile tab** → Record → trigger the slow operation → Stop → see which functions took the most time
- **Memory tab** → Take Heap Snapshot → see what objects are consuming memory (look for the unbounded Maps from Section B)

#### clinic.js (automated diagnosis)

```bash
# Install
npm install -g clinic

# Detect CPU bottlenecks (generates an HTML flamegraph)
clinic doctor -- node /path/to/openclaw/gateway.js

# Generate a flamegraph (visual call-stack breakdown)
clinic flame -- node /path/to/openclaw/gateway.js

# Detect async bottlenecks (event loop delays)
clinic bubbleprof -- node /path/to/openclaw/gateway.js
```

#### 0x flamegraph (lightweight alternative)

```bash
# Install
npm install -g 0x

# Profile the Gateway (press Ctrl+C to stop and generate flamegraph)
0x -- node /path/to/openclaw/gateway.js

# Auto-open flamegraph in browser when done
0x -o -- node /path/to/openclaw/gateway.js
# Wider bars in the flamegraph = more CPU time spent in that function
```

### Network monitoring

*OpenClaw communicates via WebSocket (Gateway port 18789), HTTP APIs, and outbound connections to AI providers. These tools help you see what's connected and how much traffic is flowing.*

```bash
# Who's connected to the Gateway WebSocket port? (both platforms)
lsof -i :18789
# Shows every client connected — useful to check if browser extension / web UI is attached

# List all connections by the Gateway process (Linux)
ss -tnp | grep openclaw
# Shows established TCP connections, remote IPs, and ports

# macOS equivalent
lsof -i -P -n | grep openclaw

# Count active WebSocket connections
lsof -i :18789 | grep -c ESTABLISHED

# Watch connection count over time
watch -n 5 'lsof -i :18789 | grep -c ESTABLISHED'
```

### Disk size checks

*These commands give you a quick health check on the biggest disk consumers documented in Section C.*

```bash
# Check total state directory size
du -sh ~/.openclaw/

# Check transcript sizes (largest first)
du -sh ~/.openclaw/agents/*/sessions/*.jsonl | sort -rh | head -20

# Check browser profile sizes
du -sh ~/.openclaw/browser/*/user-data/

# Check SQLite sizes
ls -lh ~/.openclaw/memory/*.sqlite

# Check log sizes
ls -lh /tmp/openclaw/

# Check temp file accumulation
ls -lh /tmp/tts-* /tmp/openclaw-* 2>/dev/null

# Check commands.log
ls -lh ~/.openclaw/logs/commands.log
```

### Automated monitoring over time

*One-time checks are useful, but problems often creep in gradually. A cron job that logs resource usage lets you spot trends — like memory slowly growing or transcripts silently ballooning.*

#### Simple RSS logger (both platforms)

```bash
# Save this as ~/openclaw-monitor.sh
#!/bin/bash
TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')
PID=$(pgrep -f "openclaw")
if [ -n "$PID" ]; then
  RSS=$(ps -p "$PID" -o rss= 2>/dev/null | tr -d ' ')
  CPU=$(ps -p "$PID" -o %cpu= 2>/dev/null | tr -d ' ')
  DISK=$(du -sm ~/.openclaw/ 2>/dev/null | cut -f1)
  echo "$TIMESTAMP  PID=$PID  RSS_KB=$RSS  CPU=$CPU%  DISK_MB=$DISK" >> ~/openclaw-resource.log
fi

# Make executable
chmod +x ~/openclaw-monitor.sh
```

#### Cron schedule

```bash
# Log every 5 minutes (add with: crontab -e)
*/5 * * * * ~/openclaw-monitor.sh
```

After a few days, review trends:

```bash
# See memory trend (RSS column)
awk '{print $1, $2, $4}' ~/openclaw-resource.log | tail -50

# Check if disk usage is climbing
awk '{print $1, $2, $6}' ~/openclaw-resource.log | tail -50
```

#### Linux: leverage sysstat automatic collection

If you enabled sysstat (see above), it already collects system-wide stats every 10 minutes. No extra cron needed — just query with `sar` whenever you want historical data.

#### When to consider Prometheus/Grafana

For most single-instance deployments (Mac mini, small VPS), the tools above are sufficient. Consider Prometheus + Grafana only if:
- You run **multiple OpenClaw instances** and need centralized dashboards
- You want **alerting** (e.g., "notify me if RSS exceeds 2GB")
- You're already running a monitoring stack for other services

Setting up Prometheus/Grafana is beyond the scope of this guide — see the [Prometheus docs](https://prometheus.io/docs/introduction/overview/) for getting started.

---

## Summary

| Resource | Biggest risks | Key mitigation |
|----------|--------------|----------------|
| **CPU** | Screenshot normalization (42 ops), image optimization (20 ops), local embeddings | Disable browser tools, use API embeddings, install sqlite-vec |
| **Memory** | Chromium (200MB-2GB), unbounded Maps (`seqByRun`, group histories), session store cloning | Periodic Gateway restart, limit browser usage, cap history |
| **Disk** | Transcript JSONL (never pruned), browser profiles (GBs), `commands.log` (unbounded) | Manual cleanup, SQLite VACUUM, session maintenance config |
