# Baron Forge

Globales Claude-Code-Plugin. Operator-Persona (knapp, entscheidet, führt aus), persistentes Memory, automatische Agent-Auswahl, Projekt-Brain-Injektion, und konsequente Token-Einsparung durch Fast-Path-Hooks.

## Inhalt (v2.0.0)

### Skills — 10 aktiv, 6 deaktiviert

| Skill | Funktion |
|-------|----------|
| **operator** | Verhaltensbasis: deutsch by default, handeln statt fragen, Agent-Dispatch-Tabelle, Humanizer immer aktiv |
| **compact** | Token-Effizienz: Tier-Notation, Diffs statt Full-Files, refs zu Batch-API/LLMLingua/Caching |
| **memory** | Markdown-Store, auto-recall, proaktives Speichern |
| **reverse-engineering** | APK/DEX/smali, jadx/apktool, .so/native, Flutter/RN, Frida |
| **protocols** | libsignal (X3DH/Double Ratchet), WhatsApp (Noise/binary-XMPP/wa-proto), protobuf, WebSocket/stanza |
| **fullstack** | Rust/C/Node/Dart-Flutter/React, Cross-Bridges (napi/frb), npm publish |
| **commit-work** | Conventional Commits, logisches Splitten, kein Co-Authored-By |
| **humanizer** | Entfernt AI-Schreibmuster (auch inline in operator eingebaut) |
| **dependency-updater** | taze/cargo-upgrade/pub upgrade Workflows |
| **plugin-forge** | Claude-Code-Plugins erstellen, Manifeste, Marketplace |

Deaktiviert (`.disabled`): `crafting-effective-readmes`, `mermaid-diagrams`, `database-schema-designer`, `qa-test-planner`, `ship-learn-next`, `skill-judge`

### Agents — 12, isoliert + parallel dispatchbar

| Agent | Wann |
|-------|------|
| `debugger` | Build/Test schlägt fehl, Stack Trace, falsches Verhalten |
| `fixer` | Failing Build/Test bis Green treiben |
| `code-reviewer` | Code reviewen ohne Fixes (CRITICAL/HIGH/MEDIUM/LOW) |
| `test-writer` | Tests schreiben + laufen lassen bis grün |
| `committer` | Stage + humanisierten Commit erstellen |
| `refactor` | Ein Modul/Funktion refactorn, Tests bleiben grün |
| `rust-analyst` | cargo check/clippy/audit/Perf |
| `node-analyst` | Node.js/Baileys/Anthropic-SDK Audit |
| `brain-builder` | `.baron-brain.md` schreiben/aktualisieren |
| `re-analyst` | APK/.so/Binary Tiefenanalyse, Protokoll-RE |
| `apk-unpacker` | APK/AAB dekompilieren, Struktur-Report |
| `protocol-analyst` | Wire-Protokoll reverse engineeren |

Automatische Auswahl über die Dispatch-Tabelle im `operator`-Skill — kein manuelles Benennen nötig.

### Commands

`/mem-save` `/mem-load` `/mem-set` `/re` `/fix` `/proto-decode` `/brain`

### Hooks — vollautomatisch

| Event | Script | Was passiert |
|-------|--------|-------------|
| SessionStart | `baron-mem hook-session` | Banner + Memory-Injektion + Git-Status in einem Python-Call |
| SessionStart | `baron-brain hook-inject` | `.baron-brain.md` des aktuellen Projekts injizieren (+ Stale-Warnung) |
| UserPromptSubmit | `baron-mem hook-recall` | Turn-Counter; kompakter Hint alle 15 Turns; Stop-Word-Filter; Memory-Recall |
| UserPromptSubmit | `baron-stack` | Stack-Trace-Referenzen extrahieren (Rust/Node/Python/Dart) |
| PreToolUse(Bash) | `baron-guard` | Tripwire: destruktive Befehle blocken (rm -rf, fork bomb, dd auf /dev/sd…) |
| PreToolUse(Write) | `baron-secret` | API-Key-Scan vor dem Schreiben (sk-, ghp_, AKIA, AIza…) |
| PostToolUse(Edit/Write) | `baron-quality` | Auto-Format/Lint; meldet nur echte Fehler, kein Noise |
| PostToolUse(Bash) | `baron-bash-check` | Fehlersignatur → 1-Zeilen-Fix-Hint; Failure-Flag → Memory-Save-Hint nach Recovery |

### Bin-Scripts

`baron-mem` · `baron-guard` · `baron-quality` · `baron-bash-check` · `baron-brain` · `baron-secret` · `baron-stack` · `baron-doctor`

Self-Check: `bash bin/baron-doctor`

---

## Token-Einsparungen

Alle Zahlen sind Schätzungen auf Basis von Zeilenzahlen × durchschnittlicher Token-Dichte (~8 tok/Zeile) und gemessenen Startup-Kosten.

### Skill-Dateien komprimiert

