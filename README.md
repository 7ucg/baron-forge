# Baron Forge

Globales Claude-Code-Plugin: Operator-Persona (knapp, entscheidet, führt aus), starker Full-Stack-Coder + Debugger + Reverse-Engineering-Rig, Messaging-Protokoll-Wissen (libsignal / WhatsApp / Signal / protobuf / ws / VoIP) und **persistentes Memory über alle Sessions**.

## Inhalt (v2.0.0 — 16 Skills)

**Core (eigene):**

- **operator** — Verhaltensbasis: knapp, deutsch default, handeln statt fragen, clean-code, speed/token/agent-economy, find-fix-loop.
- **reverse-engineering** — apk/dex/smali, jadx/apktool, .so/native, flutter/RN, frida, resign.
- **protocols** — libsignal (X3DH/Double Ratchet), WhatsApp (Noise/binary-XMPP/wa-proto), protobuf, ws/stanza/graphql/mex, media/VoIP (WebRTC/SRTP).
- **fullstack** — Rust/C/Node/Dart-Flutter/React + Cross-Bridges (napi/frb), npm publish.
- **memory** — globaler Markdown-Store, auto-load + prompt-relevanter Recall, proaktives Speichern.
- **compact** — maximale Token-Effizienz (Tier-Notation, Diffs, Caching, Batch-API).

**Zugekauft (softaworks/agent-toolkit + blader):**

- **humanizer** — entfernt AI-Schreibmuster aus Prosa.
- **commit-work** — Conventional Commits, logisches Splitten.
- **crafting-effective-readmes**, **mermaid-diagrams**, **database-schema-designer**, **qa-test-planner**, **dependency-updater**, **ship-learn-next**, **plugin-forge**, **skill-judge**.

**Commands** — `/mem-save` `/mem-load` `/mem-set` `/re` `/fix` `/proto-decode`

**Agents** (isolierte Subagents, parallel dispatchbar) — `debugger`, `re-analyst`, `apk-unpacker`, `protocol-analyst`, `fixer`

**Hooks (Automatik):**

- SessionStart → Memory in Kontext laden
- UserPromptSubmit → prompt-relevante Memory injizieren
- PostToolUse(Edit/Write) → Auto-Format/Lint, meldet nur Fehler (`baron-quality`)
- PostToolUse(Bash) → Fehlersignatur → 1-Zeilen-Fix-Hint (`baron-bash-check`)
- PreToolUse(Bash) → Tripwire: blockt nur Total-Loss-Befehle, loggt riskante (`baron-guard`)

**Bins** — `baron-mem` (Memory-CLI), `baron-quality`, `baron-bash-check`, `baron-guard`, `baron-doctor` (Self-Check: `bash bin/baron-doctor`).

## Installation (global, gilt überall)

> Vollständige Schritt-für-Schritt-Anleitung (Slash-Command + settings.json + Update/Deinstall): siehe [INSTALL.md](INSTALL.md).

Kurzfassung (Details → [INSTALL.md](INSTALL.md)):

```text
/plugin marketplace add https://github.com/7ucg/baron-forge
/plugin install baron-forge@baron-forge-marketplace   # scope: user/global
```

Prüfen:

```text
/plugin           # baron-forge enabled?
/help             # commands sichtbar?
baron-mem path    # Memory-Datei-Pfad
```

## Memory benutzen

```bash
baron-mem save "WA ws frame: 3-byte BE length prefix vor Noise payload"
baron-mem set wa_proto_repo "github.com/7ucg/wa-proto-baron"
baron-mem search libsignal
```

Oder im Chat: `/mem-save ...`, `/mem-load`. Wird bei jedem Session-Start automatisch injiziert.
Datei: `~/.claude/baron-forge/memory.md` (Markdown, editierbar). Override-Pfad via `BARON_FORGE_HOME`.

## Hinweis

Die Operator-Persona ist maximal knapp/ausführend für deine eigene RE-/Coding-/Protokoll-Arbeit. Sie schaltet keine harten Kern-Sicherheitsgrenzen ab (würde technisch eh nicht greifen) — für deinen normalen Workflow ist davon nichts betroffen.
