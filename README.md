# personas

Generic **role contracts** for a human-aligned Chief-of-Staff (CoS), a
machine-local root, and an st2 agent network. These are the roles each agent plays
— the mission, boundaries, and hard-won operating rules — with **zero personal or
network-specific data**. A real network consumes this repo read-only, pinned to a
SHA; everything private lives elsewhere (see below).

## The split: public contract vs private state

- **This repo (public):** the role contracts. Reusable by anyone. No identities, no
  repos, no priorities, no PII.
- **A private `cos` repo (per person):** the human-facing state — who the
  principal is, their roster, priorities, and the CoS's trackers (`team.md`,
  `priorities.md`, `SITREP.md`, `IN-FLIGHT.md`). Never shared.
- **st2/catalog + runtime state:** declared agent, machine, root, and CoS
  relationships. A root reads this state to operate its machine; machine-specific
  data does not belong in this public repo.

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

**Five base roles** — every agent is one of these:
- **`chief-of-staff.md`** — **start here for human work.** Exactly one per human: priorities, decisions, roadmap, and cross-machine work routing.
- **`root.md`** — **start here for machine health.** Exactly one per machine: st2 reconciliation, services, fabric reachability, PTYs, and local runtime shepherding; never general code work.
- **`supervisor.md`** — leads a workstream/team and keeps its work progressing; distinct from a root's machine-local runtime supervision. Spawner (`bypassPermissions`).
- **`worker.md`** — the leaf: does one job (or owns one repo end-to-end), doesn't spawn. Runs `bypassPermissions` for now (`auto`+sandboxes is future work).
- **`technical-manager.md`** — a hybrid lead: owns a repo **hands-on AND** supervises a team (does work *and* orchestrates). Spawner (`bypassPermissions`).

The two cardinality rules are independent: **one CoS per human** and **one root
per machine**. One root can operate agents for several humans. Work may route
directly from a CoS to a repo owner; runtime health always routes through the
agent's machine root. See `ARCHITECTURE.md`.

**Support + reference:**
- **`first-run-interview.md`** — the one-time interview the CoS runs on a fresh network to set up the private repo.
- **`dev-practices.md`** — cross-cutting engineering norms.
- **`known-harness-bugs.md`** — real Claude Code / Codex quirks + workarounds.
- **`ARCHITECTURE.md`** — how the base roles + overlays compose.

**Overlays.** A base can be sharpened by overlay fragments: **public
specializations**, when present, layered on a base, and **private per-person
context layered from your own `cos` repo** (your opinions on how a role should
behave, your private tooling). This public repo holds only generic contracts —
**anything personal stays in your private `cos` repo and never lands here.** See
`ARCHITECTURE.md`.

## The bigger picture

This repo is one piece of a network that also needs st2 (catalog, reconciliation,
message bus, and shared agent state), [pty](https://github.com/compoundingtech/pty)
(the harness-neutral session primitive), and the configured fabric between
machines. Declare the human/CoS, machine/root, agent/repo-owner, and agent/CoS
relationships in st2; point each role at these personas; then run the first-run
interview for each human.
