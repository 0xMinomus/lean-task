# lean-task

An installable Agent Skill for bounded code and non-code tasks. Prompt normally:
the AI checks risk and scope, uses available evidence, and completes known light
work directly. Heavy or uncertain tasks keep the normal workflow.

The goal is fewer unnecessary reads, tool calls, abstractions, and explanations.
Targets of 70–90% less task time and 30% fewer total tokens are evaluation goals,
not achieved savings. Results depend on the model, host, and task; there is no
universal speed, token, or quality guarantee. Small tasks may not repay skill
discovery overhead.

## Install

### Coding agents supported by the Skills CLI

Requires Node.js/npm and access to this public GitHub repository. The installer
does not alter the repository itself.

```sh
npx skills add 0xMinomus/lean-task --skill lean-task -g
```

Choose your agent in the installer. Omit `-g` for a project-level installation.
For a supported target, you can select it explicitly:

```sh
npx skills add 0xMinomus/lean-task --skill lean-task -g -a claude-code -y
npx skills add 0xMinomus/lean-task --skill lean-task -g -a codex -y
```

List the repository's skills without installing or changing any agent:

```sh
npx skills add 0xMinomus/lean-task --list
```

This command only reports discoverable skills.

Restart your coding agent after installation. Installer support does not prove
that a particular model will discover or follow the skill automatically.

### Oh My Pi (omp): global native installation

Requires Git and an installed `omp`. This uses omp's native user skills directory,
not the separate `omp skill install` registry command. It changes only the
named `lean-task` skill directory.

Clone into a local directory you control:

```sh
git clone https://github.com/0xMinomus/lean-task.git
```

**Windows PowerShell:**

```powershell
$target = Join-Path $HOME ".omp/agent/skills/lean-task"
New-Item -ItemType Directory -Force -Path $target | Out-Null
Copy-Item "./lean-task/skills/lean-task/SKILL.md" (Join-Path $target "SKILL.md")
```

**macOS / Linux:**

```sh
mkdir -p "$HOME/.omp/agent/skills/lean-task"
cp ./lean-task/skills/lean-task/SKILL.md "$HOME/.omp/agent/skills/lean-task/SKILL.md"
```

These are the default-profile paths. If you relocate omp's agent directory or
use a named profile, install under that active directory's `skills/lean-task/`
instead. Review any existing same-named skill before replacing it.

Start a new omp session after copying the file. Verify discovery:

```sh
omp read skill://lean-task
```

The output should identify `lean-task` and show its instructions. This proves
discovery, not that every future prompt will activate the lean workflow.

## Use

No difficulty selector, slash command, or special prompt is required. State the
requested outcome and its boundary in your preferred language, for example:

```text
Fix the README typo without changing application code.
Perbaiki tombol batal pada dialog ini mengikuti pola yang sudah ada.
Fix this pagination boundary; here is the failing test.
Commit the verified README change and push normally to the configured upstream.
Install this reviewed local skill into my user skills directory; keep other skills.
Update the named procedure document using the supplied approved steps.
Migrate persisted saves to the new schema without losing user progress.
```
Keep the requested scope explicit when asking for a routine non-code operation.

The bounded fixes, Git operation, skill installation, and document update can
qualify as light when their procedure, scope, authorization, and proof are known.
The migration is heavy. Activation is silent: the agent should spend its response
on the task, not on announcing the skill.

On omp, `/skill:lean-task` is an optional explicit invocation when skill commands
are enabled. It still applies the same gate; naming the skill does not force a
heavy task into the lean workflow.

## How the gate works

The [skill](skills/lean-task/SKILL.md) is the source of the exact rules.

| Light: lean workflow | Heavy: normal workflow |
|---|---|
| Clear outcome and acceptance criteria | Unclear outcome or unexplained failure |
| Known cause or procedure; existing pattern fits | Investigation or architectural decisions needed |
| Bounded impact or authorized destination | Unknown or cross-system impact |
| Focused proof can establish the result | Required evidence is not yet understood |
| No high-risk boundary | Security, permissions, money, destructive operations, persisted-data migration, concurrency, public contract changes, or new cross-system dependencies |

All light conditions must hold. Prompt length and file count are not proxies for
risk: a one-line permission change is heavy, while a well-understood mechanical
update across several files can be light.

