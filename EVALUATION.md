# Evaluation

This skill is a model instruction, not a deterministic classifier. A passing
sample is evidence for that sample, not a universal correctness or savings claim.

## Gate scenarios

The reproducible task evidence and expected classifications are in
[evaluation/cases.json](evaluation/cases.json). These are synthetic scenarios,
not claims about a production repository.

| Case | Expected gate | Reason |
|---|---|---|
| README typo | Light | Bounded, behavior-neutral edit with known verification |
| Private pagination off-by-one | Light | Known cause, mapped consumers, boundary checks |
| Local dialog label | Light | Existing translation pattern and surface checks |
| Mechanical update at 12 known uses | Light | Same meaning and all consumers identified |
| Local clear button | Light | Existing component pattern and specified behavior |
| One-line permission removal | Heavy | Authorization boundary |
| Saved-field migration | Heavy | Persisted user data |
| Intermittent reconnect freeze | Heavy | Unknown cause and impact |
| Exported return/error behavior change | Heavy | Public contract and unknown consumers |
| Payment rounding change | Heavy | Money boundary |
| Typo plus authentication replacement | Heavy overall | Easy subtask does not erase security risk |
| Null guard with newly discovered error contract | Heavy after discovery | Exit the lean workflow before incompatible edits |

On 2026-10-06, 12 independent stateless model calls received the complete skill
and one scenario each. Each returned a classification, lean-active flag, reason,
and next action. All 12 matched the expected gate and activation flag; the final
case explicitly exited the lean workflow. The session's default completion model
was used; the completion interface did not expose its resolved model ID or usage,
so no named-model accuracy or token measurement is claimed for this check.

This evaluates decisions on supplied evidence. It does not exercise repository
exploration, tool use, automatic discovery, or a real multistep escalation.

The original 12-scenario table remains historical. The expanded fixture contains
23 scenarios, including code, document, Git, installation, and escalation cases.

## Repeat the gate check

1. Give the same model one case at a time, with `SKILL.md` as its instructions.
2. Request `classification` (`light` or `heavy`), `lean_active`, a reason, and the
   next action. Keep that response format outside the installed skill.
3. Compare with the fixture's expected class and activation (`light` means active).
4. Inspect actions as well as labels. A heavy label followed by curtailed security
   verification still fails. A light label followed by an unrelated rewrite fails.
5. Repeat ambiguous cases and test escalation with information disclosed later.

## Compare actual task execution

Use equivalent clean fixtures, the same model, tools, repository rules, and
acceptance checks. Run once without this skill and once with it available through
normal discovery; do not name the skill in the task prompt. Repeat across task
categories and vary run order to avoid interpreting a single sample as a trend.

Capture:
- Correctness against the same acceptance and boundary checks.
- Unrequested changes, abstractions, or dependencies.
- Evidence that the skill was discovered and read when appropriate.
- Exit from the lean workflow when new evidence reveals a heavy boundary.
- Tool calls, input/output tokens, cached tokens, and wall-clock duration when
  the client reports them. Include skill-loading overhead; separate cold/warm runs.

A faster run fails if it skips required evidence or changes behavior outside the
request. A heavy task must retain its normal investigation and quality gates.
Usage cannot be compared fairly if the model, task, tools, or context differ.

## Scope of compatibility evidence

Skills CLI 1.7.0 discovered exactly one skill, `lean-task`, from both the local
repository and the published GitHub repository. After a native global copy on
Windows, `omp read skill://lean-task` resolved the user-global
`.omp/agent/skills/lean-task/SKILL.md`. This checks packaging and installation,
not behavior in every client; other clients have not been behavior-tested here.
The installed skill has only a Markdown instruction file; there are no hooks,
executables, or runtime dependencies to run on a user's machine.

## Actual omp smoke runs

On 2026-10-06, omp 18.6.1 ran fresh disposable fixtures using
`openai-codex/gpt-6.1-sol`. Only `read`, `edit`, and `bash` were selected for the
pagination and contract tasks; the migration-planning run had only `read`.
Extensions and rules were disabled, and sessions were not persisted by the CLI.
The skill was placed in the fixtures' native `.omp/skills/lean-task/` directory.
Normal task prompts did not name the skill.

### Light task: private pagination boundary

Baseline and skill fixtures started with the same private helper:

```js
export function paginate(items, page, pageSize) {
  const start = page * pageSize;
  return items.slice(start, start + pageSize - 1);
}
```

Both runs changed only the end expression to `start + pageSize` and passed the
same checks: `[1,2,3]` on the first page, `[4,5,6]` on the middle page, `[7]` on
the partial page, and `[]` for past-end and empty input. Each reproduced a failure
before its edit and observed passing execution afterward. The skill-enabled run
automatically read `skill://lean-task`; neither introduced an abstraction or
dependency. That run also wrote a changelog entry under the host's documentation
requirement; the baseline did not.

