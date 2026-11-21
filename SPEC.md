# Network events: specification

Status: **agreed, not implemented** (2026-11-21). This file records what was decided so work can resume later. The schema in `schema/event.schema.json` and the files in `examples/` are final drafts and have been checked against each other; nothing else is built yet.

## 1. What a network event is

A dated fact about 42 Chain or Infinite Drive: a software release, a network upgrade, an incident, a security notice, the result of a governance proposal with technical effect, or maintenance. Each event is one JSON file that follows the schema.

- **Producers:** mainly automated processes (release publishing, governance watching, network monitoring). People too, using exactly the same format.
- **Where it is shown:** in the Console, under *Network events* (`/console/events`), with one permanent page per event.
- **What it is not:** campaigns, opinions, third-party announcements or "we decided" statements. Only facts, with their source.

### Relation to the chain

The chain is the source of truth for anything that happens on it (proposals, votes, upgrade plans, transactions). This repository does not replace it: it tells those facts in readable form, adds context (why, what to do), and records facts that never touch the chain (releases, RPC incidents, maintenance). An event about an on-chain fact always cites it in `refs`. If the two ever disagree, the chain wins.

## 2. Decisions

| Topic | Decision |
| --- | --- |
| Language | **English only.** The record is the source of information; translations, if any, are an interface concern and never live here. |
| Format | One JSON file per event, validated against JSON Schema draft 2020-12. |
| Publishing | **Always through a pull request,** including automated processes. Maintainers review and merge. Nobody pushes to `main` directly. |
| Append-only | Events are never rewritten or deleted. They evolve by appending to `updates`; a mistake is corrected with an update that explains it; an event that turned out false becomes `cancelled`. |
| Relations | Written once, in the newer event, pointing to the older one. The site computes the inverse links at build time, so old events never need editing. |
| Site-wide banner | Shown for `warning` or `critical` events that are `in-progress`, or `scheduled` within the next 7 days. |

## 3. Repository layout (target)

```
schema/event.schema.json        the schema
events/<year>/<id>.json         the events (empty until launch)
examples/                       illustrative events, never published
scripts/validate.mjs            validation (to build)
.github/workflows/validate.yml  CI (to build)
README.md                       how to write an event
SPEC.md                         this file
```

The file name is the event `id` plus `.json`, inside the folder of its year.

## 4. Fields

| Field | Type | Required | Purpose |
| --- | --- | --- | --- |
| `id` | lowercase, digits, hyphens; starts with `YYYY-MM-DD-` | yes | Unique, permanent; it is the event's URL |
| `type` | enum | yes | What kind of fact it is |
| `network` | enum | yes | Which network it affects |
| `severity` | enum | yes | How much it matters to users |
| `status` | enum | yes | Current state (the last one in `updates`, if any) |
| `startsAt` | UTC date-time | yes | When it happens or starts |
| `endsAt` | UTC date-time | no | When it ended or is expected to end |
| `title` | string, max 100 | yes | One-line headline |
| `summary` | string, max 400 | yes | What happens and what the reader should do, if anything |
| `context` | object | no | The why: `problem`, `cause`, `solution`, `decision`, `plan` (strings, max 1500 each) |
| `components` | list of enums | no | Which parts it touches |
| `refs` | object | no | Verifiable data: `version`, `height`, `chainId`, `proposals[]`, `txs[]`, `addresses[]` (each `{address, label}`) |
| `relations` | list | no | Links to other events: `{event, kind, note?}` |
| `links` | list | no | Sources: `{label, url}` (https only) |
| `source` | object | yes | `{kind: "process", name}` or `{kind: "manual"}` |
| `updates` | list | no | Append-only history: `{at, status, note}` |

### Allowed values

- `type`: `release`, `upgrade`, `incident`, `security`, `governance`, `maintenance`, `notice`
- `network`: `mainnet`, `testnet`, `all`
- `severity`: `info`, `warning`, `critical`
- `status`: `scheduled`, `in-progress`, `resolved` (incidents), `completed`, `cancelled`
- `components`: `node`, `consensus`, `evm`, `ibc`, `governance`, `rpc`, `explorer`, `console`, `docs`
- `relations[].kind`:

| kind | This event… | Shown on the other one as |
| --- | --- | --- |
| `implements` | carries out what the other decided or planned | "Carried out by…" |
| `follows` | is the next step of the other | "Followed by…" |
| `explains` | explains or gives context to the other | "Explained in…" |
| `supersedes` | replaces the other (which becomes historical) | "Superseded by…" |
| `corrects` | corrects a mistake in the other | "Corrected by…" |
| `relates` | is related, nothing more | "Related…" |

## 5. Rules

1. The `id` never changes and matches the file name.
2. All dates are UTC with an explicit `Z`. The site shows them in the reader's local time.
3. Plain text only: no HTML, no Markdown. Links go in `links`, never inside text.
4. Verifiable data goes in `refs`, not in text, so the site can link it to the explorer or release notes.
5. A process always identifies itself with `source.name`.
6. Only facts, with a public source in `links` whenever one exists.

## 6. Validation (to build)

`scripts/validate.mjs`, run in CI on every pull request:

1. Every file under `events/` validates against the schema.
2. The `id` matches the file name and the year folder.
3. No duplicate `id`s.
4. Every `relations[].event` exists, and relations form no cycles.
5. Append-only: compared with `main`, existing files may only gain entries at the end of `updates` and a matching `status` / `endsAt`; nothing else may change, and no file may be deleted.
6. `status` equals the last `updates[].status`, if there are updates.
7. Build the site with this repository's changes and check links, so broken references fail here, before merging.

## 7. How the site consumes it (to build)

- The code repository includes this one as a git submodule and always builds from the latest `main`.
- A merge to `main` triggers a Cloudflare deploy hook, so the site republishes.
- Pages generated at build time:

| Where | What | Which events |
| --- | --- | --- |
| `/console/events` | Timeline, newest first, filterable by type, network and component | All |
| `/console/events/<id>` | Title, summary, context, `refs` linked to explorer and release notes, links, updates, related events in both directions | Each |
| Site-wide banner | A discreet strip under the header linking to the event | See the banner rule above |
| `/console/events.xml`, `/console/events.json` | Feeds | All |

## 8. Still open

- Do the seven types cover what processes will generate (for example, network parameter changes)?
- Which processes come first, and the `source.name` of each.
- License for the data (CC0 is a natural fit for a factual record; CC BY 4.0 matches the rest of the content).
- Whether the remote lives under the project's GitHub organization, and its final name.

## 9. Next steps when work resumes

1. Write `scripts/validate.mjs` and the CI workflow (section 6).
2. Add the submodule to the code repository and build the Console pages (section 7).
3. Write the first real events and remove nothing from `examples/`.
