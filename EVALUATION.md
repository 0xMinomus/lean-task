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

The local Skills CLI 1.7.0 discovered exactly one skill, `lean-task`, from the
repository layout. This checks packaging/discovery, not behavior in every client.
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
