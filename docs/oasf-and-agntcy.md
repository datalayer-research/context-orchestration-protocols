# OASF and AGNTCY, evaluated

Part of [Context Orchestration Protocols](./index.md).

Section 12's Phase 4 asks whether the [`AgentDescriptor`](./strategy.md) maps
to [OASF](https://github.com/agntcy/oasf) records and whether
[AGNTCY](https://github.com/agntcy) directory integration is worth the
coupling — "the answer is written down with the mapping or the reason there
is none" (ORCHESTRATOR.md, O4-02). This is that answer, built from the
schema itself rather than from the marketing description of it: the actual
`record.json`, `locator.json`, `a2a_data.json` and `descriptor.json` object
schemas at
[github.com/agntcy/oasf](https://github.com/agntcy/oasf/tree/main/schema/objects),
read field by field against the descriptor `core/datalayer_core/orchestration/descriptor.py`
already builds from an A2A card, an ACP entry, and a Datalayer agentspec.

## What OASF is

A schema, not a protocol. The **Record** is an OASF document describing one
agentic application — name, version, skills, where to find it — and the
**Directory** (`agntcy/dir`) is a content-addressed store of them: records
are pushed and pulled by content identifier (CID), optionally signed
(`dirctl push record.json --sign`), and found again through a routing and
search layer, self-hosted or against Outshift's hosted directory. Nothing
here competes with A2A, ACP or the extension of §7.2 — a Record can *point
at* an A2A endpoint, the way a library catalogue card points at a book
rather than being one.

## The Record, field by field

| OASF `record` field | Required | `AgentDescriptor` equivalent |
|---|---|---|
| `name` | yes | `agent_id` / `name` — split, where OASF keeps one |
| `version` | yes | `version` |
| `schema_version` | yes | none — OASF's own document version, not an agent's |
| `description` | yes | `description` |
| `authors` | yes | none |
| `created_at` | yes | none |
| `skills` | yes | `skills: list[AgentSkill]`, the human-facing catalogue |
| `domains` | recommended | none on the descriptor (an agentspec's `domain` and `tags` are dropped before the descriptor, not carried this far) |
| `locators` | optional | `endpoints: list[ProtocolEndpoint]` — see below, this is not a match |
| `modules` | recommended | nothing named, but the right shape for everything below |
| `annotations` | optional | none |

Four fields (`name`, `version`, `description`, `skills`) map directly and
losslessly. `authors` and `created_at` are provenance OASF asks every record
to carry and the descriptor has no field for — not a gap in the mapping, a
thing Datalayer records elsewhere (an artifact's provenance, section 5.4)
that would have to be assembled for the OASF document rather than read off
the descriptor.

## `locators` is not where the endpoint goes, and that is not a gap

The load-bearing finding. OASF's `locator.type` is a closed enum:
`unspecified, helm_chart, container_image, package, source_code, binary,
url`. Every one of them is a place to **download** the thing the record
describes — a chart, an image, a package, a tarball. None of them is *ask
this agent to do something*, which is what a `ProtocolEndpoint` is. Reading
`endpoints` into `locators` would be the same mistake section 7.2's report
already named once, mapping a human-facing field onto a machine contract
because the two happen to both be strings — here, mapping *where to fetch
the artifact* onto *where to send it a message* because both happen to be
URLs.

OASF already has the right place for this, and it is not a locator: the
`modules` array, plus an `a2a_data` module OASF ships for exactly this case.
Its `card_data` field is deprecated as of OASF 1.0.0 in favour of
`module.artifact` — an OCI-style [`descriptor`](https://raw.githubusercontent.com/agntcy/oasf/main/schema/objects/descriptor.json)
(`media_type`, `size`, `digest`, `urls`) referencing the card by content hash
rather than inlining it. That is precisely what `to_agent_card()` already
produces (O0-08): the mapping does not need a second representation of an
A2A endpoint, it needs the existing card content-addressed and referenced
from an `a2a_data` module. The same shape would carry an ACP entry once OASF
or a Datalayer extension module names one; `agentspec_communication_protocol_a2a.json`
in the same schema tree suggests the object model expects more than one
protocol module to exist side by side, which is the right assumption for a
descriptor that is already protocol-neutral.

## `capabilities` needs a module of its own

OASF's `skills` array is classified against OASF's own taxonomy — a
different axis from `capabilities`, the closed, dotted vocabulary
`agents.discover` matches on (`notebook.validate`, `data.analyse`, …, 17
entries as of O2-07). This is the identical distinction section 5.1 already
draws between a descriptor's `skills` (human-facing) and `capabilities` (a
scheduling contract), and OASF's `modules` mechanism — "a flexible
mechanism to extend records with additional information in a modular and
composable way" — is exactly the extension point built for a second,
platform-specific classification living beside the standard one. A
`datalayer_capabilities` module naming `AGENT_CAPABILITIES` ids would carry
it without asking OASF to adopt Datalayer's vocabulary as its own taxonomy,
and without asking Datalayer to abandon a closed vocabulary discovery
depends on for OASF's open one.

## What has no OASF home at all

`authentication`, `data_classifications`, `trust_level`, `runtime`, `cost`,
`latency`, `availability` — seven of the descriptor's fourteen fields appear
nowhere in the Record schema's surface. Consistent with how `from_agent_card`
and `from_agentspec` already report what a source format cannot state
(section 5.1) rather than inventing a value, an OASF export would report
these as gaps unless a second Datalayer module were defined to carry them —
which is a real design decision (one module or several; whether cost and
trust belong in the same one) and not one to take unprompted, for the same
reason O4-03's native adapters wait on demand rather than being built ahead
of it.

## Directory integration is a deployment decision, not a schema one

Publishing to `agntcy/dir` needs a directory server — self-hosted, or
Outshift's hosted one — content-addressed storage, and a signing key; none
of that is implied by getting the Record schema right. It is also strictly
optional: a Record is a useful, standalone document (an OASF-shaped export
of the agent catalogue) whether or not anything ever pushes it anywhere.
Coupling the two questions — can a Datalayer agent be *described* as an OASF
record, and should Datalayer *run or depend on* a directory service — would
answer a smaller question by deciding a bigger one it does not need to
settle.

## The answer

The mapping exists and four of eleven top-level fields are exact. The one
real structural gap — where a live endpoint goes — is not a limitation of
OASF, it is a limitation of reading `locators` as if it were the answer; the
schema's own `modules` mechanism, used as OASF intends it, closes it with no
new format for the endpoint itself. Building the export is therefore a
known, bounded piece of work — a `to_oasf_record()` beside `to_agent_card()`,
an `a2a_data` module wrapping the existing card as a content-addressed
artifact, a `datalayer_capabilities` module for the dotted vocabulary, and a
gap report for the seven fields with no OASF home — and *running or
depending on* a directory server is a separate, later decision that the
export does not force. Neither is built now: section 12 frames Phase 4 as
demand-driven, and there is no reported demand for either yet. This
document is what "the reason there is none" looks like when the reason is
"not yet, and here is exactly what it would take."

## Sources

- [Open Agentic Schema Framework](https://docs.agntcy.org/oasf/open-agentic-schema-framework/) — AGNTCY documentation
- [agntcy/oasf](https://github.com/agntcy/oasf) — the schema repository
- [`schema/objects/record.json`](https://raw.githubusercontent.com/agntcy/oasf/main/schema/objects/record.json) — the Record object
- [`schema/objects/locator.json`](https://raw.githubusercontent.com/agntcy/oasf/main/schema/objects/locator.json) — the locator's closed `type` enum
- [`schema/objects/a2a_data.json`](https://raw.githubusercontent.com/agntcy/oasf/main/schema/objects/a2a_data.json) — the A2A card module, `card_data` deprecated in favour of `module.artifact`
- [`schema/objects/descriptor.json`](https://raw.githubusercontent.com/agntcy/oasf/main/schema/objects/descriptor.json) — the OCI-style content-addressed artifact reference
- [agntcy/dir](https://github.com/agntcy/dir) and its [CLI reference](https://docs.agntcy.org/dir/directory-cli/) — push, pull, sign, search
- [Hosted Outshift Agent Directory](https://docs.agntcy.org/dir/hosted-agent-directory/)
