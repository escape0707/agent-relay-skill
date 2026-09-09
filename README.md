# Agent Relay Orchestration

A reusable agent skill for coordinating a Main/Orchestrator Agent with delegated Subagents through one-to-one Git relay branches, with manual fallback for sensitive or large artifacts.

## Install

```bash
npx skills add escape0707/agent-relay-skill --skill agent-relay-orchestration
```

Runtime relay state uses a separate private `<github-login>/agent-relay` repository. Create it once if needed:

```bash
gh repo create "$(gh api user --jq .login)/agent-relay" --private
```

## Structure

```text
README.md
AGENTS.md
skills/
└── agent-relay-orchestration/
    ├── SKILL.md
    └── references/
        ├── manual-transfer.md
        └── nested-advisory.md
```