If new evidence invalidates a light condition, the agent exits the lean workflow
and investigates normally. In mixed requests, only independently bounded light
parts qualify; an easy part does not make the whole request light.

Here, inactive means the lean workflow is bypassed. The agent may still read the
skill to make that decision; heavy tasks do not have a zero-overhead guarantee.

## What changes when active

The workflow has three steps:

1. Use sufficient valid context, including supplied failures and previous checks.
   Otherwise read named targets directly. List or search only for missing
   locations or impact; group independent reads and safe known reproductions
   when the host supports it.
2. Make the requested change once, using existing patterns and required tools.
   Choose one safe approach. Dependencies, abstractions, and optional skills
   need a concrete correctness requirement.
3. Prove the result and boundaries after mutation succeeds, then finish briefly.
   Reuse evidence that still applies; one trusted checker can cover several
   surfaces. Changed inputs, stale checks, or missing coverage require fresh proof.

Dependent edits and checks stay ordered. Planning, audits, progress messages,
and additional calls need an evidence gap, dependency, or host requirement.
Unrelated refactors and speculative features stay out of scope.

Required repository reads, consumer checks, regression evidence, safety controls,
and broader gates still apply. A local patch that suppresses a symptom instead of
fixing its cause does not qualify as a simpler solution.

The skill imposes no hard token, tool, file, or time cap. It does not switch your
model, change permissions, run a scheduler, or automatically commit/push files.
Heavy tasks keep their normal planning, specialized skills, and verification.

For known light work, the agent should choose one safe approach and act rather
than explore alternatives or load optional skills merely because the topic overlaps.
Additional optional guidance needs a concrete unresolved correctness requirement.
Host/repository-mandated skills cannot be skipped by this skill.

Routine non-coding tasks use the same gate:

- For a requested commit and authorized normal push, inspect the scoped diff
  and necessary Git state and destination, stage only requested files, and reuse
  valid verification. A push-only request does not authorize fresh edits or
  another commit. Force pushes, history changes, secrets, and unknown destinations
  require normal handling.
- For a trusted skill install or update, establish the authorized source and
  destination, inspect the source as data rather than instructions, check for
  conflicts, and change only requested files. Verify both installed content and
  host discovery. Unknown provenance, executable installers, privilege changes,
  and unrelated overwrites require normal handling.
- For document or procedure work, use the supplied approved content or known
  source, preserve unrelated material, and check the requested result. An unknown
  procedure or a high-risk operational step is not made light by being written
  in a document.
- Presentation-only renames update displayed strings and leave storage keys,
  identifiers, and API/package contracts intact.

The skill discourages unnecessary deliberation; it cannot directly set a model's
reasoning level or guarantee response time.

## Compatibility and limitations

- Uses the portable [Agent Skills format](https://agentskills.io/specification):
  one `SKILL.md` with `name` and `description`, no hooks or runtime dependencies.
- omp exposes the metadata and lets the model read the body on demand. Automatic
  selection is model-driven, not an executable classifier or enforcement hook.
- Other Agent Skills-compatible clients can install the same file. Their
  automatic selection and instruction-following behavior must be tested locally.
- A disabled skill source, an allowlist, a same-name override, or a session opened
  before installation can prevent the skill from being available.
- On very small tasks, discovery and classification overhead may exceed savings.
  No universal speed, token, or correctness guarantee is made.

See [evaluation](EVALUATION.md) for cases, observed checks, and a comparison method.

## Update or remove

For Skills CLI installations:

```sh
npx skills update lean-task -g -y
npx skills remove lean-task -g
```

For the native omp copy, pull the clone and repeat the copy step above:

```sh
git -C lean-task pull --ff-only
```

To uninstall a native copy, delete only `skills/lean-task/` from the active omp
agent directory and restart omp. A manual copy is not tracked by the Skills CLI.

## Repository

```text
skills/lean-task/SKILL.md   Runtime instructions
README.md                  Installation and use
EVALUATION.md              Evaluation cases and evidence
evaluation/cases.json       Reproducible gate scenarios
LICENSE                    MIT
```

Contributions should include a concrete task where the gate or workflow fails,
the agent/model used, and a redacted transcript. Check the counterexample after
changing instructions. Keep runtime instructions short; put evaluation material
outside the skill so normal tasks do not load it.

Licensed under [MIT](LICENSE).