| CLI-reported sample | Without skill | With skill |
|---|---:|---:|
| Tool calls | 6 | 9 |
| Uncached input tokens, summed per assistant message | 10,537 | 22,397 |
| Cached input tokens, summed per assistant message | 44,928 | 68,992 |
| Output tokens, summed per assistant message | 564 | 700 |
| Total tokens, including cached input | 56,029 | 92,089 |
| Command wall time, including startup | 42.34s | 60.21s |

This sample did **not** demonstrate savings. Skill discovery/loading and extra
documentation contributed overhead; cache state, model variation, and host
instruction compliance also differ. These are raw per-run measurements, not
universal token estimates or a causal benchmark. Correctness passed in both runs,
but one bounded pagination fixture cannot establish quality parity generally.

### Heavy task: persisted-data migration

A normal prompt requested a plan, not execution, for renaming a saved profile
field while malformed records and legacy writers existed. The model did not read
the lean skill body. It retained a normal migration plan: inventory readers and
writers, preserve data, mixed-client compatibility, conditional/atomic writes,
idempotence, malformed-record handling, concurrent-write checks, backup/restore,
and rollback accounting for post-migration writes. No migration was executed.

### Escalation: apparently local null guard

A prompt described a small null guard, conditional on preserving behavior. The
model automatically read the skill, then the helper and its contract. The contract
revealed external consumers relying on errors to reject malformed imports.
The model made no edit and explained that returning an empty string would break
the public contract; caller investigation and a compatibility decision were
required. This exercises leaving the lean path when newly read evidence reveals
a heavy boundary, rather than merely predicting a label from a complete prompt.

### Reproduce a client smoke run

After installing the skill, place the helper above and an independent boundary
checker in a disposable directory. Use this same task prompt in two clean copies:

```text
Fix the off-by-one in the private paginate helper in pagination.mjs.
Array.slice uses an exclusive end; retain the existing input and output contract.
check.mjs covers first, middle, partial, past-end and empty pages.
Verify the corrected behavior. Work only in this fixture directory.
```

Run baseline with `--no-skills` and the candidate with `--skills lean-task`,
keeping the same model and tool configuration:

```sh
omp --no-session --no-title --no-extensions --no-rules --no-skills --tools read,edit,bash --model <your-model-id> --mode json -p "<task>"
omp --no-session --no-title --no-extensions --no-rules --skills lean-task --tools read,edit,bash --model <your-model-id> --mode json -p "<task>"
```

Inspect completed `agent_end` messages for tool calls, provider-reported usage,
and actual tool results. Confirm discovery via a read of `skill://lean-task`,
not just an assistant assertion. Keep credentials and personal paths out of
published transcripts. Repeat across models and tasks before making a savings claim.

## Local fast-path refinement (2026-10-06)

User testing reported excessive deliberation on push requests and unnecessary
specialist loading on a presentation-only game rename. The installed skill was
updated to choose one safe approach, load optional skills only for a concrete
correctness need, reuse unchanged verification, and provide routine push and
display-rename fast paths. Historical measurements above describe the initial
instructions, not this refinement.

Seven new stateless model samples matched the expected behavior:
- Known authorized push and unchanged documentation push: light, no additional
  optional skill, no repeat of unchanged checks or new commit.
- Display-only rename: light, no additional optional skill, focused render check,
  preserve save keys and internal/public identifiers.
- Force-push with unknown remote state: heavy.
- Rename with persisted save-key migration: heavy.
- Known local correction with a host-mandated specialist: light, but load the
  required guidance.
- Permission change with unknown policy: heavy.

A real omp smoke used the globally installed skill with normal discovery and a
disposable local Git remote. The model read the skill, checked Git state and the
configured destination, and performed a normal push. Git reported `main -> main`.
No new commit, implementation edit, or repeated test was performed. One additional
host-mandated skill was still read; the refinement cannot cancel higher-priority
requirements. No optional chain was loaded in that sample.

This checks task routing and execution, not improved latency or token savings.
The renamed-display scenario was decision-tested, not a browser walkthrough.

## Evidence-driven code and non-code update (2026-10-06)

The runtime instructions now prefer sufficient current evidence and direct
known-path reads, group independent operations when supported, and keep
dependent edits and verification ordered. Routine authorized commit/push and
trusted file-only skill installation use the same risk gate as bounded coding
work. Executable installers, unknown provenance, privilege changes, and
unauthorized overwrites do not qualify for the installation fast path.

