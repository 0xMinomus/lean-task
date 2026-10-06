---
name: lean-task
description: Use for bounded code fixes, small features, local UI changes, documentation edits, or mechanical updates. Classify by risk and scope first; apply a lean anti-overengineering workflow only to light tasks. For heavy or uncertain tasks, leave the normal workflow unchanged.
license: MIT
---

# Lean Task

Reduce unnecessary work, not the evidence needed for correctness. Follow the user's scope, repository instructions, and applicable safety requirements; this skill grants no new permissions.

## Gate: light or heavy

Use the request and already available context. Inspect only the relevant implementation, conventions, and consumers needed to resolve uncertainty; required repository reads still apply. Do not audit the whole project just to classify a task.

A task is **light** only when all are supported by evidence:
- The requested outcome and acceptance criteria are clear.
- The cause or implementation path is understood; an existing pattern fits.
- The impact is bounded and affected consumers can be identified.
- A focused check can demonstrate the changed behavior.
- There is no high-risk boundary: security or permissions, money, destructive operations, persisted-data migration, concurrency, a public contract change, or a new cross-system dependency.

Otherwise it is **heavy**. File count, prompt length, and urgency are not classifiers. A known local bug can be light; a one-line permission change is heavy. A mechanical update across several files can be light when every affected use is known and meaning stays unchanged.

For heavy tasks, stop applying this skill and use the normal task-appropriate workflow. Keep required planning, investigation, specialized skills, and verification intact. For mixed requests, do not label the whole request light because one part is easy; only independently bounded light parts qualify.

Classify silently unless the user asks. Do not ask them to choose a difficulty or add an activation announcement. If later evidence breaks any light condition, leave the lean workflow immediately, retain useful work, and investigate normally. Do not patch around uncertainty to stay light.

## Light workflow

1. **Bound the result.** Identify the requested behavior and what proves it. Ask only about a material ambiguity that the request or repository cannot resolve.
2. **Read enough.** Locate the relevant code and existing pattern. Trace affected consumers before changing shared behavior. Prefer targeted reads and reuse evidence already gathered; repeat a check only when new information warrants it.
3. **Make the direct change.** Prefer existing functions, files, and dependencies. Add an abstraction only for a current requirement or demonstrated duplication; use a dependency only for a concrete capability the existing stack cannot reasonably provide. Keep unrelated refactors, speculative options, and extra features outside scope. A symptom-suppressing patch is not a simpler fix.
4. **Verify proportionately.** Run the focused test or scenario covering the changed behavior, relevant boundaries, and affected consumers. For a bug, use a failing-before/passing-after check when practical; add a regression test when it protects a plausible recurring failure. Exercise UI changes on the actual surface when available. Run broader gates when required by the repository or actual impact, not as a substitute for a focused check. Report any verification limit.
5. **Stop at completion.** The requested outcome works, affected consumers and relevant docs are accounted for, and verification evidence exists. Report the change and exercised check briefly. An unresolved failure or untested important behavior is not completion.

Use a short plan only when sequencing helps or instructions require one. Do straightforward bounded work inline; delegate only genuinely independent substantial work. Batch independent lookups when useful. Keep tool calls purposeful and explanations proportional to the request. Do not impose a fixed token, tool, file, or time cap that could cut off correct work.
