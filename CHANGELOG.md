# Changelog

## 0.2.0 - 2026-05-19

### Features

- **ENV Builder** — interactive UI at `/env-builder` for configuring all Space secrets. Grouped sections: Core, Backup, Telegram, Terminal, Providers, Cloudflare, Advanced. Model picker with provider/model-name presets. Import/export as `HUGGINGMES_ENV_BUNDLE` or plain `.env`.
- **JupyterLab terminal** — full shell access at `/terminal/`. On by default (`DEV_MODE=true`). Uses `GATEWAY_TOKEN` as terminal password — no separate `JUPYTER_TOKEN` needed. Dashboard button added.
- **Chromium browser tools** — installs Chromium and display/font libs so Hermes browser-use tools work out of the box.
- **Plugin persistence** — Hermes plugin directory symlinked into the persistent volume; plugins survive container restarts.
- **Secret redaction** — enabled by default in Hermes config (`security.redact_secrets: true`).
- **Cloudflare Keepalive** — Cloudflare Worker setup for automatic space keep-awake.

### Fixes

- **Space stuck at RUNNING_APP_STARTING** — root cause: `start_jupyter()` called `python3 -c "import jupyterlab"` using system Python; JupyterLab is installed in the Hermes venv. Import failed → `return 1` → `set -euo pipefail` killed `start.sh` → container crashed every boot. Fixed to use `/opt/hermes/.venv/bin/python`.
- **Terminal double password prompt** — proxy now injects `Authorization: token <JUPYTER_TOKEN>` header when forwarding requests to JupyterLab, bypassing its own login screen. One login instead of two.
- **Gemini 404 errors** — strip `google/` or `gemini/` prefix when setting Hermes model name; Hermes gemini provider expects bare model name (e.g. `gemini-2.5-flash`, not `google/gemini-2.5-flash`).
- **Config persistence** — use `setdefault` for user-configurable fields; always overwrite `model.default` and `model.provider` from env so deploy-time settings win without clobbering dashboard changes.
- **Keys disappearing after restart** — sync state to HF Dataset on natural gateway exit (in addition to periodic sync and SIGTERM path).
- **Sync timeouts** — set `HF_HUB_DOWNLOAD_TIMEOUT=300` and enable `HF_XET_HIGH_PERFORMANCE` for faster dataset transfers.
- **hermes not found in terminal** — symlink `hermes` CLI into `$HERMES_HOME/.local/bin`; add `/etc/profile.d/hermes-venv.sh` so PATH includes venv bin in all shell types.
- **Kanban migration crash** — wrap `ALTER TABLE ADD COLUMN` in try/except; idempotent on existing databases.
- **Health endpoint returning 503** — `/health` always returns HTTP 200 (gateway status in JSON body). Returning 503 when gateway was starting caused Docker HEALTHCHECK to fail indefinitely.

### Changes

- Space emoji updated to 🪽 (Hermes winged sandals) across README and dashboard.
- Login page redesigned to match HuggingClaw dark-card aesthetic.
- ENV Builder, Terminal, and Control UI all require session auth (single `GATEWAY_TOKEN` login).
- `HF_HUB_ENABLE_HF_TRANSFER` (deprecated) replaced with `HF_XET_HIGH_PERFORMANCE=1`.
- HEALTHCHECK `start-period` tuned to 60s; health endpoint always returns 200.

## 0.1.0 - 2026-05-03

- Initial HuggingMes Docker Space wrapper for Nous Research Hermes Agent.
- Added HF Space dashboard, `/health`, `/status`, `/v1/*` proxy, and Telegram webhook proxy.
- Added Cloudflare Worker setup for Telegram Bot API base URL proxying.
- Added private HF Dataset backup and restore for Hermes state.
- Added Cloudflare Keepalive Worker setup for automatic space keep-awake.