Twenty-three independent, tool-free omp calls received the pilot-candidate
instructions and one synthetic scenario each. The release subsequently refined
the evidence-read policy without changing the risk gate. The actual response model was
`openai-codex/gpt-6.1-sol`, with thinking requested as `high`. All 23 matched
the expected classification and activation flag. Their proposed actions retained
the heavy boundaries, stale-evidence rechecking, mandatory host guidance, and
installation authorization constraints. These are decision samples, not proof
of repository execution, automatic discovery, or efficiency. Their usage and
duration are excluded from the task benchmark.


### Fresh three-arm task benchmark

A 27-run pilot compared no skill, the old snapshot, and a compact candidate
on pagination, CLI title rename, and document correction. All behavioral checks
passed. The pilot candidate removed directory inventories but did not improve
aggregate efficiency against the fresh baseline: 255.16s versus 209.32s, and
454,743 versus 431,387 total tokens. No pilot sample was discarded.

One evidence-driven refinement then made the stated contract and trusted checker
authoritative, reserving unchanged consumer/checker reads for a concrete missing
invariant or a mandatory host rule. A separate confirmation used six categories:
pagination, CLI rename, document correction, twelve-site mechanical update,
scoped commit/push to a disposable local bare remote, and trusted skill
installation with real omp discovery. Five repetitions and three arms produced
90 fresh runs. The release snapshot was
`0778f9c6dd56e748e5134cd8f4f0240108d48f8fcc704feb664b99ee99c60844`.

The model was `openai-codex/gpt-6.1-sol`, thinking requested as `high`, on
omp 18.6.1. Runs were sequential, arm order rotated, and each used a fresh
workspace/session. Prompt cache was shared and not purged. Two repetitions used
Indonesian; one of those supplied actual source-read snapshots and valid initial
failure evidence equally to every arm. Supplied context and language were
therefore not independent strata.

| Confirmation total, 30 runs per arm | No skill | Old skill | Release |
|---|---:|---:|---:|
| End-to-end seconds | 889.84 | 927.32 | 817.05 |
| Total tokens, including cache | 1,478,900 | 1,747,735 | 1,432,330 |
| Non-cache input | 341,373 | 373,621 | 356,606 |
| Cache-read input | 1,120,000 | 1,357,696 | 1,060,992 |
| Output | 17,527 | 16,418 | 14,732 |
| Tool calls | 187 | 205 | 174 |
| Completed assistant messages | 153 | 170 | 144 |
| Independent behavior checks passed | 30/30 | 30/30 | 30/30 |
| Strict scope/quality checks passed | 25/30 | 25/30 | 25/30 |

Against the fresh no-skill baseline, the release reduced aggregate time by
8.18%, total tokens by 3.15%, and tool calls by 6.95%. Against the old skill,
time decreased 11.89% and total tokens 18.05%. These are descriptive sample
differences, not controlled causal estimates or billing-cost measurements.

The 70–90% time and 30% total-token targets were **not achieved**. Aggregate
savings also did not satisfy the planned per-category no-regression floors:
the document-correction median was slower and used more tokens, and several
other categories had higher median token usage than baseline. No universal
improvement or equal quality beyond the tested boundaries is claimed.

All 15 Git runs passed actual commit-parent, scoped commit-tree, local remote-ref,
and unchanged/untracked scratch checks. Their strict scope flags failed because
the fixtures gained `.code-graph/index.db`, `.code-graph/index.lock`, and
`.gitignore`. These appeared in every arm and were not committed. Recorded model
tools were only read/bash, with no edit/write, and the first Git-status output
already showed the untracked ignore file. Host/indexing artifacts are a plausible
explanation; the exact writer was not isolated. Failed scope flags were retained,
not waived or replaced with favorable reruns.

An additional release escalation run read the external consumer contract, made
no edit, retained both TypeError cases, ran the checker, and requested authority
before changing the error contract. Its original automatic wording heuristic
incorrectly rejected the explanation; manual review confirmed the substantive
behavior. The original flag was retained, and the wording-based gate was removed
rather than re-pinned to the response's phrasing.

Every completed task response used the requested model, and per-message
`totalTokens` reconciled with input/output/cache components. Token usage was
summed only from assistant `message_end`, never duplicated from `agent_end`.
Startup, skill loading, model calls, tools, and shutdown are included in task
time; fixture setup and independent validation are excluded. Full logs, read
timestamps, versioned protocol, failed flags, and detailed Markdown reports stay
in local Git-excluded benchmark artifacts, not the published repository.

Skills CLI 1.7.0 also discovered exactly one skill from the updated local package.
After replacing only the user-global native `lean-task/SKILL.md`, a fresh
`omp read skill://lean-task` resolved the global copy and displayed the release
instructions. Its bytes matched the locally benchmarked release snapshot.


