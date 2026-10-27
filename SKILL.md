---
name: change-test-review
description: Analyze functionality changes in this project, map UI, business logic and database dependencies, assess risks, and plan and run relevant existing tests. Require explicit approval before editing application or test code.
---

# Change test review

Assess a requested or existing functionality change and provide evidence-based test results and a reviewable proposal for missing coverage. Invoking this skill does not approve code edits or establish a file-watching automation.

Example invocation: `$change-test-review Review the checkout changes, assess dependencies and risks, and plan and run relevant existing tests. Propose any application or test edits for approval.`

## Analyze the application and change

- Locate the project root containing this skill under `.agents/skills/`; resolve project file paths below from that root. Read applicable `AGENTS.md` guidance first.
- Inspect `README.md`, relevant task requirements, implementation, test configuration, and existing tests. Cite project paths and line numbers for findings and requirements.
- Identify the requested behavior and changed files. Use an available diff and state its baseline. Preserve existing user changes. If no diff or clear change description is available, inspect independently and ask which change or baseline to assess before making change-specific conclusions.
- Distinguish documented requirements, observed behavior, and unknown decisions. Do not invent validation or rounding policies.

## Map dependencies

Trace callers, affected rules, data flow, imports, and test fixtures for the changed behavior. Report a compact table covering:

| Layer | What to identify |
|---|---|
| UI | Entry points, components, user flows, and links to affected logic |
| Business logic | Public functions, input/output contracts, rules, and callers |
| Database | Persistence adapters, models, queries, migrations, and fixture requirements |
| External dependencies | Services, packages, configuration, and execution prerequisites |

Mark each layer as **present**, **not found in inspected scope**, or **unknown**, with evidence and the scope inspected. Do not add UI, database, mocks, or integrations just to fill the table.

## Generate a plan and identify risks

Before execution, present a concise plan mapping affected behavior to existing tests and proposed coverage. Include commands, working directory, prerequisites, expected outcomes, and execution order. Separate existing tests that can run immediately from proposed tests requiring edits.

Assess evidence-supported risks such as compatibility changes, arithmetic precision, untested branches, missing dependencies, and integration side effects. State impact and mitigation. Flag uncertainty without treating it as a confirmed defect.

For this checkout project, verify these anchors against the current files rather than assuming they remain unchanged:

- `src/checkout.py`: calculation entry point and float contract.
- `tests/test_checkout.py`: standard-library unit tests for single items, multiple items, empty carts, and float results.
- `task_card.md`: discount requirements and formatting requirements are separate tasks; review only the requested functionality.
- `sample_orders.json`: synthetic sample input, not evidence of a database.

For discount changes, assess default compatibility, 0%, 10%, 100%, out-of-range values, keyword-only usage, and float results. For formatting changes, assess whole-dollar and fractional values, trailing zeros, and separation from calculation. Apply cases only when supported by the requested task and current contract; clarify unspecified policies first.

## Run existing tests

Inspect test setup and side effects before running. Invocation authorizes running relevant existing local tests with no destructive or external effects; it does not authorize source edits.

Run from the project root using an available Python interpreter:

```sh
python3 -B -m unittest discover -s tests -v
```

On Windows, use `py -3 -B -m unittest discover -s tests -v` when supported. The `-B` flag avoids writing bytecode caches. Use targeted existing tests first when helpful, followed by the project suite for a small change.

If the interpreter or another prerequisite is unavailable, report the precise blocker. Do not install dependencies, provision services, or modify configuration without approval. Obtain approval before tests that write database data, call external services, or have destructive effects; describe the target environment and side effects first.

Do not change application code, tests, fixtures, snapshots, dependencies, schemas, or configuration to make tests pass before the approval gate below.

## Approval before edits

The user requires approval before any application or test code changes. Complete available analysis and safe existing-test execution first. Present a concrete proposal including:

- Each file to edit and the behavior or coverage gap addressed.
- Proposed code changes or a focused patch, shown in the conversation without applying it.
- Test inputs, expected outcomes, risks, and verification commands.

Ask explicitly for approval of those edits and wait. A request to review, run tests, or use this skill is not approval to edit. Prior explicit approval of the same concrete edits is sufficient; do not ask again unnecessarily. If approval is declined, finish with findings and unapplied suggestions. If approval is granted, apply only the approved scope and run appropriate checks. Seek approval again for materially expanded edits.

## Report results

Report the inspected scope and baseline, dependency findings, risks, commands and actual outcomes, coverage gaps, and any pending approval. Distinguish passed, failed, skipped, blocked, and unexecuted checks. Cite evidence for conclusions; do not infer that a failure was introduced by the change without baseline evidence. After approved edits, summarize changed files and remaining limitations.
