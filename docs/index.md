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

## OASF and AGNTCY, evaluated

[The mapping](./oasf-and-agntcy.md), read from the schema rather than from
its description: four of eleven `Record` fields match the descriptor
exactly, `locators` is a download pointer and not where an endpoint goes,
`modules` is the right extension point for both the endpoint and the
capability vocabulary, seven descriptor fields have no OASF home yet, and
directory publication is a separate deployment decision the schema mapping
does not force. Not built — Phase 4 is demand-driven and there is none yet —
but the answer is written down rather than left open.

## Reference agents, and a public conformance suite

[The worker, and the tests](./reference-agents.md): the smallest thing that
speaks the extension correctly — no model, no framework beyond `fasta2a`, no
Datalayer control plane — driven over real ASGI HTTP by hand-built JSON-RPC
requests matching the normative wire shapes directly. Six scenarios,
runnable by anyone against their own worker.

## Interoperability

[Traces](./interoperability.md) of a standard A2A client against the
reference worker, and of Datalayer's own dispatch pattern against a worker
that has never heard of this extension — against `a2a-sdk`, the A2A
project's own reference implementation, pinned and independent of
`fasta2a`. Looking for a trace found two real, previously undocumented
gaps between what `fasta2a` actually serves and what the specification
requires: a card missing its required `url`, and a `message/send` result
wrapped where the spec wants it bare — the second one load-bearing enough
that no specification-conformant client can complete a basic send against
any `fasta2a`-based worker today, Datalayer's production ones included.
Neither is this project's bug to fix; both are recorded as tests holding
today's actual behaviour, so a `fasta2a` fix shows up as a newly-failing
assertion rather than a silent gap.

## Chapters to come

Written as the work they describe is built and measured:

- **What orchestration costs** — the overhead measures: acceptance latency,
  time to first worker event, context bytes referenced against copied, and
  the tokens orchestration itself adds.

Read more on <https://datalayer.ai/research/context/orchestration-protocols>.
