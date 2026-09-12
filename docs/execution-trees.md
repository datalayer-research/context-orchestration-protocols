# Execution trees

Part of [Context Orchestration Protocols](./index.md).

What changes when a worker becomes an orchestrator. The single-level case is
mostly a durability problem; the tree is mostly an **authority** problem.

## Children are requested, not taken

A worker asks the control plane to create a child; the control plane resolves
the worker, authorises the context and dispatches. A worker never dispatches
a child itself.

This looks like ceremony until you write down what a worker is. Section 9 of
the plan treats remote workers, protocol metadata, artifacts and requested
actions as untrusted — and a worker that can dispatch can escape its own
depth limit, its own budget and its own context scope, because all three are
enforced where the dispatch is decided. The `delegate_task` tool that agents
already had became a *request*, and the existing subagent examples run
unchanged through the new path, which is the evidence that the ceremony costs
the agent author nothing.

## Authority is split, and the split is the finding

The original design had one enforcement point: identity would issue and
validate capability-scoped delegation tokens, so a compromised orchestrator
could not widen a child's authority. That is the strongest position and it
was the wrong place to put all of it — it requires the identity service to
carry a model of orchestration (depth, capability, fan-out) that means
nothing to anything else it does.

What shipped is a split, and it holds because each side checks only what it
already understands:

- **Identity enforces resource scope.** The child's credential is the
  execution's own short-lived key, narrowed to the subset of the context
  manifest the child may see. No model of orchestration is needed to check it.
- **The control plane enforces orchestration limits** — remaining depth,
  permitted capabilities, remaining budget — at the command boundary, because
  those are its own concepts.

A child can widen neither, and the test that matters is that neither refusal
path can be satisfied by the other service.

## What the tree costs, and who pays

Two budgets that look like one and are not:

- The **model budget** — tokens, requests, cost per run — already folds a
  delegated run's usage into its parent's. The control plane sets its limits
  from the execution's policy rather than reimplementing them.
- The **platform budget** is a reservation and its credits, which is what
  actually meters compute. A tree mints one reservation on the root and
  children draw against it.

Exhausting either cancels the subtree, and the two record *different*
reasons — because "you ran out of tokens" and "you ran out of credits" send
a person to two different places.

Spend per node had to come from the workers, and nothing was reporting it:
a budget's cost is a limit, not a measurement, and credits are held per tree
rather than per node. Workers now answer each turn with the tokens they used
and the cost of their model where they can price it, and the control plane
records it on the attempt the turn ended. Without that, a tree could say what
it was allowed to spend and not what it spent.

## Retry is a rerun

A finished execution cannot move again — terminal states are absorbing, which
is exactly what makes a cancel racing a completion safe. So "run this node
again" cannot be a transition. It re-delegates the finished node's agent,
objective, context and policy as a **new execution**: under its parent while
the parent still runs, and as a new root otherwise.

This is the sort of thing a lifecycle diagram tells you if you let it. The
first design had a retry edge out of `completed`, and every question about it
— what happens to the first result, which artifact is the answer, what a
subscriber sees — dissolved once the edge was removed.

## Mixed protocols under one parent

The scenario that justifies the whole canonical-model argument is an A2A
child and an ACP child under one parent, producing one trace and one
consolidated artifact set. It passes. Nothing in the parent knows which
protocol either child speaks; the only protocol-specific line anywhere is a
dictionary from a protocol to its adapter.

Gap 9 in [the gap table](./gaps.md) was recorded as *predicted, not
observed*, because at the time both the suite and the example were single
level. It is now observed: parent, root and child relationships are not
expressible in either protocol, and the tree is held entirely in the control
plane's own records. A worker in a Datalayer tree does not know it is in one.

## Teams are trees

The team specification the platform already had — members with roles and
dependencies, a supervisor, a delegation depth, a context-sharing mode —
turned out to describe an execution tree without naming one. Execution mode
and member dependencies give the schedule, the supervisor is the root
execution, and the context mode maps onto the manifest's shared and private
flags. No new concept was needed, which is the most useful thing that can
happen to a design.
