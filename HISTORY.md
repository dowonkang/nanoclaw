# NanoClaw Setup History

Chronological record of customizations applied to this NanoClaw instance.

---

## 2026-03-08 — Skills System & Initial Feature Setup

### Infrastructure: Branch-Based Skills (`cd65fce`)
- Migrated from plugin-based to branch-based local skill system
- Skills now live as git branches, enabling easier updates and isolation

### Channel: Gmail (`d69c1ab`)
- Installed Gmail channel as a skill branch

### Feature: Voice Transcription (`ef2e094`)
- Added WhatsApp voice message transcription via OpenAI Whisper API
- Migrated to skill branch format

### Feature: Image Vision (`1d17378`)
- Added image analysis for WhatsApp attachments via Claude multimodal

### Feature: PDF Reader (`0993e54`)
- Added PDF text extraction via pdftotext CLI

### Feature: Ollama Tool (`80f23ad`)
- Added Ollama MCP server for local model inference (summarization, translation)

---

## 2026-03-09 — Container Security

### Infrastructure: Credential Proxy (`13ce4aa`)
- Enhanced container environment isolation
- API keys and secrets no longer passed directly to containers

---

## 2026-03-10 — Marketplace & Channel Forks

### Infrastructure: Marketplace Skills (`d572bab`)
- Migrated skills to local project skills from remote marketplace

### Infrastructure: Forked Channels (`f41b399`, `5118239`)
- Channels moved to separate git remotes (forks) with individual registration
- Enables independent channel updates

---

## 2026-03-14 — Remote Control

### Feature: Remote Control (`4e7eb3e`)
- Added `/remote-control` for host-side Claude Code access from chat

---

## 2026-03-17 — Security Hardening

### Security: Prompt Logging Redaction (`cf89904`)
- Stopped logging user prompts on error — prevents accidental secret leaks in logs

---

## 2026-03-18 — Access Control & Container Optimization

### Feature: Admin Mode / Sender Allowlist (`fe0a309`, `4de981b`)
- Per-chat access control with sender allowlist

### Infrastructure: Container Stop Timeout (`b5d0f1e`)
- Reduced container stop timeout for faster shutdown

---

## 2026-03-19 — Diagnostics & Secret Redaction

### Feature: Diagnostics (`f18e507`)
- Added PostHog opt-in telemetry framework

### Security: API Key Redaction (`d2a72e5`)
- Strip secrets from diagnostics output

---

## 2026-03-20 — WhatsApp Channel

### Channel: WhatsApp (`7978241`)
- Merged WhatsApp channel fork (separate remote: `whatsapp/main`)
- Includes emoji reactions support (`ab9abbb`)

---

## 2026-03-22 — Telegram Channel

### Channel: Telegram (`217bc03`)
- Installed Telegram channel fork (separate remote: `telegram/main`)
- IPv4 connectivity fix applied (`e0c01cb`)

### Channel: WhatsApp Disabled (`70d8264`)
- Switched to Telegram-only — WhatsApp channel disabled

---

## 2026-03-23 — Vault Sync, Bot Pool & IPC

### Feature: Host Script IPC (`c91e9d9`)
- Added `run_host_script` IPC for executing host-side commands from container agents

### Feature: Telegram Bot Pool (`fe1bb98`)
- Added bot pool for agent swarm — each subagent gets its own Telegram bot identity

### Feature: Vault Sync (`aaaf91c`)
- Obsidian vault (second-skull) synced before/after scheduled task runs

### Chore: Dependency Update (`1d9455e`)
- Updated dependencies and minor formatting

### Feature: /compact Command (`6b5cd79`)
- Merged `skill/compact` — manual `/compact` session command for context compaction
- Main-group or admin-only access; archives transcript before compacting
- Added `src/session-commands.ts` with command parsing and auth

---

## 2026-03-23 — Telegram Reactions