| Datei | Vorher | Nachher | Gespart |
|-------|--------|---------|---------|
| `humanizer/SKILL.md` | 436 Zeilen | 143 Zeilen | ~2 300 tok |
| `dependency-updater/SKILL.md` | 492 Zeilen | 121 Zeilen | ~3 000 tok |
| `compact/SKILL.md` Layer 3 | 73 Zeilen | 4 Zeilen | ~550 tok |
| `operator/SKILL.md` Humanizer-Block | 23 Zeilen | 7 Zeilen | ~130 tok |
| `plugin-forge/SKILL.md` Pattern-Bäume | 47 Zeilen | 5 Zeilen | ~340 tok |

Skills werden nur bei Bedarf geladen — die Einsparung gilt pro Skill-Load in einer Session.

### Hook-Fast-Paths (weniger Python-Spawns)

Jeder Python-Interpreter-Start kostet ~150–300 ms und ~1–2 k Kontext-Tokens (Overhead durch Subprocess-Output). Die Fast-Paths vermeiden das:

| Hook | Fast-Path-Bedingung | Python-Spawn ohne FP | Mit FP |
|------|--------------------|--------------------|--------|
| `baron-guard` | kein destruktives Pattern im JSON | immer | nur wenn verdächtig (~5 % der Calls) |
| `baron-bash-check` | kein Fehler-Keyword in Output | immer | nur bei Fehlern (~20 %) |
| `baron-mem hook-recall` | Prompt ≤ 120 Zeichen | immer | nur bei langen Prompts mit Memory-Hits |
| `baron-secret` | kein Token-Prefix im File-Content | immer | nur bei echten Treffern (<1 %) |

**Hochrechnung pro Session (20 Turns):**

- `baron-guard`: 20 Aufrufe × 200 ms eingespart = ~4 s schneller
- `baron-bash-check`: ~16 von 20 ohne Python = ~3 s schneller
- `baron-recall` Short-Path: ~8 kurze Prompts übersprungen = ~1,5 s + ~16 k tok weniger Overhead

### Brain-Injektion statt Live-Exploration

Ohne Brain macht der Model beim Session-Start 10–15 Datei-Reads um Projektstruktur zu verstehen:

- 10 × Read(~200 Zeilen) = ~16 000 Tokens reine Exploration
- Plus Overhead für Glob/Grep-Calls

Mit `.baron-brain.md` (≤120 Zeilen ≈ 800 Tokens injiziert einmalig):

- Exploration entfällt komplett → **~15 000 tok gespart pro Session** in unbekannten Projekten
- Auch neue Agents/Chats starten sofort mit Kontext

### Session-Banner ohne Skill-Namensliste

Die Skill-Namen waren redundant (SDK injiziert sie schon im system-reminder). Entfernt:

- ~70 Tokens pro Session-Start gespart

### Memory-Recall Stop-Word-Filter

50+ Stop-Wörter (DE + EN) reduzieren False-Positive-Hits im Memory-Lookup. Weniger irrelevante Memory-Zeilen = weniger Token-Noise im Kontext. Typisch ~50–200 tok gespart pro relevanter Recall-Event.

### Zusammenfassung

| Bereich | Einsparung (typische Session) |
|---------|-------------------------------|
| Skill-Dateien (wenn komprimierte Skills geladen) | 500–6 000 tok |
| Hook-Fast-Paths (Python-Spawns vermieden) | 5–10 s + ~20 k tok Overhead |
| Brain-Injektion vs. Exploration | ~15 000 tok |
| Banner-Kürzung | ~70 tok |
| Memory Stop-Word-Filter | 100–400 tok |
| **Gesamt (grob)** | **~15 000–40 000 tok pro Session** |

Der größte Hebel ist die Brain-Injektion — besonders bei Projekten mit vielen Dateien.

---

## Installation (global, gilt überall)

> Schritt-für-Schritt mit Slash-Command + settings.json + Update/Deinstall: [INSTALL.md](INSTALL.md)

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

---

## Memory benutzen

```bash
baron-mem save "WA ws frame: 3-byte BE length prefix vor Noise payload"
baron-mem set wa_proto_repo "github.com/7ucg/wa-proto-baron"
baron-mem search libsignal
```

Im Chat: `/mem-save …`, `/mem-load`. Automatisch bei Session-Start injiziert.
Datei: `~/.claude/baron-forge/memory.md` (Markdown, manuell editierbar).
Override: `BARON_FORGE_HOME=/pfad/zu/dir`.

---

## Projekt-Brain

```text
/brain          # brain-builder Agent startet, schreibt .baron-brain.md
```

Wird automatisch bei jedem Session-Start injiziert. Der Brain-Builder erkennt Rust/Node/Dart/Python/Go/Android-Projekte, mappt Entry Points, Commands und Konventionen — alles unter 120 Zeilen.

Wenn `Cargo.toml`, `package.json`, `pubspec.yaml` etc. bearbeitet werden, setzt `baron-quality` automatisch einen `.baron-brain-stale`-Marker. Beim nächsten Session-Start erscheint ein Hinweis.

---

## Hinweis

Die Operator-Persona ist maximal knapp/ausführend für eigene RE-/Coding-/Protokoll-Arbeit. Harte Kern-Sicherheitsgrenzen bleiben aktiv (lassen sich technisch sowieso nicht per Skill abschalten).
