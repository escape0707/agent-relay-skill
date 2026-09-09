---
name: agent-relay-orchestration
description: Shared protocol for a main/orchestrator agent and delegated subagents using one-to-one GitHub relay branches, compact round files, and manual artifact fallback.
---

# Agent Relay Orchestration

## 1. Purpose and Core Invariants

Use this SKILL when one agent owns strategy and final synthesis while another agent performs delegated work in a different environment.

The same SKILL applies to Main Agents and Subagents.

Core invariants:

1. The Main Agent owns strategy; the Subagent owns delegated execution.
2. One relay branch has exactly one Main↔Subagent writer pair.
3. Runtime relay branches live in the private repository `<github-login>/agent-relay`, where `<github-login>` is the GitHub identity authenticated for relay operations.
4. Local relay Git state lives under `~/workspaces/agent-relay`, separate from task-project worktrees.
5. Only the Main endpoint for a relay channel creates formal new rounds.
6. Role is contextual and relational, never inferred from model capability alone.
7. Published relay history moves forward; do not rewrite it by default.
8. Do not invent protocol state, registries, logs, or housekeeping files.
9. Required sensitive or large artifacts use manual transfer rather than Git.
10. Nested high-capability advisory sessions are ephemeral advisory leaves, not persistent workers.

Formal relay materials MUST be in English: `prompt.md`, `response.md`, relay commit messages, bootstrap prompts, and protocol artifacts unless task semantics require otherwise. User-facing conversation stays in the language established with the user unless they request a change.

---

## 2. Role Inference and Responsibilities

### Role inference

Prefer Main / Orchestrator when the agent is planning, coordinating, creating rounds, reviewing results, or lacks direct access to the delegated environment.

Prefer Subagent when the user provides a relay branch and asks to execute its pending round, provides a concrete prompt-file path, or otherwise clearly delegates execution in an environment with the required local capabilities.

A concrete bootstrap such as this is strong Subagent evidence:

```text
Read and follow the installed Agent Relay SKILL.

Relay branch: relay/laptop/photo-import-cleanup

Execute the pending round.
```

An agent MAY be Subagent upstream and Main downstream. Model family, reasoning level, product name, and local-vs-web execution do not determine role by themselves.

### Main Agent

The Main Agent MUST:

- own the overall goal, strategy, risk posture, and final answer;
- decide when delegation is useful;
- create the relay branch and every formal round for that channel;
- define round scope, permissions, tasks, and deliverables;
- review `response.md`, artifacts, work commits, deviations, user decisions, and nested relays;
- integrate results into the user-facing reasoning.

When starting a Subagent session, the Main Agent SHOULD provide a short copy/paste bootstrap containing the relay branch.

### Subagent

The Subagent MUST:

- execute only the current delegated round;
- stay within the prompt's allowed scope unless the user explicitly changes it;
- act conservatively when scope is ambiguous;
- distinguish findings from actions and side effects;
- report durable user decisions and material deviations;
- prepare required artifacts before publishing a terminal response;
- never create the next formal round;
- never silently edit a published Main-Agent prompt.

### Permission precedence

Apply permissions in this order:

1. system/platform constraints;
2. current explicit user instruction;
3. Main-Agent round prompt;
4. Subagent judgment.

User authorization is narrow and local to what was actually authorized. If it changes the upstream permission boundary, report the change under `## Deviations`.

---

## 3. Relay Infrastructure and Topology

### Runtime repository

The relay repository is:

```text
<github-login>/agent-relay
```

It MUST be private. Resolve `<github-login>` from the GitHub identity authenticated for relay operations. If the repository does not exist, report the missing prerequisite; do not create it unless the user explicitly asks. Do not introduce a persistent repository-binding config merely for convenience.

The relay repository is independent of task-project repositories. Do not place relay protocol state in a project repository just because the agent is running there.

### Local relay workspace

Keep local relay Git state under:

```text
~/workspaces/agent-relay/
├── repo/
└── worktrees/
    └── <relay-branch-safe-name>/
```

`repo/` is the reusable clone. Per-channel worktrees are disposable execution surfaces. The GitHub relay branch is the durable source of truth.

On first local use, reuse or create the clone, fetch the requested branch, create/use an isolated worktree, and verify Git read/write and publication capability before substantial work.

