# Context Orchestration Protocols

How an agent hands work to other agents over the protocols they already
speak — A2A and ACP — and what those protocols cannot say about that work
today. The documents here are written from things that were built and run,
not from a reading of the specifications, and they grow as the work does.

## What A2A and ACP could not express

The protocol gap report: one parent agent delegating the analysis of a real
notebook to an A2A worker and to an ACP worker over one code path, a
conformance suite of fifteen orchestration scenarios run against both, and
the gaps both surfaced. Each gap maps to one proposed extension field, and
three fields the scope started with do not survive the rule that a field
needs a scenario that shows it.

1. [How the gaps were found](./method.md) — the conformance suite, the
   two-protocol example, and the rule the report is written to.
2. [The gaps, and the field each one maps to](./gaps.md) — nine gaps, where
   each showed up, and the scenario that needs its field.
3. [Fields that no scenario needs](./out-of-scope.md) — pause and resume,
   trace propagation, and worker release: why each is out of the first
   version.
4. [Two things that look like gaps and are not](./not-gaps.md) — `collect`
   and ACP `loadSession`.
5. [What a plain worker still gets](./plain-workers.md) — the reduction a
   worker that implements no extension field is reported, not hidden.

## Chapters to come

Written as the work they describe is built and measured:

- **A durable control plane** — executions, attempts and leases that
  outlive the process that dispatched them; what recovery looks like when a
  worker dies after it accepted work.
- **Execution trees** — children requested rather than taken, delegation
  that narrows authority at every level, and events aggregated up the tree.
- **The extension, specified** — the fields above as an A2A extension, with
  reference agents and a public conformance suite.
- **Interoperability** — traces of standard clients and of other
  orchestration protocols against the same scenarios.

Read more on <https://datalayer.ai/research/context/orchestration-protocols>.
