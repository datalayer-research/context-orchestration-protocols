[![Datalayer](https://assets.datalayer.tech/datalayer-25.svg)](https://datalayer.ai)

[![Become a Sponsor](https://img.shields.io/static/v1?label=Become%20a%20Sponsor&message=%E2%9D%A4&logo=GitHub&style=flat&color=1ABC9C)](https://github.com/sponsors/datalayer)

# ⚗️ 📡 Context Orchestration Protocols

Read more on https://datalayer.ai/research/context/orchestration-protocols.

The documents are in [`docs/`](./docs/index.md), starting from its index:

- [The strategy](./docs/strategy.md): why orchestration is modelled apart
  from any transport, why the proposal is an extension to A2A rather than
  another protocol, what is deliberately out of scope, and the order it is
  being built in.
- [What A2A and ACP could not express](./docs/index.md#what-a2a-and-acp-could-not-express):
  the protocol gap report, written from a conformance suite and a working
  two-protocol delegation rather than from a reading of the two
  specifications — [how the gaps were found](./docs/method.md),
  [the gaps and their fields](./docs/gaps.md),
  [the fields no scenario needs](./docs/out-of-scope.md),
  [two things that are not gaps](./docs/not-gaps.md), and
  [what a plain worker still gets](./docs/plain-workers.md).
- [The extension, specified](./docs/extension-v1.md): the normative
  document — URI, negotiation on both protocols, the envelope, every field,
  and what is specified but not yet on the wire.
- [A durable control plane](./docs/durable-control-plane.md): what it took
  to make a delegation outlive the process that dispatched it, and where the
  fifteen conformance scenarios are actually proved.
- [Execution trees](./docs/execution-trees.md): what changes when a worker
  becomes an orchestrator — authority, budgets, retry, and mixed-protocol
  children under one parent.
