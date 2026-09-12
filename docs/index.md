# Context Orchestration Protocols

How an agent hands work to other agents over the protocols they already
speak — A2A and ACP — and what those protocols cannot say about that work
today. The documents here are written from things that were built and run,
not from a reading of the specifications, and they grow as the work does.
Where building something contradicted what we had written, the earlier
document is corrected and the correction is said out loud.

## The strategy

[Why this shape](./strategy.md) — four claims (orchestration is not a
transport; use the protocols that exist; the platform is authoritative, not
the session; reference context rather than copying it), what is deliberately
out of scope, why the proposal is an extension rather than a protocol, and
the order it is being built in.

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

## A durable control plane

[What it took](./durable-control-plane.md) to make a delegation outlive the
process that dispatched it: why dispatch moved into the durable worker, why
an ending nobody saw is unknown rather than failed, why a retry needs its own
run, and where the fifteen scenarios are actually proved. Several of these
are corrections to what the strategy assumed.

## Execution trees

[What changes](./execution-trees.md) when a worker becomes an orchestrator:
children requested rather than taken, an authority split between the identity
service and the control plane that replaced a single stronger enforcement
point, two budgets that look like one, why retry had to become a rerun, and
why the team specification turned out to describe a tree already.

## The extension, specified

[The normative document](./extension-v1.md): the URI, how it is negotiated on
A2A and on ACP, the one-key envelope, the fields each side sends, the one
method, and — marked as such — the fields that are specified because a
scenario needs them and are not yet on the wire. Optional throughout: a worker
implementing none of it still runs.

## Chapters to come

Written as the work they describe is built and measured:

- **Reference agents** — a worker implementing the extension and one
  implementing none of it, with a public conformance suite anybody can run
  against their own.
- **Interoperability** — traces of standard clients and of other
  orchestration protocols against the same scenarios.
- **What orchestration costs** — the overhead measures: acceptance latency,
  time to first worker event, context bytes referenced against copied, and
  the tokens orchestration itself adds.

Read more on <https://datalayer.ai/research/context/orchestration-protocols>.
