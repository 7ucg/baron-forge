---
name: memory
description: "Persistent cross-session memory for Baron. Use whenever Baron says remember/save/note this, asks what you remember, references a past decision/fact/config, or starts work that should continue from earlier state. Backed by a global markdown store (baron-mem) that loads automatically at session start and is editable via the /mem-save, /mem-load, /mem-set commands. Save: project state, decisions, conventions, endpoints, key offsets/field maps, gotchas, todos."
---

# Memory

Global, project-independent store at `$HOME/.claude/baron-forge/memory.md`, managed by the `baron-mem` CLI (on PATH via the plugin). Auto-injected at session start by the SessionStart hook, so earlier facts are already in context.

## When to write
**Save proactively — don't wait to be told.** Whenever you solve something non-obvious or learn a durable fact during a task, `baron-mem save` it immediately (one terse line). A bash hook can't summarize for you; this is on you.
- Baron says "merk dir / remember / speicher das".
- A decision is made (architecture, naming, which lib, which approach).
- A hard-won fact: an offset, a protobuf field number, an endpoint, a server quirk (e.g. `IS_SANDBOX=1` inheritance), a build flag.
- An open todo to resume next session.

## How
```bash
baron-mem save "WA edge ws frame: 3-byte big-endian length prefix before Noise payload"
baron-mem set wa_proto_repo "github.com/7ucg/wa-proto-baron"
baron-mem get wa_proto_repo
baron-mem search libsignal
baron-mem show
```
- `save` = timestamped note (history). `set KEY val` = single-value fact (upsert). Use `set` for things that change, `save` for log/findings.
- Keep entries terse and factual — they're reloaded verbatim every session, so no fluff.

## When to read
- It's already loaded at session start. If unsure whether something was stored, `baron-mem search <term>` before claiming you don't know.

## Hygiene
- Don't store secrets/tokens/keys in plaintext memory — reference where they live instead.
- Prune stale entries with `baron-mem edit` when a fact is superseded.
