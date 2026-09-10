# What a plain worker still gets

Part of [Context Orchestration Protocols](./index.md).

Every field above is optional and capability-negotiated, and the reduction is
reported on the execution rather than hidden. A worker that has never heard of
Datalayer keeps: created, assigned, running, completed, failed and cancelled;
artifacts; and cancellation. It does not get an acceptance acknowledgement
beyond its transport's, a checkpoint, steering, a pause, or a lease beyond
polling. The two-protocol example is the demonstration that this is enough to
be useful: two workers, neither of which implements any extension field,
completed the same objective over two protocols and agreed on the answer to
the byte.
