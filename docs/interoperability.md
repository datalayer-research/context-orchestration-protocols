# Interoperability

Part of [Context Orchestration Protocols](./index.md).

Traces of a standard A2A client against [the reference worker](./reference-agents.md),
and of Datalayer's own dispatch pattern against a worker that has never
heard of this extension — both directions
[section 19.8](https://github.com/datalayer/ui/blob/main/plans/ORCHESTRATOR.md)
calls the reduced-guarantees claim, run against
[`a2a-sdk`](https://pypi.org/project/a2a-sdk/) (pinned `0.3.26`): the A2A
project's own reference implementation, independent of `fasta2a`
([ORCHESTRATOR.md O3-05](https://github.com/datalayer/ui/blob/main/plans/ORCHESTRATOR.md)).
The suite that produced every trace here is
`tests/test_standard_client_compatibility.py` in
[`agent-teams`](https://github.com/datalayer/agent-teams), run in CI on
every push.

## A standard client, against the reference worker

`a2a-sdk`'s own client code — not anything Datalayer wrote — against
[`agent_teams.a2a.reference_worker`](./reference-agents.md). This is where
looking for a trace found two real gaps, neither expected, both between
what `fasta2a` actually serves and what the current A2A specification (and
`a2a-sdk`, which implements it) requires. Not `agent-teams`' bug to fix —
its reference worker hands `fasta2a`'s `FastA2A` nothing about card or
response shape at all — and not particular to the reference worker either:
`agent-runtimes`' production A2A route is built on the same `FastA2A`, so
both apply to every Datalayer A2A worker actually running today.

### Gap 1 — the served card has no `url`

The current specification requires a top-level `url`; `fasta2a` writes only
`supportedInterfaces`. Already found once, by validating a hand-built card
against `a2a-sdk`'s own `AgentCard` model (`agent_runtimes`'
[O4-01](https://github.com/datalayer/ui/blob/main/plans/ORCHESTRATOR.md));
reproduced here for real, live, over the wire — `a2a-sdk`'s own card
resolver, against a running worker, refuses to even start:

```text
GET /.well-known/agent-card.json →

{
  "name": "datalayer-orchestration-reference-worker",
  "supportedInterfaces": [
    { "protocolBinding": "JSONRPC", "url": "http://localhost:8000", "protocolVersion": "1.0" }
  ],
  "capabilities": {
    "extensions": [
      { "uri": "https://datalayer.ai/extensions/orchestration/v1", "required": false }
    ]
  }
  # no top-level "url"
}

A2ACardResolver.get_agent_card() raises A2AClientJSONError:
  JSON Error: Failed to validate agent card structure: [{"type":"missing",
  "loc":["url"],"msg":"Field required", ...}]
```

### Gap 2 — `message/send`'s result is wrapped where the spec wants it bare

Found looking past gap 1 (by handing the client a card patched with a
`url`, to see what happened next): `fasta2a.schema.SendMessageResult` and
`StreamResponse` (the first event of `message/stream`) are `{task: Task}` /
`{message: Message}` — TypedDicts with named, optional fields — where the
specification's own `SendMessageSuccessResponse.result: Task | Message`
wants the bare object, no wrapper. This is not a corner case: it is the
very first response of *any* exchange, and both of `a2a-sdk`'s client
implementations — the deprecated `A2AClient` and its `ClientFactory`
replacement — share the one JSON-RPC transport this lives in, and fail
identically:

```text
POST / → message/send →

{"jsonrpc": "2.0", "id": "...", "result": {"task": {"id": "...", ...}}}
                                            ^^^^^^ the spec wants this bare

A2AClient.send_message() raises pydantic.ValidationError (7 errors):
  SendMessageSuccessResponse.result.Task.contextId
    Field required [input_value={'task': {'id': '0a78ad67...'}}]
  SendMessageSuccessResponse.result.Task.id
    Field required [input_value={'task': {'id': '0a78ad67...'}}]
  ... (5 more, one per required Task/Message field, all missing because
      pydantic is validating the wrapper object, not what is inside it)
```

**No specification-conformant A2A client can complete `message/send`
against a `fasta2a`-based worker today.** Both gaps are recorded as tests
holding today's actual, non-conformant behaviour
(`pytest.raises` against the real error) rather than fixed in
`agent-teams` — the fix belongs in `fasta2a` itself.

### What still interoperates, despite both gaps

`a2a-sdk`'s own request models build a fully standard request — proving the
request side is already conformant — sent over raw HTTP because gap 2 means
no A2A client library can parse the reply; the reply read by hand
(`result["task"]`, not `result`) to check the worker actually answered it,
correctly, extension fields included:

```text
POST / → message/send, header A2A-Extensions: https://datalayer.ai/extensions/orchestration/v1

{"jsonrpc": "2.0", "id": "...", "method": "message/send",
 "params": {"message": {"kind": "message", "role": "user",
   "parts": [{"kind": "text", "text": "What is the capital of France?"}]}}}

GET tasks/get, polled to a terminal state →

{
  "status": { "state": "completed" },
  "history": [
    { "role": "user", "parts": [{"text": "What is the capital of France?"}] },
    { "role": "agent", "parts": [{"text": "echo: What is the capital of France?"}],
      "metadata": { "datalayer": { "usage": {
        "currency": "USD", "inputTokens": 30, "outputTokens": 36
      } } } }
  ]
}
```

The worker answered, correctly, with what it spent — once a client gets
past the two gaps above.

## Datalayer, against a standard worker

The reverse direction: the same hand-built JSON-RPC dispatch pattern the
reference worker's own conformance suite uses, against a worker built
entirely on `a2a-sdk`'s own server framework
(`a2a.server.agent_execution.AgentExecutor`) — independent of `fasta2a` on
*both* sides of this direction, standing in for any A2A implementation
Datalayer has never heard of.

```text
POST / → message/send, header A2A-Extensions: https://datalayer.ai/extensions/orchestration/v1

{"params": {"message": {"parts": [{"text": "hello from Datalayer"}],
  "metadata": {"datalayer": {
    "execution": {"executionId": "exec_1", "rootExecutionId": "exec_1", "depth": 0},
    "budget": {"outputTokens": 4000},
    "checkpoint": {"checkpointId": "ckpt-nobody-here-reads"}
  }}}}}

→ (this worker's DefaultRequestHandler waits for the executor; no polling needed)

{
  "result": {
    "status": {
      "state": "completed",
      "message": {
        "parts": [{"text": "echo: hello from Datalayer"}]
        # no "metadata" key at all — not stripped, not read, not
        # acknowledged: this worker has never heard of "datalayer"
      }
    }
  }
}
```

The delegation completes. The execution reference, the budget, the
checkpoint — all sent, none of it read, written to, or choked on. This is
the reduced-guarantees claim itself, not an assertion about it: a worker
that implements none of the extension still runs, completes, and answers,
exactly as [the plain-worker chapter](./plain-workers.md) says it must.

## Reading these yourself

```bash
git clone https://github.com/datalayer/agent-teams
cd agent-teams
pip install -e ".[a2a,test]"
pytest tests/test_standard_client_compatibility.py -v
```

Four tests, one per section above (the two gaps split into their own,
plus one for each direction's positive claim); `.github/workflows/py-tests.yml`
runs them on every push.
