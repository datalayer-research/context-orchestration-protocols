# Two things that look like gaps and are not

Part of [Context Orchestration Protocols](./index.md).

**`collect`.** Both adapters refuse it and neither should be read as a
protocol gap. Artifacts are read from the execution store, where they were
registered as they arrived; fetching them back from a worker would be asking
an untrusted party what it produced after the fact. No field.

**ACP `loadSession`.** Three of the six conformance skips are one registered
worker not declaring it, and that is ACP working as designed: the capability is
optional, and the adapter narrows its report per worker rather than opening a
second conversation and calling it the same one. An A2A extension does not fix
another protocol's optional capability; it is one to follow as ACP's proposals
stabilise.
