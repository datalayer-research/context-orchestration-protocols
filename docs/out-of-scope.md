# Fields the proposal started with that no scenario needs

Part of [Context Orchestration Protocols](./index.md).

Three of the nine items in the extension's original scope do not survive the
rule, and they are the useful part of this report.

**Pause and resume.** Both adapters refuse both, and none of the fifteen
scenarios pauses anything. An execution can be held on the orchestrator's side
by not dispatching the next attempt, which covers a paused *execution*; what
needs a protocol is pausing a worker that is already mid-turn, and nothing in
the suite asks for that. **Out of the first version**, and back in scope the
day durable checkpoints produce the scenario that shows whether a mid-turn
pause is needed at all.

**Trace propagation.** It is not missing from A2A. The W3C traceparent rides
on the HTTP request headers and the `httpx` instrumentation carries it without
anyone's help, which is exactly how the example's artifacts came out carrying
the tree's trace id. It *is* missing from ACP, where a WebSocket has no
per-message header and the adapter can only stamp the connection's handshake —
so a trace per turn is not expressible there. That makes it a question for
ACP's own evolution, not a field in an A2A extension. **Removed** from the A2A
scope.

**Worker release, or `terminate`.** Both adapters refuse it, and for the same
reason: `tasks/cancel` and a cancelled turn stop *work*, and neither protocol
has a notion of who owns a worker or of handing one back. No scenario covers
it, and our own design still has the question open — whether terminating an
execution may release a worker another execution is attached to, or only one
this tree created. A field defined before that question is answered would be
a field defined against a guess. **Out of the first version**, gated on that
decision.
