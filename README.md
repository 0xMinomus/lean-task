# lean-task

An installable Agent Skill that keeps light tasks small without cutting required
verification. Prompt normally: the AI checks risk and scope, uses a lean workflow
for bounded work, and leaves heavy or uncertain tasks to its normal workflow.

The goal is fewer unnecessary abstractions, tool calls, and explanations.
Savings depend on the model and task; there is no claimed percentage improvement
or guarantee of unchanged quality across every model.

## Install

### Coding agents supported by the Skills CLI

Requires Node.js/npm and access to this public GitHub repository.

```sh
npx skills add 0xMinomus/lean-task --skill lean-task -g
```

Choose your agent in the installer. Omit `-g` for a project-level installation.
For a supported target, you can select it explicitly:

```sh
npx skills add 0xMinomus/lean-task --skill lean-task -g -a claude-code -y
npx skills add 0xMinomus/lean-task --skill lean-task -g -a codex -y
```

List the repository's skills without installing:

```sh
npx skills add 0xMinomus/lean-task --list
```

Restart your coding agent after installation. Installer support does not prove
that a particular model will discover or follow the skill automatically.

### Oh My Pi (omp): global native installation

Requires Git and an installed `omp`. This uses omp's native user skills directory,
not the separate `omp skill install` registry command.

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

Open a new omp session. Verify discovery:

```sh
omp read skill://lean-task
```

The output should identify `lean-task` and show its instructions. This proves
discovery, not that every future prompt will activate the lean workflow.

## Use

No difficulty selector, slash command, or special prompt is required. Write your
normal request in your preferred language, for example:

```text
Fix the README typo without changing application code.
Perbaiki tombol batal pada dialog ini mengikuti pola yang sudah ada.
Fix this pagination boundary; here is the failing test.
Migrate persisted saves to the new schema without losing user progress.
```

The first three can qualify as light once their scope and implementation path
are known. The migration is heavy. Activation is silent: the agent should spend
its response on the task, not on announcing the skill.

On omp, `/skill:lean-task` is an optional explicit invocation when skill commands
are enabled. It still applies the same gate; naming the skill does not force a
heavy task into the lean workflow.

## How the gate works

The [skill](skills/lean-task/SKILL.md) is the source of the exact rules.

| Light: lean workflow | Heavy: normal workflow |
|---|---|
| Clear outcome and acceptance criteria | Unclear outcome or unexplained failure |
| Known cause or implementation path; existing pattern fits | Investigation or architectural decisions needed |
| Bounded impact; affected consumers identified | Unknown or cross-system impact |
| Focused verification can prove the behavior | Required evidence is not yet understood |
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

The AI bounds the requested result, reads relevant context, makes the direct
change using existing patterns, runs proportionate checks, and stops at verified
completion. Abstractions and dependencies need a current concrete requirement.
Unrelated refactors, speculative features, and repeat investigations stay out of
scope.

Required repository reads, consumer checks, regression evidence, safety controls,
and broader gates still apply. A local patch that suppresses a symptom instead of
fixing its cause does not qualify as a simpler solution.

The skill imposes no hard token, tool, file, or time cap. It does not switch your
model, change permissions, run a scheduler, or automatically commit/push files.
Heavy tasks keep their normal planning, specialized skills, and verification.

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