# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is an **OpenClaw** workspace with a **Feishu/Lark plugin** integration. OpenClaw is an AI agent framework that connects to messaging channels (Feishu/Slack/etc.) and provides tools for tasks like document management, calendar, tasks, and bitable operations.

## Architecture

```
~/.openclaw/
├── workspace/           # Agent workspace (skills, agents, docs)
│   ├── skills/          # Skill definitions (feishu-message, etc.)
│   └── agents/          # Agent configs (tech, writer, researcher, etc.)
├── extensions/
│   └── feishu-openclaw-plugin/  # Feishu plugin (Node.js, ES modules)
│       ├── src/
│       │   ├── channel/         # Feishu channel implementation
│       │   ├── tools/           # Tool implementations (oapi, mcp, tat)
│       │   ├── messaging/       # Inbound/outbound message handling
│       │   ├── card/            # Interactive card components
│       │   └── core/            # Core utilities (lark-client, trace)
│       └── skills/              # Skill Markdown docs
└── openclaw.json        # Runtime configuration
```

### Plugin Structure (`extensions/feishu-openclaw-plugin/`)

- **Entry**: `index.js` - Registers channel, tools, CLI commands
- **Tools**: Organized by API type:
  - `tools/oapi/` - Direct Feishu OpenAPI calls (calendar, task, bitable, im, wiki, drive)
  - `tools/mcp/` - Model Context Protocol tools (doc fetch/create/update)
  - `tools/tat/` - Tool-as-a-Service patterns
- **Skills**: Markdown-based skill definitions in `skills/` with usage guidelines

### Skill System

Skills are Markdown files (`SKILL.md`) that define:
- When to activate the skill
- Tool usage patterns with parameter examples
- Common error codes and troubleshooting
- Domain-specific constraints (e.g., field value formats for Bitable)

Key skills:
- `feishu-message` - Send/receive Feishu messages
- `feishu-create-doc`, `feishu-update-doc`, `feishu-fetch-doc` - Document CRUD
- `feishu-bitable` - Bitable (Airtable-like) operations with 27 field types
- `feishu-calendar` - Calendar/event management, free/busy lookup
- `feishu-task` - Task/tasklist management

## Commands

### OpenClaw CLI

```bash
# Start the gateway (Feishu channel listener)
openclaw gateway run

# View plugin/channel status
openclaw plugins list
openclaw channels status

# Configuration
openclaw config get channels.feishu
openclaw config set channels.feishu.requireMention true --json

# Pairing (when bot sends a code in Feishu)
openclaw pairing approve feishu <code> --notify

# Plugin-specific diagnostics
openclaw feishu-diagnose                    # Check config/permissions
openclaw feishu-diagnose --trace <msg_id>   # Trace message handling
```

### Plugin Commands (in Feishu chat)

- `/feishu start` - Verify installation
- `/feishu doctor` - Run diagnostics
- `/feishu auth` - Complete batch authorization

## Configuration

Key `openclaw.json` settings under `channels.feishu`:

| Setting | Values | Default |
|---------|--------|---------|
| `replyMode` | `auto`/`streaming`/`static` | `auto` |
| `dmPolicy` | `open`/`pairing`/`allowlist` | `pairing` |
| `groupPolicy` | `open`/`allowlist`/`disabled` | `allowlist` |
| `requireMention` | `boolean`/`open` | `true` |

## Key Conventions

### User ID Format

All tools use **open_id** format (`ou_xxxxxx`), not user_id or union_id. Always extract from `SenderId` in message context.

### Time Format

- Use **ISO 8601 with timezone**: `2026-02-25T14:00:00+08:00`
- Do NOT use Unix timestamps (seconds/milliseconds)
- Timezone is fixed: `Asia/Shanghai` (UTC+8)

### Bitable Field Value Formats

| Type | Format | Example |
|------|--------|---------|
| User | `[{id: "ou_xxx"}]` | `[{id: "ou_abc123"}]` |
| Date | Millisecond timestamp | `1674206443000` |
| SingleSelect | String | `"选项 A"` |
| MultiSelect | String array | `["A", "B"]` |
| Url | Object | `{link: "...", text: "..."}` |

### Document Markdown (Lark-flavored)

- Images/files: Use `<image url="..."/>` and `<file url="..."/>` - system auto-uploads
- Callouts: `<callout emoji="💡" background-color="light-blue">...</callout>`
- Tables: Use `<lark-table>` for complex content (supports nested blocks)
- Mermaid: ``` ```mermaid ... ``` ``` auto-converts to whiteboard

## Common Pitfalls

1. **Bitable empty rows**: `app.create` includes empty records - delete them first with `record.list` + `batch_delete`
2. **Doc media tokens**: `<image token="..."/>` is read-only; use `url` for creation
3. **Wiki node type**: `/wiki/TOKEN` could be docx/sheet/bitable - query `wiki_space_node.get` first
4. **Task ownership**: Always pass `current_user_id` when creating tasks to ensure editability
