# Known harness bugs

Shared registry of bugs/quirks in the agent harnesses (Claude Code, Codex, etc.).
Intended to be inherited by every agent so a new one knows the failure modes without
rediscovering them. Keep it current; add an entry each time you find a reproducible one.

Entry format: symptom → diagnosis → recovery/workaround → status.

---

## Context-saturation wedge (harness freezes; drops messages)
- **Symptom:** at very high context % (seen ~99%), the harness wedges — the input line won't clear (`ctrl+u`/`Esc`/backspace are no-ops), the pane stops repainting (stale frames on `pty peek`), and **incoming st2 messages stop being processed** (they land in the inbox but never surface).
- **Diagnosis:** distinguish from a merely *parked* agent (parked = alive, next action drafted-but-unsent, a poke advances it). The tell: send a channel message; if it never surfaces on an idle agent AND the input won't clear, the harness is frozen, not parked.
- **Recovery:** the machine root runs `pty restart <session>` (resumes the pinned
  session-id). For context saturation, resume **from summary** (headroom) rather
  than full — full-resume reloads the state that wedged it. Root handles
  post-restart runtime gates, then verifies st2 status, inbox processing, and live
  PTY state.
- **Do NOT** hand-deliver the dropped message by typing it into the pty — non-arrival is the bug; recover the harness instead.
- **Status:** open (harness bug). Prevention: don't let agents ride at ~99%
  context; the machine root watches runtime saturation and owns recovery.

---

## Idle-agent delivery-wake — arrival solved; turn wake remains open
- **Was (non-ding / MCP-only path):** a message delivered to a healthy,
  low-context, **idle** agent's inbox sometimes didn't re-trigger its loop — the
  file-watch layer (FSEvents/chokidar) could drop the underlying `add` event so
  the channel notification never surfaced. A bare Enter didn't wake it; a pty
  poke did.
- **Solved by DING:** the **DING sidecar supplies message-arrival events**,
  closing the FSEvents arrival gap. Agents match the stable `[DING]` prefix and
  `[id:<rand6>]`, not the complete sentence, then drain on cold boot/new ids.
- **Still open:** DING arrival does not guarantee Codex turn submission or wake.
  In a live 2026-07-26 case, the sidecar observed arrival and staged the notice,
  but Codex displayed the idle "Create a plan?" choice. The modal-safe gate
  correctly withheld Return, so the notice remained staged indefinitely and the
  agent did not start a turn.
- **Recovery/workaround:** use bounded pty diagnosis/recovery when a staged DING
  does not wake the turn. Low-frequency inbox polling by the Remote CoS is a
  temporary, principal-authorized exception pending an evented st2 fix; agents
  should not add routine inbox polling. Do not weaken the active-human/modal
  safety gate to force submission.
- **Status:** open for guaranteed turn submission/wake; resolved only for
  file-watch arrival.

---

## Spawned agent inheriting the launcher's identity — prevented by st2
- **Was:** an agent launched from *another* agent's shell could inherit the launcher's identity env — a child booting as if it were the CoS — so its bus tools resolved to the wrong inbox.
- **Prevented by st2:** the catalog declares the child's identity and the compiled
  runtime configuration sets it explicitly instead of relying on the launcher's
  environment. The machine root reconciles that declaration and verifies the live
  identity before calling the agent healthy.
- **Good behavior to preserve:** an agent unsure of its identity should *pause and ask* rather than act as someone else.
- **Status:** resolved on the st2/catalog launch path.

---

## GLM/ollama launch path misses `--permission-mode`
- **Symptom:** a GLM-backed agent launched via an ollama-routed path hits Bash approval gates and stalls, because that path controls the harness argv and doesn't pass `--permission-mode auto` through.
- **Recovery/workaround:** approve the gate manually (poke), or pre-seed permission settings in the folder's `.claude` config.
- **Status:** open (launcher gap, ollama path only). Niche — affects GLM-via-ollama launches, not the default path.

---

## "Terminal active" is meaningless in a pty — never infer the principal is present from it
- **Symptom:** the PushNotification / desktop-notify tool reports the terminal as ACTIVE ("this terminal is active, so your output already reaches the user; a notification would be redundant — not sent"), and an agent reads that as "the principal is here right now" — then addresses them as present, or suppresses a push they actually needed. But the principal is NOT necessarily there.
- **Diagnosis:** every agent runs inside a **pty**. The pty's terminal is always live (it is the agent's own session), so any "is the user at the terminal?" presence check is fooled 100% of the time — it returns "active" whether or not a human is watching. The signal describes the pty, not the principal's presence.
- **Recovery/workaround:** never infer the principal's presence from a terminal-active / "not-sent-because-active" signal. Decide whether to push or surface on whether you NEED them, not on a presence guess (err toward pushing when you need them). A push's "not sent / suppressed" response is unreliable — fire when you need them, and don't re-fire assuming it failed. Scheduler-triggered output is a status note, not a live conversation — do NOT greet or address the principal as if they just arrived; wait for an actual human turn.
- **Status:** inherent to running in a pty — not fixable; a standing rule for every agent.
