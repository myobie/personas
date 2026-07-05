# Driving your network remotely (Claude Code Remote Control)

Your agents run as local Claude Code sessions under pty. **Remote Control** lets you
drive any of them — most importantly your CoS — from your phone or a browser, while the
session keeps running on your machine (your files, MCP servers, and tools stay local;
the remote view is just a window in). This is how you brief and steer the network when
you're away from the keyboard.

> Requires Claude Code **v2.1.51+**. Facts here are from the official docs
> (code.claude.com/docs/en/remote-control) as of mid-2026; verify against your version.

## Prerequisites (all must hold, or it silently stays off)

- **Plan:** Pro, Max, Team, or Enterprise. (API-key auth is *not* supported.)
- **Login:** signed in via `claude.ai` OAuth — run `claude` then `/login`. Not a raw
  `CLAUDE_CODE_OAUTH_TOKEN`.
- **Direct endpoint:** talking straight to `api.anthropic.com` — no `ANTHROPIC_BASE_URL`
  override, gateway, Bedrock, or similar.
- **Workspace trust:** run `claude` in the project dir once and accept the trust dialog.
- **Team/Enterprise only:** an owner must enable the Remote Control toggle in the admin
  console.

If it won't turn on, run `/status` — it reports the specific reason (see Troubleshooting).

## Turning it on

- **In a running session:** `/remote-control [name]` — activates it and carries the
  conversation over. (In VS Code: `/remote-control` or `/rc`.)
- **Start a session with it on:** `claude --remote-control [name]` — usable locally *and*
  remotely at once.
- **Server mode:** `claude remote-control` — a process that accepts connections; press
  space to show a QR code.
- **Always-on:** `/config` → enable "Enable Remote Control for all sessions" so every
  interactive session registers one automatically.

## Using it from your phone / browser

Connect via the URL printed at startup, a scanned QR code, or the session list at
**claude.ai/code** or the Claude mobile app. From there you can send messages, reference
files, **approve gate prompts**, and watch output stream in real time — and hop between
desktop CLI, phone, and browser in the same session.

A few commands are **local-only** (interactive pickers like `/resume`/`/plugin`, and
`/remote-control` itself); most others (`/compact`, `/clear`, `/context`, `/usage`, …)
work remotely.

## ⚠️ Restart behavior — the gotcha for long-lived agents

**Remote Control does NOT survive a process restart.** When an agent's `claude` process
is restarted (which the CoS does routinely — context-wedge recovery, headroom resets),
the remote connection drops and **must be re-enabled by hand** in the fresh session:

- Re-run `/remote-control` in the new session, **or**
- Resume + reconnect with `claude remote-control --continue` (most recent session in the
  dir) or `--session-id <id>` (v2.1.200+).

This is why **re-running `/remote-control` is a human step after restarting your CoS** —
an agent can't re-enable it for you (it involves your account/device). Plan for it: after
you restart the CoS, reconnect it from your phone before walking away.

## Security model (brief)

Your local process is the origin of every request — Remote Control **opens no inbound
ports**; the session polls Anthropic over TLS for work. Credentials are short-lived and
purpose-scoped. Team/Enterprise can additionally require enrolled **Trusted Devices**
(recent auth + device binding) to connect.

## Troubleshooting (via `/status`)

| Symptom | Cause | Fix |
|---|---|---|
| "requires a claude.ai subscription" | API-key auth | `claude auth login` → choose claude.ai |
| "requires a full-scope login token" | `CLAUDE_CODE_OAUTH_TOKEN` in use | `claude auth login` for a session token |
| "not yet enabled for your account" | rollout gate / stale entitlements | `claude auth logout` → `login` |
| "only available … via api.anthropic.com" | `ANTHROPIC_BASE_URL` / Bedrock / gateway | unset it; use the direct API |
| "disabled by your organization's policy" | Team/Enterprise toggle off | owner enables in admin console |
