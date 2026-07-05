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

## Idle-agent delivery-wake is unreliable
- **Symptom:** a smalltalk message delivered to a healthy, low-context, **idle** agent's inbox sometimes doesn't re-trigger the agent's loop — the message sits unprocessed. A bare Enter doesn't wake it; a pty poke (a short wake pointer + a separate `key:return`) does, after which it reads the inbox and acts normally.
- **Diagnosis:** the file-watch layer (FSEvents/chokidar) can drop the underlying `add` event so the channel notification never surfaces.
- **Recovery/workaround:** run the bus's **polling backstop** if it has one (recommended — it closes the gap and idle agents wake with zero keystrokes). Absent that, poke the loop to wake it — NOT by typing the message content (the message is already delivered; just wake the loop).
- **Status:** fixable in the bus (polling backstop); a launch-time async-rewake hook is useful defense-in-depth.

---

## Spawned agent can inherit the launcher's identity
- **Symptom:** an agent launched from *another* agent's shell can inherit the launcher's identity env — a child booting as if it were the CoS — so its bus tools resolve to the wrong inbox (reads/writes the launcher's, not its own).
- **Fix:** the launcher must write the child's identity (`ST_AGENT`) explicitly into the generated `pty.toml` env and never rely on inheritance. `st launch` does this now — the child's own identity wins over any leaked env.
- **Good behavior to preserve:** an agent unsure of its identity should *pause and ask* rather than act as someone else.
- **Status:** resolved in `st launch` (identity written into `pty.toml`, kill-tested).

---

## GLM/ollama launch path misses `--permission-mode`
- **Symptom:** a GLM-backed agent launched via an ollama-routed path hits Bash approval gates and stalls, because that path controls the harness argv and doesn't pass `--permission-mode auto` through.
- **Recovery/workaround:** approve the gate manually (poke), or pre-seed permission settings in the folder's `.claude` config.
- **Status:** open (launcher gap, ollama path only). Niche — affects GLM-via-ollama launches, not the default path.