### Relay branches

One relay branch MUST have exactly one Main↔Subagent writer pair. Parallel Subagents require separate branches.

Recommended naming:

```text
relay/<endpoint-or-agent-name>/<task-slug>
```

Examples:

```text
relay/desktop/nas-cache-sharing
relay/laptop/photo-import-cleanup
relay/nas/permission-audit
```

Actual project work stays on the user's normal project branch convention, such as `<agent-name>/<branch-name>`. Relay branches carry communication; work branches carry project changes. Reference work commits instead of copying large diffs into the relay repository.

---

## 4. Round Model and State

Each relay branch uses this visible root-level structure:

```text
rounds/
└── 0001-<slug>/
    ├── prompt.md
    ├── response.md       # absent while pending
    ├── artifacts/        # optional
    └── artifacts.json    # optional, only when useful
```

Do not add a hidden wrapper directory.

Do not create `session.json`, `round.json`, `running.json`, heartbeats, locks, status files, registries, cumulative summaries, or narrative logs as protocol state.

Round state is inferred only from file presence:

```text
prompt.md exists + response.md absent = pending/open
response.md exists                     = terminal/responded
```

`response.md` outcome MUST be one of:

```text
Completed
Partial
Blocked
```

All three are terminal. `Partial` or `Blocked` does not authorize automatic rerun or creation of another round; the Main Agent decides any follow-up.

### Round selection

For `next round`, the Subagent MUST:

1. fetch/pull the relevant relay branch;
2. find valid round directories whose `prompt.md` exists and `response.md` does not;
3. select the LOWEST-NUMBERED pending round;
4. execute exactly that round.

If no pending round exists, report that briefly and do not create one.

The Main Agent SHOULD normally keep only one pending round per channel. It MAY queue several only when they are intentionally independent.

### Publishing a terminal response

Before publishing `response.md`:

1. reach a reportable outcome;
2. prepare GitHub-safe artifacts;
3. prepare any required manual return package;
4. record relevant hashes/references if useful;
5. publish `response.md`.

The existence of `response.md` makes the round terminal. Do not publish it while promising required artifacts that do not yet exist.

---

## 5. Prompt Contract

Use this as the strong default prompt shape:

```markdown
# Round 0004 — <Title>

## Objective
...

## Context
...

## Scope
### Allowed
- ...
### Not Allowed
- ...

## Tasks
1. ...

## Deliverables
- `response.md`
- ...

## Return Policy
- GitHub-safe artifacts: ...
- Sensitive/large artifacts: ...
```

The Main Agent MAY adapt sections when task semantics require it. Round-specific explicit instructions take precedence over generic formatting guidance.

Keep task instructions in `prompt.md`. Do not duplicate identifiers or metadata that are already unambiguous from the branch/path merely to satisfy a schema.

---

## 6. Response Contract

Use this as the strong default response shape:

```markdown
# Round 0004 — Response

## Outcome
Completed / Partial / Blocked

## Findings
...

## User Decisions / Instructions
None.

## Actions Performed
...

## Changes
...

## Validation
...

## Artifacts
...

## Nested Relays
None.

## Deviations
None.

## Issues / Decisions Needed
None.
```

Preserve these semantic distinctions even if formatting adapts:

- `Findings`: conclusions, observations, and evidence.
- `User Decisions / Instructions`: durable user choices or instructions that matter upstream; not conversational transcript.
- `Actions Performed`: actual operations and side effects.
- `Changes`: project/work-branch refs and concise change summary; do not duplicate large diffs.
- `Validation`: tests/checks and outcomes.
- `Artifacts`: committed artifacts and declared manual package.
- `Nested Relays`: every nested relay created during the round, or `None.`
- `Deviations`: material departures from the original prompt, including user-authorized scope changes, or `None.`
- `Issues / Decisions Needed`: unresolved upstream decisions, or `None.`

For project changes, prefer references such as:

```text
Work repo: owner/project
Work branch: codex/cache-sync
Base commit: abc123...
Result commit: def456...
```

If there were no project worktree changes, say so explicitly.

The Main Agent SHOULD inspect `User Decisions / Instructions`, `Deviations`, `Nested Relays`, relevant work commits/diffs, validation, and artifacts before deciding whether another round is needed.

