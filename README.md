# personas

Generic **role contracts** for a Chief-of-Staff (CoS) + [smalltalk](https://github.com/myobie/smalltalk)
agent network. These are the roles each agent plays — the mission, boundaries, and
hard-won operating rules — with **zero personal or network-specific data**. A real
network consumes this repo read-only, pinned to a SHA; everything private lives
elsewhere (see below).

## The split: public contract vs private state

- **This repo (public):** the role contracts. Reusable by anyone. No identities, no
  repos, no priorities, no PII.
- **A private `cos` repo (per person):** the actual network state — who the principal
  is, their roster, priorities, and the CoS's trackers (`team.md`, `priorities.md`,
  `WAITING-ON-YOU.md`, `IN-FLIGHT.md`). Never shared.

The private data isn't *scrubbed out* of the public repo — it never goes in. The CoS
gathers it per-person via the **first-run interview** (`first-run-interview.md`) the
first time it boots into a fresh network, and writes it into the private repo. That's
the seam between the shareable persona and the personal network.

## How it's consumed

A `cos` repo pins this repo at a specific **SHA** (e.g. a git submodule at
`cos/personas/`), so every network is on a known, reproducible version of the
contracts and bumps when it chooses. The CoS reads its role from here; it writes only
to its own private repo.

## The files

- **`chief-of-staff.md`** — **start here.** The CoS role: mission, the public/private
  split, boundaries, brief shape, driving work to done, "ask for no's not yes's,"
  escalation, freeze recovery, and the first-run hook.
- **`first-run-interview.md`** — the one-time interview the CoS runs on a fresh network
  to set up the private repo (where it lives, who you are, your repos, priorities,
  team, channels, comms style).
- **`manager.md` / `technical-manager.md`** — team-lead roles (a manager owns a team; a
  technical manager also writes code).
- **`specialist.md`** — a single-repo implementer.
- **`supervisor.draft.md`** — a watchdog/verifier role (draft).
- **`standalone.md`** — an agent that owns its own repo end-to-end.
- **`dev-practices.md`** — cross-cutting engineering norms.
- **`known-harness-bugs.md`** — real Claude Code / Codex quirks to expect + how to
  work around them.

## The bigger picture

This repo is one piece of a network that also needs [smalltalk](https://github.com/myobie/smalltalk)
(the file-based message bus + `st` CLI) and [pty](https://github.com/myobie/pty)
(the session manager that runs each agent). Start from smalltalk's onboarding; point
your CoS at these personas; run the first-run interview; go.
