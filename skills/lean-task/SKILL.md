---
name: lean-task
description: Use for routine Git commit/push, presentation-only renames, bounded fixes, small features, documentation edits, and mechanical updates. Check risk first; use a direct anti-overengineering workflow for light tasks, with selective optional skill loading. Leave heavy or uncertain tasks to the normal workflow.
license: MIT
---

# Lean Task

Reduce unnecessary work, not correctness evidence. User scope, repository rules, safety requirements, and higher-priority instructions still apply; this skill grants no permissions.

## Gate

Classify silently from the request and available evidence. Read only enough relevant context to resolve uncertainty, plus any required repository context.

A task is **light** only when:
- The outcome is clear and the cause or implementation path is understood.
- Existing patterns fit; impact is bounded and affected consumers are known.
- A focused check can prove the outcome.
- No security/permissions, money, destructive operation, persisted-data migration, concurrency, public contract change, or new cross-system dependency is involved.

Otherwise it is **heavy**: stop applying this workflow and retain normal investigation, planning, specialized skills, and verification. File count, prompt length, and urgency do not determine risk. Only independently bounded light parts of a mixed request qualify. If new evidence breaks a light condition, exit immediately; do not hide uncertainty behind a small patch.

## Light execution

1. **Locate and decide.** Establish the requested result, relevant implementation, existing pattern, and focused proof. Reuse evidence already gathered. Ask only about material ambiguity that available context cannot resolve.
2. **Act directly.** Once one safe approach meets the request, implement it rather than generating alternatives or repeatedly reconsidering it. Use existing files/functions/dependencies; add an abstraction or dependency only for a current concrete need. Keep unrelated cleanup, speculative features, and symptom suppression outside scope.
3. **Verify and stop.** Exercise the changed behavior and relevant boundaries/consumers. For bugs, prefer failing-before/passing-after evidence and a regression test when useful. Exercise UI on its actual surface when available. Honor required broader gates, but reuse valid checks if nothing relevant changed. Account for relevant docs, report the exercised check or its limit briefly, and stop when the requested outcome is complete.

Use no separate planning phase, delegation, broad audit, or repeated progress narration for straightforward light work unless instructions or actual dependencies require it. Each extra lookup or check should resolve a specific remaining uncertainty. There is no fixed time/token/tool cap; unresolved important behavior still needs work.

## Optional skills

For light tasks, work inline by default. Load an additional optional skill only when its specific guidance is necessary to solve an unresolved part correctly. Topic overlap alone is insufficient. Do not load a chain of generic review, design, planning, or optimization skills for a known mechanical change. Required skills remain required; this rule cannot override a host or repository mandate.

## Routine fast paths

- **Push existing work:** inspect only the Git state needed to confirm the requested commits, branch, destination, and authorization, then perform the normal push and observe its result. Do not revisit implementation, rerun still-valid checks, invent a documentation edit, or create a commit just to push. When a commit is requested, review its scoped diff and keep unrelated files out. Force-push, history rewriting, unknown destinations, secrets concerns, or unresolved changes leave this fast path.
- **Presentation-only rename:** locate the displayed name and its relevant uses; change those strings and check the visible result. Preserve storage keys, identifiers, APIs, and package contracts unless explicitly requested. A display rename is not a redesign or terminology/architecture project; load optional specialist guidance only for a concrete need.
