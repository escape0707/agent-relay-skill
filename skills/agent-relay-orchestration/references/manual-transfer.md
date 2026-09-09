# Manual Transfer Reference

Read this file only when a round requires browser/manual transfer or GitHub relay is unavailable.

## When to use manual transfer

Use manual transfer for required artifacts that are sensitive, private, large, binary, awkward for Git, or otherwise unsuitable for durable Git history.

Do not commit secrets such as passwords, private keys, access tokens, or credentials. Necessary sensitive task data may be transferred manually.

## Naming

Recommended manual filenames:

```text
round-<NNNN>-prompt-<slug>.md
round-<NNNN>-response-<slug>.md
round-<NNNN>-return-<slug>.zip
```

Example:

```text
round-0004-prompt-audit-czkawka-cache.md
round-0004-response-audit-czkawka-cache.md
round-0004-return-audit-czkawka-cache.zip
```

## Single-upload return

A completed round requires at most one manual upload action by the user.

If any manual return is required, prepare exactly one return ZIP containing:

- the round response Markdown;
- every artifact that requires manual transfer;
- a small manifest only when it adds useful integrity or identification information.

Include the response even when it was also committed to the relay branch so the ZIP is self-contained.

Prepare the complete ZIP before publishing the terminal `response.md`.

Publishing `response.md` still makes the round terminal. If the ZIP has not yet reached the Main Agent, the Main Agent should treat manual delivery as not yet ingested; `next round` MUST NOT rerun the terminal round.

## Git references and hashes

The relay branch MAY contain non-sensitive references to manually transferred material, such as filename, purpose, and SHA-256.

Do not create `artifacts.json` merely because manual transfer occurred. Use it only when a machine-readable artifact inventory or integrity record is useful.

Avoid recursive ZIP self-hashing: do not require a response inside the ZIP to contain the final ZIP's own SHA-256. If the ZIP hash is useful, record it outside the ZIP after creation, for example in `artifacts.json` or a later forward-moving relay commit.

## GitHub-unavailable fallback

If GitHub relay itself is unavailable, use the same round protocol through manual files:

- Main sends one English prompt Markdown file;
- Subagent returns one self-contained ZIP containing `response.md` and required artifacts;
- normal prompt/response semantics and permission rules still apply.

A direct prompt-file path is strong evidence that the receiving agent is acting as Subagent for that round.
