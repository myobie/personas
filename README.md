# personas

Generic **role contracts** for a Chief-of-Staff (CoS) + [smalltalk](https://github.com/compoundingtech/smalltalk)
agent network. These are the roles each agent plays — the mission, boundaries, and
hard-won operating rules — with **zero personal or network-specific data**. A real
network consumes this repo read-only, pinned to a SHA; everything private lives
elsewhere (see below).

## The split: public contract vs private state

- **This repo (public):** the role contracts. Reusable by anyone. No identities, no
  repos, no priorities, no PII.
- **A private `cos` repo (per person):** the actual network state — who the principal
  is, their roster, priorities, and the CoS's trackers (`team.md`, `priorities.md`,
  `SITREP.md`, `IN-FLIGHT.md`). Never shared.

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

**Four base roles** — every agent is one of these:
- **`chief-of-staff.md`** — **start here.** The single point of contact: triage, track, surface. Spawner (`bypassPermissions`).
- **`supervisor.md`** — the middle tier: spawns + drives workers, keeps them progressing (`CoS → supervisor → worker`); orchestrates *actors*, doesn't touch code. Spawner (`bypassPermissions`).
- **`worker.md`** — the leaf: does one job (or owns one repo end-to-end), doesn't spawn. Runs `bypassPermissions` for now (`auto`+sandboxes is future work).
- **`technical-manager.md`** — a hybrid lead: owns a repo **hands-on AND** supervises a team (does work *and* orchestrates). Spawner (`bypassPermissions`).

**Support + reference:**
- **`first-run-interview.md`** — the one-time interview the CoS runs on a fresh network to set up the private repo.
- **`remote-control.md`** — the principal driving an agent's pty directly (parachuting).
- **`dev-practices.md`** — cross-cutting engineering norms.
- **`known-harness-bugs.md`** — real Claude Code / Codex quirks + workarounds.
- **`ARCHITECTURE.md`** — how the base roles + overlays compose.

**Overlays.** A base can be sharpened by overlay fragments: **public specializations** (e.g. an integrator, a lead-developer) layered on a base, and **private per-person context layered from your own `cos` repo** (your opinions on how a role should behave, your private tooling). This public repo holds only generic contracts — **anything personal stays in your private `cos` repo and never lands here.** See `ARCHITECTURE.md`.

## The bigger picture

This repo is one piece of a network that also needs [smalltalk](https://github.com/compoundingtech/smalltalk)
(the file-based message bus + `st` CLI), [pty](https://github.com/myobie/pty)
(the lean session primitive each agent runs in), and `convoy` (launch, orchestration,
and hosting — `convoy cos` bootstraps a CoS; `convoy add` stands up an agent). Start
from smalltalk's onboarding; point your CoS at these personas; run the first-run
interview; go.