### Feature: Emoji Reactions for Telegram
- Added incoming reaction support: `bot.on('message_reaction')` stores reactions to SQLite `reactions` table
- Added `sendReaction()` and `reactToLatestMessage()` methods to `TelegramChannel` via `bot.api.setMessageReaction()`
- Added `react_to_message` MCP tool to container agent (`ipc-mcp-stdio.ts`) — agents can now react to messages by emoji
- Added IPC handler for `reaction` type in `ipc.ts`, wired `sendReaction` dep in `index.ts`
- Added `reactions` table to DB schema (auto-migrated on startup)
- Fixed `container-runner.ts` to always sync `ipc-mcp-stdio.ts` from canonical source on each container start, preventing stale MCP tool lists in existing sessions
- Added `container/skills/reactions/SKILL.md` agent documentation

---

## 2026-03-23 — Telegram Reaction Emoji Compatibility

### Fix: Telegram Reaction Invalid Emoji
- Telegram only supports a limited set of reaction emojis; status emojis (✅, ❌, 🔄, 💭) used by container agents caused `REACTION_INVALID` errors
- Added compat map in `TelegramChannel.sendReaction` mapping unsupported emojis to Telegram-supported equivalents: ✅→👍, ❌→👎, 🔄→⚡, 💭→🤔

---

## 2026-03-23 — /clear and /resume Session Commands

### Feature: `/clear` and `/resume` Telegram Commands
- Added `/clear` command: wipes Claude session ID (forces fresh context on next message), saves previous session for restore
- Added `/resume` command: restores the session saved by `/clear`
- Added `deleteSession()` to `src/db.ts` for session cleanup
- Extended `SessionCommandDeps` with `clearSession`, `resumeSession`, `isCommandSenderAuthorized`
- Fixed authorization: session commands now allowed for trigger-allowlisted senders (not just `is_from_me`), enabling Telegram use
- Fixed Telegram `@botname` normalization: `/clear@botname` → `/clear` before storage (`src/channels/telegram.ts`)
- Fixed cursor ordering: `advanceCursor` now called only after successful send, enabling retry on failure
- Fixed silent failure: `TelegramChannel.sendMessage` now rethrows errors after logging; session command handler catches and returns `success: false` to trigger GroupQueue retry

---

## 2026-03-24 — CI Fix & Vault Sync Fix

### Fix: update-tokens CI Workflow
- `actions/create-github-app-token` required `APP_ID` / `APP_PRIVATE_KEY` secrets that only exist on upstream, not in fork
- Replaced GitHub App token steps with built-in `GITHUB_TOKEN` on the checkout step

### Fix: Obsidian Sync Broken Under launchd (`040e384`)
- `syncVault` called `bun run sync` as a bare command, but NanoClaw's launchd plist PATH (`/usr/local/bin:/usr/bin:/bin`) excludes `/opt/homebrew/bin` where `bun` lives
- Fixed by using full path `/opt/homebrew/bin/bun run sync` in `src/task-scheduler.ts`

---

## 2026-03-23 — Agent Swarm Group & 3rd Pool Bot

### Config: Telegram Bot Pool Expanded
- Added 3rd pool bot (`maclaw_swarm_3_bot`) to `TELEGRAM_BOT_POOL`
- Fixed: plist `EnvironmentVariables` overrides `.env` — updated both files to keep in sync
- Registered new Telegram group for agent swarm use

---

## Installed Skill Branches

| Branch | Purpose |
|--------|---------|
| `skill/ollama-tool` | Local model inference via MCP |
| `skill/compact` | Manual context compaction command |
| `skill/apple-container` | macOS-native container runtime |
| `feat/remote-control` | Host Claude Code access from chat |
| `feature/diagnostics` | PostHog telemetry framework |

## Channel Remotes

| Remote | Channel | Status |
|--------|---------|--------|
| `telegram/main` | Telegram | Active |
| `whatsapp/main` | WhatsApp | Disabled |

## Other Merged Channels

- **Slack** (`ee7f720`) — merged via /add-slack
- **Discord** (`d5f3370`) — migrated to branch-based skill
- **Gmail** (`d69c1ab`) — skill branch based
