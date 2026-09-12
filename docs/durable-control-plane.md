# A durable control plane

Part of [Context Orchestration Protocols](./index.md).

What it took to make a delegation outlive the process that dispatched it.
Everything below was found by building it, and several of the findings are
corrections to what we assumed when [the strategy](./strategy.md) was written.

## The shape

An execution is a durable workflow. The control plane owns commands, canonical
state, the store and the event stream; it does **not** dispatch. Dispatch
happens inside the durable worker, which imports the adapters and speaks A2A
and ACP in-process. That split was not the original design — the first sketch
had the control plane dispatching and the workflow calling it over HTTP — and
it changed for one reason: a durable step that hands off to a request handler
cannot be replayed safely, because the handler has already done the thing.

The consequence to design for is that a **worker process, not a request,
holds the protocol connections** — an A2A HTTP client and an ACP WebSocket —
across a step boundary and a pod roll.

## Six findings

**1. An ending nobody saw is unknown, not failed.** The first implementation
failed an execution whose lease expired. That is wrong, and section 6.4 of the
plan says so in one line that took a rewrite to understand: a control plane
that fails an execution because *its own* connection dropped will retry work
that is still running. The attempt is now recorded as unknown, the worker is
asked again through the handle the attempt recorded — an A2A `tasks/get`, an
ACP session — and whatever the task actually came to is recorded. Only work
nobody can find again is retried.

**2. The intent record is the whole mechanism.** A step that is not idempotent
has to write down that it is about to run, *before* it runs, and a durable
engine has to record that write. `claim_dispatch` records the attempt and the
claiming process; `dispatch` then sends the objective only from that process,
re-attaches from any other, and with no handle records a lost lease rather
than sending twice. Without it, a worker that died mid-dispatch delegated the
same work twice on recovery.

**3. One run per attempt.** A retry cannot be another step in the same
workflow run, because a replayed step has to be able to find the successor it
already started rather than start a second. Attempt *n* gets its own run keyed
`<execution>:attempt-<n>`, which is its identity on both engines; commands
walk the keys from the newest attempt down. The alternative — one long run
with a loop — replays the loop.

**4. Idempotency is a claim, not a lookup.** Checking whether a key exists and
then creating is two operations and a race. The key is claimed atomically in
the index (`_version_: -1`, "create only if absent"); a repeated key returns
the first execution, and the same key behind *different* work is refused
rather than silently answered with the wrong execution.

**5. The engine is a value, and that has to be tested rather than asserted.**
`none`, `dbos` and `temporal` answer the same six questions, and nothing above
the engine boundary knows which answered. What keeps that true is a scenario
suite that runs on all three — including the deliberately boring in-process
one, which is what catches a step that blocks the event loop before it reaches
a real deployment.

**6. A blocking call is a deployment bug.** A step that blocks the event loop
breaks the served in-process engine and the Temporal worker in the same way
and for the same reason. Every blocking call goes off the loop, and the suite
that proves it is the one that runs on all three engines.

## What the conformance scenarios are actually run against

Correcting [the method](./method.md), which described one suite: the fifteen
scenarios are proved in three places, because they are claims about three
different things.

| Scenarios | Suite | What it holds constant |
|---|---|---|
| 1–7, 9, 11 | `agent_runtimes/tests/orchestration`, run against three bindings (A2A, ACP, and an ACP worker without `loadSession`) | The **adapter** contract: the same scenario over every protocol, with nothing in the test naming one |
| 14 | `ai-agents` control-plane tests | The **command boundary**: a refusal that names the limit |
| 10, 12, 13, 15 | `services/durable` tests | The **platform**: approvals, context grants, artifact commits and mixed-protocol trees, with a real engine under them |

The adapter suite reports each binding's rate at the end of a run, because
the useful number is not how many scenarios passed but what share of the ones
a binding *could* run it passed — a scenario a protocol cannot express is
skipped quoting the adapter's own words, never silently.

## What is still open

Recovery is proved against the in-process and fake engines and in the
scenario suites. The remaining half is a worker killed mid-dispatch on a
real deployment, which is a deployment exercise rather than a design
question — and the honest statement is that until it has been run, the claim
is that the mechanism is right, not that it has survived production.
