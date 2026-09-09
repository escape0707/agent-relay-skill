# Nested Advisory Relay Reference

Read this file when creating, reporting, or consuming an ephemeral advisory leaf.

## Role relationship

A parent agent may be Subagent on its upstream channel while acting as Main Agent on the child channel. The direct parent creates the child relay and its round; the parent and child are the only writers to that relay branch.

Ancestor agents MAY read the child relay after discovery is propagated, but SHOULD NOT write to it.

## Intended use

An ephemeral advisory leaf is a one-off high-capability session used for explanation, teaching, unfamiliar-domain reasoning, decision support, user steering, or independent high-quality analysis. It is not a persistent worker channel.

The user MAY interact directly and deeply with the advisory leaf while its round is open. Important conclusions and durable user decisions from that interaction belong in the child's own final `response.md`.

## Upward discoverability

Every parent that creates or learns of a nested advisory relay MUST propagate its existence upward until the top-level Main can discover it.

The parent `## Nested Relays` entry SHOULD provide exact discoverability information:

```text
- Relay repository: <github-login>/agent-relay
- Relay branch: relay/web-advisory/explain-rust-lifetimes
- Purpose: Explain lifetime constraints relevant to the current decision.
- Child role: Ephemeral advisory leaf.
- Relevant completed round: 0001-explain-borrowing-model
- Result commit: abc123...
```

Report location and identity, not a substitute summary of the child's content.

## Original response is authoritative

The direct parent MUST NOT replace the child's formal `response.md` with its own rewritten or paraphrased version. Ancestor Main Agents SHOULD inspect the original child response directly when incorporating the result.

The parent may describe its own subsequent actions or conclusions in its parent response, but that does not substitute for the child artifact.

## Terminal lifecycle

A nested advisory relay is terminal after:

1. the leaf publishes its final response;
2. its relay location is propagated upward;
3. the relevant ancestor Main absorbs the result.

After that, do not send follow-up rounds to that leaf and do not treat it as a persistent worker.

If a later independent advisory need appears, create a new child session and a new relay branch rather than reviving the completed leaf.
