# OpenClaw Configuration Backup

This repository contains a backup of my OpenClaw configuration with sensitive credentials excluded.

## What's Included

- **CLAUDE.md** - Project documentation and architecture guide
- **extensions/feishu-openclaw-plugin/** - Feishu/Lark plugin source code
- **.gitignore** - Excludes sensitive files
- **.env.example** - Environment variable template
- **openclaw.example.json** - Configuration template (fill in your own values)

## What's Excluded (Sensitive)

- `openclaw.json` - Contains API keys and secrets
- `credentials/` - Credential files
- `devices/` - Device identity data
- `identity/` - User identity data
- `memory/` - Session memory
- `logs/` - Runtime logs
- `agents/` - Agent configurations with credentials
- `.claude/` - Local Claude Code settings
- `workspace/` - Has its own git repository

## Restore Instructions

### Prerequisites

1. Install OpenClaw: `npm install -g openclaw`
2. Clone this repository to `~/.openclaw/`

### Step 1: Create .env file

```bash
cp .env.example .env
```

Then edit `.env` and fill in your actual API keys:
- `DASHSCOPE_API_KEY` - Alibaba Cloud DashScope API key
- `KIMI_SEARCH_API_KEY` - Kimi Web Search API key
- `FEISHU_APP_SECRET` - Feishu/Lark app secret
- `OPENCLAW_GATEWAY_TOKEN` - Gateway authentication token

### Step 2: Create openclaw.json

```bash
cp openclaw.example.json openclaw.json
```

Then edit `openclaw.json`:
- Update paths to match your system
- Update `appId` with your Feishu app ID
- The sensitive values already reference environment variables (e.g., `${DASHSCOPE_API_KEY}`)

### Step 3: Verify installation

```bash
openclaw gateway run
```

## Directory Structure

```
~/.openclaw/
├── .env.example          # Environment variable template
├── .gitignore            # Git ignore rules
├── CLAUDE.md             # Project documentation
├── openclaw.example.json # Configuration template
├── extensions/
│   └── feishu-openclaw-plugin/  # Feishu plugin source
└── workspace/            # Agent workspace (separate git repo)
```

## Configuration

Key settings in `openclaw.json`:

| Setting | Description |
|---------|-------------|
| `models.providers` | AI model provider configuration |
| `channels.feishu` | Feishu/Lark channel settings |
| `gateway` | Local gateway server configuration |
| `plugins` | Plugin management |

## Security Notes

- Never commit `.env` to GitHub
- Never commit `openclaw.json` to GitHub (contains actual secrets)
- Use environment variables for all sensitive values
- Rotate API keys regularly

## License

Same as OpenClaw project license.
