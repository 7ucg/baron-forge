---
name: dependency-updater
description: Smart dependency management for any language. Auto-detects project type, applies safe updates automatically, prompts for major versions, diagnoses and fixes dependency issues.
license: MIT
metadata:
  version: 1.0.0
---

# Dependency Updater

## Supported languages

| Language | Package file | Update tool | Audit |
|----------|-------------|-------------|-------|
| Node.js | package.json | `taze` | `npm audit` |
| Python | requirements.txt / pyproject.toml | `pip-review` | `pip-audit` |
| Go | go.mod | `go get -u` | `govulncheck` |
| Rust | Cargo.toml | `cargo update` | `cargo audit` |
| Ruby | Gemfile | `bundle update` | `bundle audit` |
| Java | pom.xml / build.gradle | `mvn versions:*` | `mvn dependency-check:check` |
| .NET | *.csproj | `dotnet outdated` | `dotnet list package --vulnerable` |

## Update policy

| Type | Range change | Action |
|------|-------------|--------|
| PATCH | `x.y.z → x.y.Z` | Auto-apply |
| MINOR | `x.y.z → x.Y.0` | Auto-apply |
| MAJOR | `x.y.z → X.0.0` | Prompt user individually |
| Fixed (no `^`/`~`) | — | Skip (intentionally pinned) |

## Workflow

1. **Detect** — scan for package files; identify package manager
2. **Prerequisites** — verify tools are installed; suggest install if missing
3. **Scan** — run language-specific outdated check; categorize MAJOR/MINOR/PATCH/Fixed
4. **Auto-apply** — apply MINOR + PATCH; report what changed
5. **Prompt** — `AskUserQuestion` for each MAJOR individually (current → new)
6. **Apply approved majors**
7. **Finalize** — run install command + security audit

## Commands by language

### Node.js
```bash
taze                              # scan
taze minor --write                # apply minor+patch
taze major --write --include pkg  # apply approved majors
npm audit && npm audit fix        # security
taze -r                           # monorepo
```

### Python
```bash
pip list --outdated
pip install --upgrade <pkg>
pip-audit
```

### Go
```bash
go list -m -u all
go get -u ./...
go mod tidy
govulncheck ./...
```

### Rust
```bash
cargo outdated
cargo update
cargo audit
```

### Ruby
```bash
bundle outdated
bundle update
bundle audit
```

### Java (Maven)
```bash
mvn versions:display-dependency-updates
mvn versions:use-latest-releases
mvn dependency-check:check
```

### .NET
```bash
dotnet list package --outdated
dotnet add package <PackageName>
dotnet list package --vulnerable
```

## Diagnosis

| Issue | Symptoms | Fix |
|-------|----------|-----|
| Version conflict | "Cannot resolve dependency tree" | Clean install; use overrides/resolutions |
| Peer dependency | "Peer dependency not satisfied" | Install required peer version |
| Security vuln | `npm audit` shows issues | `npm audit fix` or manual update |
| Unused deps | Bloated bundle | `depcheck` (Node) or equivalent |
| Duplicate deps | Multiple versions | `npm dedupe` or equivalent |

Emergency reset (Node): `rm -rf node_modules package-lock.json && npm cache clean --force && npm install`

## Security severity response

| Severity | Action |
|----------|--------|
| Critical | Fix immediately |
| High | Fix within 24h |
| Moderate | Fix within 1 week |
| Low | Fix in next release |

## Anti-patterns

- Don't update fixed versions (intentionally pinned — skip them)
- Don't auto-apply MAJOR (breaking changes — prompt individually)
- Don't skip lock files (irreproducible builds)
- Don't ignore security alerts

## Scripts

| Script | Purpose |
|--------|---------|
| `scripts/check-tool.sh` | Verify tool is installed |
| `scripts/run-taze.sh` | Run taze with proper flags |