---

## 7. User Intervention and Control Commands

The user is an authorized in-band participant in a Subagent session. The user MAY clarify, steer execution, change scope, authorize a specific operation, reject an approach, choose between options, request related checks, or ask that a durable decision be carried upstream.

- Ordinary clarification need not be reported unless it changes the outcome.
- Operational intervention belongs in `Actions Performed`.
- Scope-changing intervention also belongs in `Deviations`.
- Durable choices/messages for the Main Agent belong in `User Decisions / Instructions`.

Related user additions during execution stay in the current round. The Subagent MUST NOT invent `R0004a`, `R0004b`, or `R0005`. If the user directly requests an independent extra task and it is permitted, the Subagent MAY perform it as an in-session extension and report it, but it still does not create a formal new round.

### `next round`

Unambiguous protocol command: select and execute the lowest-numbered pending Main-issued round.

### `next`

Interpret contextually. In a quiet relay-oriented session it SHOULD mean `next round`. In active conversation, phrases such as `sure, next` may simply mean continue the current discussion/workflow.

### `pause`

Stop further execution and leave the round open. Do not publish a terminal response unless the user explicitly requests a partial report.

### `resume`

Continue the paused/interrupted open round. Reconstruct from `prompt.md`, Git/work state, artifacts, and verifiable machine state. Never blindly repeat non-idempotent side effects.

### `continue`

Normally continue the current reasoning or execution context; it is weaker and more conversational than `next round`.

---

## 8. Git, Work Branches, and Recovery

Published relay history SHOULD move append-forward.

By default, do not:

```text
git push --force
git push --force-with-lease
rebase already-published relay history
amend already-published relay commits
```

Published prompts are immutable. Do not silently rewrite a published response. Correct it with a forward commit, for example:

```text
[relay][R0004][response-fix] Correct artifact SHA
```

Recommended relay commits:

```text
[relay][R0004][prompt] <title>
[relay][R0004][response] <title>
[relay][R0004][response-fix] <description>
```

If publication is rejected because the remote moved, fetch first, preserve published remote history, safely integrate unpublished local work, and push a forward-moving result. Unpublished local commits MAY be rebased before publication. Never resolve a relay conflict by overwriting the other endpoint.

For interrupted execution, leave the round open and use `resume`. Reconstruct from real Git state, project state, artifacts, and machine state rather than inventing persistent recovery metadata.

Cleanup is explicit, not automatic. Do not create a final-summary file by default. Relay-branch deletion requires explicit cleanup intent. Project work-branch merge/deletion follows normal development workflow.

---

## 9. Artifact Transport

Use Git for GitHub-safe relay materials such as text, compact structured data, scripts, source code, and small reports.

Use manual transfer for required artifacts that are sensitive, private, large, binary, or otherwise unsuitable for durable Git history. Never commit passwords, private keys, access tokens, credentials, or similar secrets. A private repository is not a secret vault.

If any manual return is required:

- return at most one ZIP for the round;
- make the ZIP self-contained and include the round response;
- prepare the ZIP before publishing the terminal `response.md`;
- read [`references/manual-transfer.md`](references/manual-transfer.md) before packaging or documenting the transfer.

`artifacts/` and `artifacts.json` are optional. Do not create them when there is nothing useful to store or describe.

---

## 10. Nested Advisory Relays

Nested orchestration is allowed. A Subagent may act as Main relative to a child agent, but every child channel still follows the one-to-one writer rule.

A high-capability child session created for explanation, teaching, unfamiliar-domain reasoning, decision support, user steering, or independent high-quality analysis is an **ephemeral advisory leaf**.

If one is created or handled:

- it MUST use its own relay branch;
- its existence and exact discoverability information MUST be propagated upward through every ancestor;
- the parent MUST NOT replace the child's formal `response.md` with its own rewrite or summary;
- ancestors SHOULD read the original child response directly;
- after the final child response is propagated upward and absorbed, that leaf is terminal and MUST NOT receive follow-up rounds;
- a later independent advisory need requires a new child session and a new relay branch;
- read [`references/nested-advisory.md`](references/nested-advisory.md) when creating, reporting, or consuming such a relay.
