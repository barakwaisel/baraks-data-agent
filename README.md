# data-agent

A private WhatsApp data-analyst agent ("Dana") built on [OpenClaw](https://docs.openclaw.ai),
answering questions over a personal BigQuery sandbox.

## Architecture

- **OpenClaw gateway** runs as a launchd service on the Mac and bridges WhatsApp <-> LLM.
- **WhatsApp** is linked as a device on the owner's personal account (self-chat mode,
  allowlisted to one number).
- **Model**: Anthropic Claude (configured in `~/.openclaw/openclaw.json`).
- **Data**: BigQuery sandbox, dataset `my_app`, queried read-only via the `bq` CLI.

## Repo layout

```
workspace/          The agent's brain — OpenClaw agent workspace
  AGENTS.md         Rules: data access, bq usage, answer style, boundaries
  SOUL.md           Persona
  USER.md           Who the owner/admins are
  MEMORY.md         Long-lived agent notes
  skills/           One folder per recurring report (SKILL.md + scripts/)
config/
  openclaw.example.json5   Sanitized gateway config template
```

## What is NOT in this repo (on purpose)

- `~/.openclaw/openclaw.json` — contains the gateway auth token
- `~/.openclaw/credentials/` — the linked WhatsApp session keys
- Any GCP service-account keys / API keys

## Setup on a new machine (short version)

1. `npm install -g openclaw@latest`
2. Copy `config/openclaw.example.json5` to `~/.openclaw/openclaw.json`, set a fresh
   `gateway.auth` token
3. `openclaw plugins install clawhub:@openclaw/whatsapp`, then
   `openclaw channels login --channel whatsapp` (QR scan)
4. Configure Anthropic auth (`openclaw models auth ...`)
5. Install the gcloud SDK, authenticate `bq` against the sandbox project
6. `openclaw gateway install`
