# Persona architecture: base + overlays

Each agent's effective persona is **composed from layers**, so the public repo stays generic and every person adds their own private context without forking anything:

**base (public, this repo)** + **specialization overlay(s) (public, this repo)** + **private overlay (the person's private `cos` repo)** → the **effective persona** the agent runs with.

## The substrate — three tools, three distinct powers

These personas run on a stack whose power comes from three layers that compose:

- **st2 — declared topology, reconciliation, and shared state.** The catalog
  declares agents and their relationships; host services reconcile that desired
  state. The bus, statuses, contexts, and resources make coordination inspectable.
  Lifecycle decisions are st2-native rather than coupled to one harness.
- **pty — harness-neutral runtime control.** Each agent runs in a PTY that can be
  inspected and recovered without pretending every harness has the same prompts
  or rendering behavior. The machine root owns routine PTY diagnosis and recovery.
- **fabric — reachability between machines.** Roots verify the paths their local
  agents need and distinguish a host-local failure from a fabric failure. The
  public contract assumes no particular hostname, provider, or network layout.

The combination is the point: **declared and inspectable state + recoverable
runtimes + explicit cross-machine reachability.**

## The five bases

Every agent is one of five:

- **`chief-of-staff`** — exactly one per human; owns that human's priorities,
  decisions, roadmap, and cross-machine coordination.
- **`root`** — exactly one per machine; owns st2/service reconciliation, required
  fabric reachability, PTY recovery, and the local runtime roster. It never owns
  general code work.
- **`supervisor`** — leads a workstream/team and drives its work; orchestrates
  actors but does not own their repos.
- **`worker`** — does the work (one task, or owns one repo end-to-end); doesn't
  spawn.
- **`technical-manager`** — owns one repo **hands-on AND** supervises a team whose
  repos build alongside it.

**Permission posture (interim): every agent runs `bypassPermissions` for now.**
Roots and work orchestrators need the configured host authority for their
operational or spawning responsibilities. Workers still do not orchestrate, even
though they run bypass while per-agent `auto` gating awaits sandbox isolation.

## Two orthogonal topologies

There is no single chain that can express both human authority and machine
operations. Keep the planes separate:

```text
work direction                         runtime health

human                                 machine
  └─ exactly one CoS                    └─ exactly one root
       ├─ repo owner                         ├─ local CoS runtime
       └─ supervisor / tech manager          ├─ local supervisor runtime
            └─ worker                        └─ local owner/worker runtime
```

A CoS may brief a repo owner directly; work does not need to pass through root.
A root may operate agents related to several CoSs; incidents do not route to a
"machine owner." Root derives every recipient from affected agents' declared CoS
relationships and sends shared incidents to their union.

| Concern | CoS | Root | Repo owner / work lead | st2/catalog |
|---|---|---|---|---|
| Human priorities, decisions, roadmap | owns | reports facts only | executes or advises | records declared relationships |
| Cross-machine work routing | owns | reports placement/health | receives or cascades work | records placement |
| Host st2 services and reconciliation | requests outcomes | owns | reports symptoms | supplies desired state |
| Required fabric reachability | prioritizes impact | diagnoses/restores | reports symptoms | records required relationships |
| PTY crash, wedge, or restart | receives escalation | owns, except `dnd` | resumes work | identifies declared runtime |
| Repository change | routes to owner | never patches | owns within boundary | records ownership |
| Root cardinality | expects one per machine | detects/reports drift | no role | enforces |

This split preserves one dedicated owner per repo. Operational access to an
agent's PTY never grants root write authority over that agent's code or product
direction.

### Event delivery is not scheduling

DING is a message-arrival event. Match the stable `[DING]` prefix and
`[id:<rand6>]`, never the complete human-readable sentence; wording may differ
across a rolling binary window. Agents drain their st2 inbox on cold boot and for
each new id, archiving every message as soon as they act. They deduplicate by id
and do not periodically poll the inbox.

Scheduled sweeps inspect live state and trackers, not message arrival. A root's
persistent health wake is supplied by the host st2 reconciler. A CoS's periodic
work/tracker review requires a real scheduler supplied by its network/runtime.
Neither role may claim a schedule is armed merely because its persona describes
one.

## Specialization overlays (public)

A base can be **typed** by a public overlay that adds role-specifics on top. An
overlay is a persona **fragment**: it *assumes* the base and sharpens it — it never
restates the base. Public overlays, when present, follow
`overlays/<base>/<specialization>.md`; for example, `worker:integrator` could type
a worker that stitches systems together. Overlays are optional; the five bases in
this repo stand on their own.

## Private overlays (the person's `cos` repo)

The person's private `cos` repo overlays **private context** onto any base or specialization — and never leaves their machine:

- opinions on how a worker/supervisor should behave *in their network*,
- private tooling only they have (described so the agent knows to reach for it),
- anything person-specific that must not be public.

Convention: private overlays live at `cos/overlays/<base-or-specialization>.md` (e.g. `cos/overlays/worker.md`, `cos/overlays/integrator.md`). **They are committed to the private `cos` repo only — never to this public personas repo. Nothing person-specific belongs here.**

## Composition (at launch)

The st2 catalog declares the role and relationships; the configured launch path
composes base → specialization → private overlay → the effective `PERSONA.md`.

**Precedence: private overlay wins > specialization > base** (most-specific-to-the-person beats the generic). Later layers add to and may override earlier ones.

Sketch: a catalog entry typed as `<base>:<specialization>` resolves to the base,
the matching public overlay when present, and applicable private overlays.
Host-specific launch flags remain outside this public contract.

### Native catalog source

The canonical source for an agent is its hand-authored native `agent.kdl`. Do not
use the legacy `st2 add` + `st2 compile` IR pipeline. `st2 compile-agent` remains
experimental: when deliberately used, its generated KDL and all rendered
persona/bus targets must be inspected before materialization or activation.

## Shared reference (imported, not composed)

`dev-practices.md` and `known-harness-bugs.md` aren't personas — they're cross-cutting reference every agent gets. Imported alongside the composed persona, not layered into it.

## Consuming this repo

A person's `cos` **depends on this public repo at a pinned SHA** (reproducible) and overlays private context on top — it does not keep its own copy of the personas. That's the dogfood: the public base is shared and versioned; the private layer is theirs.

## Compatibility and migration

`root` is an additive base, not a renamed supervisor. A pinned network changes
only when it bumps the personas SHA.

For an existing one-machine network:

1. Declare or designate exactly one root for the machine in st2/catalog.
2. Move host service, fabric, PTY recovery, local roster, and runtime shepherd
   duties from the CoS/supervisor to root.
3. Keep private CoS trackers, human priorities, repo ownership, and existing work
   assignments unchanged.
4. Record each agent's relevant CoS relationships so root can route incidents
   correctly, including on machines shared by several humans.

st2/catalog owns enforcement of one root per machine. The root independently
checks declared and live state on boot/sweep and reports missing or duplicate
roots; it does not silently self-elect or remove another root.
