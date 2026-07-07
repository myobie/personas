# Known harness bugs

Shared registry of bugs/quirks in the agent harnesses (Claude Code, Codex, etc.).
Intended to be inherited by every agent so a new one knows the failure modes without
rediscovering them. Keep it current; add an entry each time you find a reproducible one.

Entry format: symptom → diagnosis → recovery/workaround → status.

---

## Context-saturation wedge (harness freezes; drops messages)
- **Symptom:** at very high context % (seen ~99%), the harness wedges — the input line won't clear (`ctrl+u`/`Esc`/backspace are no-ops), the pane stops repainting (stale frames on `pty peek`), and **incoming smalltalk messages stop being processed** (they land in the inbox but never surface).
- **Diagnosis:** distinguish from a merely *parked* agent (parked = alive, next action drafted-but-unsent, a poke advances it). The tell: send a channel message; if it never surfaces on an idle agent AND the input won't clear, the harness is frozen, not parked.
- **Recovery:** `pty restart <session>` (resumes the pinned session-id). For context saturation, resume **from summary** (headroom) rather than full — full-resume reloads the state that wedged it. Answer post-restart startup gates (trust-folder / channels-dev / resume-choice) — those are legit pty pokes. After restart the agent auto-drains its queued messages with zero keystrokes.
- **Do NOT** hand-deliver the dropped message by typing it into the pty — non-arrival is the bug; recover the harness instead.
- **Status:** open (harness bug). Prevention: don't let agents ride at ~99% context; a supervisor that watches transcript-byte-growth stall + restarts.

---

## Idle-agent delivery-wake — solved by ding
- **Was (non-ding / MCP-only path):** a message delivered to a healthy, low-context, **idle** agent's inbox sometimes didn't re-trigger its loop — the file-watch layer (FSEvents/chokidar) could drop the underlying `add` event so the channel notification never surfaced. A bare Enter didn't wake it; a pty poke did.
- **Solved by ding:** the **ding sidecar is the polling backstop** — it wakes an idle agent on message arrival with zero keystrokes, closing the FSEvents gap. **On a ding-based network this is not a live bug** (ding-mode delivery-wake is proven).
- **Status:** resolved for ding agents. Only the legacy non-ding path had the gap — and the network is moving ding-only, so this retires with it.

---

## Spawned agent inheriting the launcher's identity — impossible with convoy
- **Was:** an agent launched from *another* agent's shell could inherit the launcher's identity env — a child booting as if it were the CoS — so its bus tools resolved to the wrong inbox.
- **Impossible with convoy:** `convoy add` is footgun-proof by design — it writes the child's identity explicitly into the generated config and never relies on env inheritance. That is the whole point; a convoy-added agent *cannot* inherit the launcher's identity.
- **Good behavior to preserve:** an agent unsure of its identity should *pause and ask* rather than act as someone else.
- **Status:** resolved by convoy (footgun-proof add). The live network inherits this at the convoy migration.

---

## GLM/ollama launch path misses `--permission-mode`
- **Symptom:** a GLM-backed agent launched via an ollama-routed path hits Bash approval gates and stalls, because that path controls the harness argv and doesn't pass `--permission-mode auto` through.
- **Recovery/workaround:** approve the gate manually (poke), or pre-seed permission settings in the folder's `.claude` config.
- **Status:** open (launcher gap, ollama path only). Niche — affects GLM-via-ollama launches, not the default path.
