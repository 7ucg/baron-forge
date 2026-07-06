# Baron Forge — Installation

Anleitung, um das Plugin global in Claude Code zu installieren (gilt dann in jedem Projekt). Zwei Wege: per Slash-Command (empfohlen) oder direkt über `settings.json`.

---

## Weg A — Slash-Command (empfohlen)

In Claude Code:

```
/plugin marketplace add https://github.com/7ucg/baron-forge
/plugin install baron-forge@baron-forge-marketplace
```

Beim Install **scope: user/global** wählen → aktiv in allen Projekten.

---

## Weg B — direkt in settings.json (headless / ohne UI)

Datei: `~/.claude/settings.json` (Windows: `c:/Users/<user>/.claude/settings.json`). Zwei Einträge ergänzen:

```jsonc
{
  "extraKnownMarketplaces": {
    "baron-forge-marketplace": {
      "source": { "source": "github", "owner": "7ucg", "repo": "baron-forge" }
    }
  },
  "enabledPlugins": {
    "baron-forge@baron-forge-marketplace": true
  }
}
```

(Bestehende Keys nicht überschreiben — nur die zwei Einträge mergen.) Danach **Claude Code neu starten**.

---

## Prüfen

Nach dem Neustart:

```
/plugin                 # baron-forge enabled?
/help                   # /re /fix /mem-save /mem-load /mem-set /proto-decode sichtbar?
```

---

## Updaten

```
/plugin update baron-forge@baron-forge-marketplace
```

Oder Claude Code neu starten — lädt beim Marketplace-Sync die neueste Version.

---

## Deinstallieren

- Slash-Command: `/plugin uninstall baron-forge@baron-forge-marketplace`
- Oder in `settings.json` die zwei Einträge wieder entfernen und neu starten.

---

## Memory-Pfad (optional)

Memory liegt global unter `~/.claude/baron-forge/memory.md`. Überschreibbar via Env-Var `BARON_FORGE_HOME`.
