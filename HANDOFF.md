# HANDOFF — read this first (for the human and the AI assistant)

This file briefs a new machine / new Cursor agent on the state of this project
as of 2026-07-17. If you are an AI assistant: read this fully, then help the
owner finish the remaining steps below.

## What this project is

A private WhatsApp data-analyst agent ("Dana") built on OpenClaw
(https://docs.openclaw.ai), answering questions over the owner's personal
BigQuery sandbox by running read-only `bq` queries. See README.md for the
architecture and repo layout.

## Current state — what is DONE

| Piece | Status |
|---|---|
| OpenClaw gateway | Installed globally via npm (v2026.7.1), ran as a macOS launchd service on the OLD machine |
| Gateway auth | Token auth persisted in `~/.openclaw/openclaw.json` (NOT in this repo — regenerate on new machine, see below) |
| WhatsApp channel | Plugin `@openclaw/whatsapp` installed; linked to the owner's personal account in self-chat mode ("Message yourself" chat); DM allowlist = owner's number only; verified working end-to-end with `/status` |
| Model config | `anthropic/claude-opus-4-8` set as primary — but NO Anthropic credentials configured yet (see pending) |
| Agent workspace | This repo's `workspace/` folder (AGENTS.md rules, SOUL.md persona, USER.md, MEMORY.md, skills/) — gateway config points at it |
| BigQuery data | Owner's personal GCP sandbox project (no billing attached), dataset `my_app`, table `user_clicks` (~10k synthetic click events, ~200 fake users, 30 days). Created manually in the BigQuery console |
| Git | This repo, public, at github.com/barakwaisel/baraks-data-agent. Owner's personal GitHub account is `barakwaisel`; on the old machine it authenticated via an SSH key aliased as `github-personal` (the machine's default github.com credentials belong to the owner's WORK account — keep them separate!) |

## What is PENDING (the original plan, steps remaining)

1. **Anthropic auth — BLOCKS EVERYTHING.** Owner must choose:
   - Option A: reuse a Claude Pro/Max subscription (`npm i -g @anthropic-ai/claude-code`,
     `claude auth login`, then `openclaw models auth login --provider anthropic --method cli --set-default`). No per-use cost.
   - Option B: Anthropic API key from console.anthropic.com (pay per token).
2. **Step 3: BigQuery access for the agent.** Create a GCP service account with
   read-only BigQuery roles (`roles/bigquery.dataViewer` on the dataset +
   `roles/bigquery.jobUser` on the project), download a key file (keep OUT of
   git), authenticate `bq` non-interactively
   (`gcloud auth activate-service-account --key-file=...`).
3. **Step 5: first skill.** `workspace/skills/daily-clicks/` — a SKILL.md plus
   `scripts/report.py` that runs a parameterized `bq query --format=json`,
   formats a short summary (yesterday's clicks vs. prior average), prints it.
4. **Step 6: cron.** Register the skill as an OpenClaw cron job (config `cron`
   section) posting the daily summary to the owner's WhatsApp self-chat.
5. Optional: auto-commit of MEMORY.md changes; move gateway to an always-on
   server (owner was considering a small VPS vs. GCP VM).

## Migration checklist for the NEW machine

1. Install: `npm install -g openclaw@latest`, then
   `openclaw plugins install clawhub:@openclaw/whatsapp`.
   Also needed: `google-cloud-sdk` (for `bq`/`gcloud`), git.
2. Clone this repo (suggested: `~/Documents/code/data-agent`).
3. Create `~/.openclaw/openclaw.json` from `config/openclaw.example.json5`:
   - fill in the owner's real WhatsApp number in `allowFrom` and
     `commands.ownerAllowFrom` (format `whatsapp:+<number>`)
   - set `agents.defaults.workspace` to the cloned repo's `workspace/` path
   - set a fresh gateway token:
     `openclaw config set gateway.auth.mode token` and
     `openclaw config set gateway.auth.token $(openssl rand -hex 24)`
   - `chmod 600 ~/.openclaw/openclaw.json`
4. WhatsApp session — EITHER copy `~/.openclaw/credentials/` from the old
   machine (works, it's just files), OR re-link fresh:
   `openclaw channels login --channel whatsapp` and scan the QR
   (phone: Settings > Linked Devices > Link a Device; scan the NEWEST QR
   within ~30s; also remove the old machine's dead device entry from
   Linked Devices). Only ONE machine should run the session — shut down the
   old gateway (`openclaw gateway uninstall` there) before linking.
5. Anthropic auth (pending item 1 above).
6. `openclaw gateway install` to run as a service, then verify with
   `openclaw channels status` and a WhatsApp `/status` message.
7. GitHub push access: either generate a new SSH key and add it to the
   `barakwaisel` GitHub account, or copy `~/.ssh/id_ed25519_personal_github*`
   and the `github-personal` alias block from `~/.ssh/config` on the old
   machine. Remote URL format: `git@github-personal:barakwaisel/baraks-data-agent.git`.

## Hard rules (do not violate)

- This repo is PUBLIC: no tokens, no key files, no phone numbers, no session
  data may ever be committed. Check `.gitignore` before adding files.
- The agent's BigQuery access must stay read-only (SELECT via `bq` CLI).
- Secrets live on the machine (`~/.openclaw/`, key files, env vars), never in git.
