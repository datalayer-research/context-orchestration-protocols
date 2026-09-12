# How the gaps were found

Part of [Context Orchestration Protocols](./index.md).

A protocol gap report, written from two things that were built rather than
from a reading of the two specifications. Both live in the open-source
[`agent-runtimes`](https://github.com/datalayer/agent-runtimes) repository:

- **The conformance suite** (`agent_runtimes/tests/orchestration`): the nine
  scenarios that are claims about an *adapter* — successful delegation,
  streaming progress, rejection before acceptance, disconnect after
  acceptance, lost and duplicate acknowledgements, crash and recovery,
  cancellation racing completion, and budget exhaustion — each written once
  and run against three registered bindings: A2A, ACP, and an ACP worker that
  does not declare `loadSession`. Every scenario a binding cannot run is
  skipped quoting the adapter's own declared reason, never a protocol name
  written into a test, and each binding's pass rate is reported at the end of
  a run. The other six of the fifteen are claims about the control plane and
  the platform rather than about a protocol, and are proved where they belong
  — [the table is in the durable chapter](./durable-control-plane.md#what-the-conformance-scenarios-are-actually-run-against).
- **The two-protocol example** (`examples/orchestration`): one objective — the
  analysis of a real notebook — delegated to an A2A worker and to an ACP worker
  over one code path, with the agent descriptor as the only difference. Both
  runs complete and produce the same artifact content hash.

Every gap in [the gap table](./gaps.md) was declared by an adapter, skipped by
a scenario, or seen in the example's output. Nothing here is a gap somebody expected to find.

The rule this report is written to: **one gap, one field.** A gap that maps to
no field, or a field that no named scenario needs, is called out as such
rather than kept in the proposed extension because it sounds useful.
