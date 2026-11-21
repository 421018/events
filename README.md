# 42 Chain network events

The record of network events for 42 Chain and Infinite Drive: releases, upgrades, incidents, security notices, governance results and maintenance. One JSON file per event, in English, validated against [`schema/event.schema.json`](schema/event.schema.json). The site shows them in the Console at `/console/events`.

> Status: specification agreed, not implemented yet. See [SPEC.md](SPEC.md).

## Writing an event

1. Create `events/<year>/<id>.json`. The `id` starts with the event date: `2026-12-02-mainnet-upgrade-v1-4-0`.
2. Follow the schema. Your editor can check it if the file starts with `"$schema": "../../schema/event.schema.json"`.
3. Open a pull request. Every event, from a person or a process, goes through review.

To change an event, append to its `updates`; never rewrite or delete what is there. To link it to an earlier event, add a `relations` entry in the new one.

See [`examples/`](examples/) for complete events (illustrative data, never published).
