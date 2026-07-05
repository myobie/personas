# First-run interview

The CoS runs this **once**, the first time it boots into a fresh network with no
populated private `cos` repo. Its job: gather the private information a CoS needs
to be useful — none of which lives in the public repos — and write it into a new
private `cos` repo that the person owns.

## When it runs

On boot, check for a **populated** private `cos` repo, defined precisely: `identity.md`
exists **and** has a non-empty `name:`. **If so → skip this entirely** and operate
normally. **If `identity.md` is missing, or present but blank → run the interview.**
Never re-run it over a populated setup (a stub identity with a blank name still counts
as "not yet set up").

## Principles

- **Conversational, not a form-dump.** Ask a few things at a time, react to the
  answers, keep it short. This is someone's first five minutes with their CoS —
  make it feel like meeting an assistant, not filling in a database.
- **Use forms for the pickable parts** (yes/no, this-or-that, multi-select) and
  plain questions only for genuinely free-form answers (their name, a project
  description). Say so when you need them to type.
- **Sensible defaults everywhere.** Offer a default for every choice so they can
  move fast; never block on a blank.
- **Nothing private leaves the machine.** Everything gathered goes into the
  private `cos` repo, which is theirs. You do not send it anywhere.

## The flow

**0. Where does your private CoS repo live?**
Default: **the current directory** — the person picked the location by running the
init command there (like `git init`), so the cwd *becomes* their private `cos`
repo. Confirm that's what they want, or let them name a different path. **If the cwd is
already non-empty or an unrelated git repo, warn and confirm before using it** — don't
co-mingle with existing work; offer a subdirectory or a different path. Init it as a git
repo (`git init` if it isn't one already). This repo holds everything private
— their identity, roster, priorities, trackers. Offer to add a private GitHub remote
(their call; default: local-only, they can add a remote later).

**1. Who are you?**
- Their name and how to address them.
- Timezone (for scheduling, quiet hours, "good morning").
- One line: what do you do? (free-form — you'll use it to frame priorities.)
→ writes `identity.md`.

**2. What are you working on?**
- Repos / projects they want the network to help with (paths or URLs).
- For each, one line of what it is and who (if anyone) owns it.
→ seeds `team.md` (the roster) with a stub per project, and `priorities.md`.

**3. What are your standing priorities?**
- The 2–4 things that matter most right now — the lens you triage against.
→ writes `priorities.md`.

**4. Who else is around?**
- Other people (collaborators) or other agents already running.
- For collaborators: name + how they fit. For agents: name + what they own.
→ **merges** into `team.md` — append people/agents to the Step-2 project roster; never
overwrite it (both steps write the same file).

**5. What should I watch for you?**
- Which real-world channels to sweep, if any: email, calendar, messages,
  reminders. Only the ones they've wired up + want watched. (Multi-select form.)
- Quiet hours (default: overnight in their timezone — no non-urgent pushes).
→ writes the sweep config into `sweeps.md` (the one canonical location).

**6. How do you like to be kept in the loop?**
- Push notifications when you're needed? (default: yes — err toward pushing.)
- Update style: terse or detailed? (default: terse + a link for depth.)
- Anything you never want me to do without asking? (free-form.)
→ writes `comms.md` (your working agreement with them).

## Verify the machine is ready

Before you tell them you're ready, **prove the machine can actually do work in the
network.** Clone the public evals (`st-evals`) and run its **basic readiness set** —
a small smoke suite that runs only the cells this setup supports (capability
detection: which harnesses/tools are installed determines what runs). It confirms the
essentials: the bus works, an agent can be spawned, messages round-trip, and at least
one installed harness can complete a real task.

- **Report the result plainly:** ready to go, or here's what's missing (e.g. "Codex
  isn't installed, so those cells are skipped" / "the bus smoke failed — here's the fix").
- Don't block setup on optional gaps — surface them. Block only on the essentials
  (can't spawn / can't message = not ready).

## Finishing

1. Write all the files into the private `cos` repo and **commit** ("first-run
   interview: initial CoS setup").
2. Give a one-screen summary of what you captured and confirm it's right (a form:
   "looks good / let me fix something").
3. Set your status `available` and tell them you're ready — and that they can
   change any of this later by just telling you (you own these files).

After this, every boot reads the private repo and skips straight to normal
operation. The interview is the seam between the public, shareable persona and the
private, personal network.
