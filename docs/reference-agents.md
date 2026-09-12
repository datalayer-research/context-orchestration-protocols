# Reference agents, and a public conformance suite

Part of [Context Orchestration Protocols](./index.md).

A worker implementing the extension, and the tests anybody can run against
their own, with no Datalayer service anywhere in the process
([ORCHESTRATOR.md O3-04](https://github.com/datalayer/ui/blob/main/plans/ORCHESTRATOR.md)).
Both live in the open-source
[`agent-teams`](https://github.com/datalayer/agent-teams) repository,
published to PyPI as `agent-teams`.

## The worker

`agent_teams.a2a.reference_worker` is the smallest thing that speaks the
extension correctly: no model, no framework beyond
[`fasta2a`](https://github.com/datalayer/fasta2a) — which Datalayer already
contributes to — no Datalayer control plane. Run it standalone —

```bash
python -m agent_teams.a2a.reference_worker  # serves on :8000
```

— or in-process, as an ASGI app with no socket at all
(`create_reference_app()`), which is how its own test suite drives it.

What it does, deterministically and for free — a character count, not a
model call, so running it needs no API key:

- Advertises the extension on its agent card.
- Answers a turn with what it "spent" (`datalayer.usage`).
- Declines a delegation whose budget already reads `outputTokens: 0`,
  naming the limit (`datalayer.error`) — the one budget scenario a worker
  with no model can still demonstrate honestly.
- Answers a `pause` request by ending the turn at a checkpoint it makes up
  on the spot, and echoes a `checkpoint` a delegation resumes from back in
  its answer, so a caller can tell the resume actually happened.

## The conformance suite

`tests/test_reference_worker_conformance.py` drives the worker over real
ASGI HTTP (`httpx.ASGITransport`, no socket) with hand-built JSON-RPC
requests matching [the normative wire shapes](./extension-v1.md) directly —
not through this package's own client helpers, so what is asserted is the
protocol on the wire, not two pieces of code agreeing with themselves.

Six scenarios, run and passing:

- **Negotiation** — the card advertises the extension; a client that never
  activates it (no `A2A-Extensions` header) gets a plain answer, no
  `datalayer` key at all.
- **Usage** — an activated client learns what the turn spent.
- **Budget** — a delegation whose budget already reads zero is declined,
  naming the limit.
- **Pause and resume** — a `pause` request ends the turn at a checkpoint; a
  later delegation naming that checkpoint is told it resumed.

A trace of each, captured from a real run — the actual JSON-RPC exchange,
not a hypothetical one — is in [the interoperability chapter](./interoperability.md),
alongside what a genuinely independent client made of the same worker.

## Reading the traces yourself

Nothing above is asserted from a distance. Clone `agent-teams`, install the
`test` extra, and run:

```bash
pytest tests/test_reference_worker_conformance.py -v
```

Every scenario named above is one test; a failure names exactly which
claim in this document stopped being true.
