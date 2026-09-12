# The strategy

Part of [Context Orchestration Protocols](./index.md).

One agent hands work to others. The question this work exists to answer is
not *how do agents talk* — several protocols already answer that — but
**who is responsible for the work while it runs**, and what has to be true
for that responsibility to survive a disconnect, a crash, a redeploy and a
retry.

## Four claims

**1. Orchestration is not a transport.** The execution model must not depend
on A2A, ACP, HTTP, WebSocket or stdio. What an orchestrator owns is an
execution: an identity, a parent, a lifecycle, attempts, a budget, a context
manifest and artifacts. None of that is protocol-shaped, and a design that
put it on the wire would have to be redesigned for the next protocol. The
canonical model is therefore written once, and each protocol is an adapter
that translates commands down and observations up — reporting explicitly
what it cannot do rather than answering as if it had.

**2. Use the protocols that exist.** A2A is the durable remote binding, ACP
the interactive one for IDEs and coding agents, MCP the tool and context
binding. The gap report is the evidence for the only thing we propose adding:
[nine fields](./gaps.md) A2A cannot express about delegated work. Nobody
needs another general agent communication protocol, and proposing one would
have been the fastest way to be ignored.

**3. The platform is authoritative, not the session.** A protocol session and
a remote task are *projections* of a durable execution, never the source of
truth. This is the claim with the most consequences: it means an open
WebSocket, a live process or an SDK object can never be the thing that knows
what is happening, and everything about recovery follows from taking it
seriously. See [the durable control plane](./durable-control-plane.md).

**4. Reference context, do not copy it.** A worker is handed a notebook by
name and version — `datalayer:notebook/<uid>@<version>` — with a
short-lived credential scoped to exactly that, not a copy of the bytes. The
measure that matters is context bytes referenced against bytes copied, and
the security property falls out of the same decision: a worker that was given
one notebook can reach one notebook.

## What we are not trying to do

A general peer-to-peer agent network; a replacement for A2A, ACP or MCP;
standardised model-internal memory; automatic discovery across untrusted
public networks; consensus between autonomous agents. The first optimisation
target is deliberately narrow — **one orchestrator managing a tree of
workers** — because that is the shape almost every real delegation has, and
because a mesh is much harder to make recoverable, auditable or affordable.

## Why an extension rather than a protocol

Every field in the [gap table](./gaps.md) was declared missing by an adapter,
skipped by a conformance scenario, or seen missing in a working two-protocol
run. Three fields the scope started with were [removed](./out-of-scope.md)
because no scenario needed them. That rule — *one gap, one field, and a
named scenario or it goes* — is the whole difference between an extension a
standards body might take and a wish list.

Keeping every field optional and capability-negotiated is the other half.
A plain A2A worker that implements none of it still runs, with
[reduced guarantees that are reported rather than hidden](./plain-workers.md).
An extension that a worker had to implement to participate would be a
protocol wearing a smaller word.

## How it is being built

In order, each phase ending in something demonstrable rather than a
milestone:

1. **The canonical model and two adapters** — one scenario running over A2A
   and ACP with no orchestration logic that knows which. Done; the evidence
   is [the method](./method.md) and the gap report.
2. **A durable control plane** — executions that outlive the process that
   dispatched them. Done; see [the chapter](./durable-control-plane.md).
3. **Execution trees** — children requested rather than taken, authority
   narrowing at every level. Done; see [the chapter](./execution-trees.md).
4. **The extension, published** — specification, reference agents, a public
   conformance suite, and interoperability traces rather than a proposal.
5. **Ecosystem** — descriptor mappings, directory integration, and native
   framework adapters where A2A and ACP genuinely cannot reach.

Phases 1 to 3 are built and running. Phase 4 is what the gap report exists
to feed.
