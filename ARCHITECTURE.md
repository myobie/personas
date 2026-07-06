# Persona architecture: base + overlays

Each agent's effective persona is **composed from layers**, so the public repo stays generic and every person adds their own private context without forking anything:

**base (public, this repo)** + **specialization overlay(s) (public, this repo)** + **private overlay (the person's private `cos` repo)** → the **effective persona** the agent runs with.

## The three bases

Every agent is one of three:

- **`chief-of-staff`** — the single point of contact; triages, tracks, surfaces. Spawner → `bypassPermissions` + `--permanent`.
- **`supervisor`** — spawns and drives a layer of workers; keeps them progressing. Spawner → `bypassPermissions` + `--permanent`.
- **`worker`** — does the work; doesn't spawn. Leaf → `auto`.

The hierarchy is `chief-of-staff → supervisor → worker`.

## Specialization overlays (public)

A worker (or supervisor) can be **typed** by a public overlay that adds role-specifics on top of the base. An overlay is a persona **fragment**: it *assumes* the base and sharpens it — it never restates the base.

- `overlays/worker/specialist.md` — a worker that owns one repo end-to-end (review/merge/ship).
- `overlays/worker/integrator.md` — a worker that stitches multiple repos/systems together.
- `overlays/worker/lead-developer.md` — a worker with architectural authority over a codebase.
- `overlays/supervisor/manager.md` / `technical-manager.md` — team-lead flavors of supervisor.

(Add more as roles clarify. Specializations are shareable — anyone can use `integrator`.)

## Private overlays (the person's `cos` repo)

The person's private `cos` repo overlays **private context** onto any base or specialization — and never leaves their machine:

- opinions on how a worker/supervisor should behave *in their network*,
- private tooling only they have (described so the agent knows to reach for it),
- anything person-specific that must not be public.

Convention: private overlays live at `cos/overlays/<base-or-specialization>.md` (e.g. `cos/overlays/worker.md`, `cos/overlays/integrator.md`).

## Composition (at launch)

`convoy` composes the layers when it stands an agent up — base → specialization → private overlay → the effective `PERSONA.md`.

**Precedence: private overlay wins > specialization > base** (most-specific-to-the-person beats the generic). Later layers add to and may override earlier ones.

Sketch (convoy owns the exact flags):
`convoy add claude --identity build-1 --as worker:integrator` → `worker.md` + `overlays/worker/integrator.md` + `cos/overlays/{worker,integrator}.md`.

## Shared reference (imported, not composed)

`dev-practices.md` and `known-harness-bugs.md` aren't personas — they're cross-cutting reference every agent gets. Imported alongside the composed persona, not layered into it.

## Consuming this repo

A person's `cos` **depends on this public repo at a pinned SHA** (reproducible) and overlays private context on top — it does not keep its own copy of the personas. That's the dogfood: the public base is shared and versioned; the private layer is theirs.
