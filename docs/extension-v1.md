# The Datalayer orchestration extension, v1

Part of [Context Orchestration Protocols](./index.md).

A versioned, optional extension that lets a worker and an orchestrator say the
things about *delegated work* that A2A and ACP leave unsaid. Every field here
closes a gap that was [found by building](./method.md), not one that seemed
useful; the gaps and the scenarios that need them are
[the gap table](./gaps.md).

```text
https://datalayer.ai/extensions/orchestration/v1
```

**Nothing in this extension is required.** A worker that implements none of it
runs, completes, returns artifacts and can be cancelled — see
[what a plain worker still gets](./plain-workers.md). An extension a worker
had to implement in order to participate would be a protocol wearing a
smaller word.

## Negotiation

A worker advertises the extension; an orchestrator sends extension fields only
to a worker that advertised it. Neither side infers support from a successful
message.

**A2A** — as an extension entry on the agent card:

```json
{ "capabilities": { "extensions": [ { "uri": "https://datalayer.ai/extensions/orchestration/v1" } ] } }
```

**ACP** — in the `initialize` result, under the agent's capabilities:

```json
{ "agentCapabilities": { "_meta": { "datalayer": { "extensions": [ "https://datalayer.ai/extensions/orchestration/v1" ] } } } }
```

## The envelope

One object, under the key `datalayer`, carried in the place each protocol
already provides for it — an A2A message's `metadata`, an ACP prompt's or
result's `_meta`. One key rather than nine keeps a plain worker's message
untouched and makes the extension trivially strippable by a proxy.

```json
{ "metadata": { "datalayer": { "execution": { "...": "..." }, "budget": { "...": "..." } } } }
```

## Fields an orchestrator sends

| Field | Shape | What it closes |
|---|---|---|
| `execution` | `{executionId, rootExecutionId, parentExecutionId?, depth, accountUid?}` | A2A's `contextId` groups tasks without saying which is whose parent or how deep, and ACP's `session/fork` is optional and unstable. This is also the scope a worker keeps its checkpoints under, and the account a worker's request for a child is made in. |
| `budget` | `{inputTokens?, outputTokens?, cost?, currency}` | A worker cannot otherwise be told its limits, so exhaustion is detected by the orchestrator *after* the spend rather than declined by the worker *before* it. |
| `credential` | `string` | The execution's own short-lived token, scoped to exactly its context manifest. A bearer credential: taken out of the message as it arrives, held for the run, and never written into task state. What the stored message keeps is that one was sent (`"withheld"`). |
| `checkpoint` | `{checkpointId}` | Resume: this delegation continues the work the named checkpoint holds, rather than starting it again. |
| `pause` | `{}` (presence) | Ask the worker to stop at a checkpoint it can be resumed from. Neither protocol has a pause — A2A cancels, ACP cancels. |

## Fields a worker sends

| Field | Shape | What it closes |
|---|---|---|
| `usage` | `{inputTokens?, outputTokens?, cost?, currency}` | What the turn spent. Nothing else records it: a budget's `cost` is a limit, not a measurement, and platform credits are held per tree rather than per node. `cost` is present only when the worker could price its own model. |
| `paused` | `{checkpointId}` | The checkpoint the worker actually stopped at, which a later delegation names to resume. |
| `error` | `{code, limit}` | A refusal the worker owns — `budget_exhausted` naming the limit it hit — as distinct from a worker that broke. Nothing another attempt could spend differently. |

## Methods

| Method | Direction | What it closes |
|---|---|---|
| `_datalayer/steer` | orchestrator → worker | A2A has no way to add instructions to a task already running; a plain A2A worker gets a new task or nothing. ACP can prompt the same session again, so this asymmetry is A2A's alone — the method is defined once so an orchestrator does not branch on protocol. The instruction is added to the worker's run before its next model request. |

## Proposed for v1, not yet on the wire

Specified because a named scenario needs them, and honestly marked because
the reference implementation does not send them yet. An implementer should
not expect them.

| Field | Shape | Why it is not implemented |
|---|---|---|
| `acknowledgement` | `{kind, attemptId, at}` | The adapters currently infer acceptance from each protocol's own states, which is exactly the inference the field exists to remove. |
| `attempt` | `{attemptId, number, leaseExpiresAt}` | A worker still cannot tell a retry from a duplicate delivery; the orchestrator tells them apart in its own store. |
| `contextManifest` | `[{uri, access, materialization, sharing, required}]` | The manifest reaches both bindings rendered as prose, so nothing on the wire says read-only, snapshot or private. |
| `artifactProvenance` | `{sourceReferences, contentHash, producedAt}` | Hashes and attributions are computed orchestrator-side, so what the execution knows about an output is what it inferred rather than what the worker claimed. |
| `approvalRequest` | `{toolCall, options, expiresAt}` | ACP has `session/request_permission` and it is used; A2A's `input-required` says a task waits without saying what for. The field would give A2A the ACP shape. |

## Deliberately out of scope

- **Trace propagation.** Not missing from A2A — `traceparent` rides on the
  HTTP headers. It is missing from ACP, where a WebSocket has no per-message
  header, and that is an ACP problem rather than an A2A extension field.
- **Worker release.** Cancel stops work; neither protocol has a notion of who
  *owns* a worker, and whether one execution may release a worker another is
  attached to is still an open question. A field defined before that answer
  is a field defined against a guess.
- **`collect`.** Not a gap. Artifacts are read from the execution's own
  store; fetching them back from a worker would be asking an untrusted party
  what it produced.
- **ACP `loadSession`.** ACP working as designed. An extension does not fix
  another protocol's optional capability.

## The schema

A machine-readable copy of the envelope above is
[`extension-v1.schema.json`](./extension-v1.schema.json) — JSON Schema
2020-12, describing only what is actually sent. It is **generated** from the
constants that are the contract, not written beside them:

```bash
python -c "import json; from agent_runtimes.orchestration.extension_schema \
  import extension_schema; print(json.dumps(extension_schema(), indent=2))"
```

`agent_runtimes/tests/test_orchestration_extension_schema.py` holds it to a
real delegation — one built by the same `delegation_meta` the adapters call,
and the answers a worker actually sends — so a field added to the envelope
without being described fails there rather than in somebody's implementation.
The envelope is `additionalProperties: false` for the same reason: a new field
has to be added deliberately.

## Conformance

An implementation is conformant when it passes the orchestration conformance
scenarios for the operations it advertises, and when every operation it does
*not* advertise is refused with a reason rather than answered as though it had
succeeded. The suite is in
[`agent-runtimes`](https://github.com/datalayer/agent-runtimes)
(`agent_runtimes/tests/orchestration`) and runs against any registered
binding; a scenario a binding cannot run is skipped quoting that binding's own
declared reason, and each binding's pass rate is reported at the end of a run.

The rule the extension is maintained to: **one gap, one field, and a named
scenario or it goes.** Three fields the scope started with were removed under
it, which is the reason to trust the nine that stayed.
