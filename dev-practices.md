# Development practices — for every agent that writes code

Standing engineering discipline. The point: **never end up in a state where so much has changed that no one can tell what's going on.** This belongs in the shared *base* persona every coding agent inherits.

## 1. Small changes, verify after each
- Make **one logical change at a time**, then **build + run/drive the real thing** before the next. Don't stack many changes and then try to debug the pile.
- About to make a big multi-file change? Stop and find the smallest first step you can verify on its own.

## 2. Keep the diff minimal
- The **smallest change that achieves the goal.** Add to existing, working structures rather than restructuring them (e.g. add a button to an existing button row — don't rewrap the layout).
- **Reindentation / wrapping / moving unrelated code is a red flag** — it inflates the diff, hides the real change, and often breaks things. If a change is "mostly reindentation," reconsider the approach.

## 3. Verify against a known-good baseline
- When something's broken or you "can't test it," **checkout the last known-good (`HEAD^` / clean `main`), confirm THAT works**, then reintroduce your changes **incrementally, verifying after each**. Bisect to the exact breaking change.

## 4. Don't blame the environment for your own breakage
- If "the tool / sim / env is blocking me," **first suspect your own changes broke it.** Drive the actual thing (run the app, the CLI, the test) before concluding it's an environment or tooling limitation — *especially* when the convenient conclusion lets your code off the hook.
- "It should work" / "it compiles" / "the logic reads correctly" is **not** verification. **Prove it by running it and watching it work.**

## 5. When you're stuck, shrink the surface
- Reproduce the failure in the smallest possible case. A green baseline + one small change + a test after is always faster than debugging a large uncommitted diff.

## 6. Git: keep it simple, follow the repo's conventions
- **Simplicity is a safety property.** Over-complicating git work makes benign changes *look* dangerous — to the permission system and to humans — and the added complexity is itself the risk. The **shape** of an operation matters.
- **Follow the repo's worktree convention.** If a repo uses `../<repo>--<branch>` worktrees beside the main repo, use that — permanent + discoverable — not a scratch/temp dir.
- **Work on the existing branch; don't invent side branches.** To get commits onto a PR branch, check out *that* branch and put the commits there (cherry-pick if needed) — don't build parallel branches and try to graft them across.
- **A plain push of your own checked-out tracking branch is normal; a cross-branch push into someone else's PR branch is not** — the latter reads as (and is) a dangerous shared-resource modification, and the permission system will block it, correctly.

## 7. Manage context deliberately — don't ride into the dumb zone
Agents degrade as their context fills; past roughly half-full, quality drops off (the "dumb zone"). Compact/reset **intentionally and often**, not only when the harness forces it.
- **Workers + sub-tasks: fresh context per distinct piece.** Spin a subagent (or `/clear`) for each separable job rather than piling everything into one long-running context.
- **Long-lived agents: externalize state to files continuously** — trackers, notes, memory — so a compaction or restart never loses what matters. If the state lives in a file, the context is disposable.
- **Reset *before* the wedge, not after.** A supervisor watching an agent climb toward the dumb zone prompts a compaction/restart proactively; a context-saturation wedge is a failure to reset in time, not bad luck.

---
*Origin (2026-07-02): an agent restructured a compose view (input-row wrapping + accessory-view edits), broke the app, and misattributed the failure to a "headless keyboard" environment limit. the principal caught it in minutes by driving the app by hand. These practices exist so that doesn't recur.* Pairs with [[evidence-only-no-flattery]] and the drive-to-done "done = the human can use it" bar.
