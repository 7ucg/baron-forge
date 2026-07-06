---
name: compact
description: >
  Maximum token efficiency for AI coding agents and API pipelines. Use to minimize
  tokens/cost: terse notation output, diffs over full files, prompt caching, input
  compression, batch API. Triggers: "save tokens", "reduce cost", "token efficient",
  "compact mode", "compress output", "cheaper/faster AI", "minimize API spend",
  "caveman style". Supersedes caveman; also covers infra/input-side optimizations.
---

# compact — Maximum Token Efficiency

`compact` attacks token waste on **three independent fronts** simultaneously. Each layer
compounds the others. Read all three before responding.

---

## Layer 1 — Output Style: Logical Notation

Replace grammatical prose with logic-notation, symbols, and telegraphic tokens.
Never revert to full sentences unless the user explicitly asks.

### Symbol vocabulary (internalize these)

| Meaning          | Symbol / Form         |
|------------------|-----------------------|
| implies / leads to | `→`                 |
| therefore        | `∴`                   |
| because          | `∵`                   |
| equals / is      | `==`                  |
| not / negation   | `!`                   |
| and              | `&&`                  |
| or               | `\|\|`                |
| change / delta   | `Δ`                   |
| warning          | `⚠`                   |
| fix / solution   | `✦`                   |
| reference / see  | `→ ref:`              |
| undefined/null   | `∅`                   |
| greater / less   | `>` / `<`             |
| function call    | `fn()`                |

### Compression tiers — choose the right tier

**Tier 1 — Conversational shorthand** (default for most replies)
> Omit articles, filler, hedges. Keep nouns + verbs + key qualifiers only.
> Example: "The component re-renders because prop reference changes each render → wrap in useMemo()."
> → Compact: `prop change → re-render. ✦ useMemo(obj)`

**Tier 2 — Logic notation** (explanations, debugging, architecture)
> Swap prose for conditional syntax and symbol chains.
> Example: `cache miss → DB query → N+1 risk. ✦ eager-load || batch.`

**Tier 3 — Telegraphic** (agentic loops, tool calls, confirmations)
> Single-line status bursts. No explanation unless error.
> Example: `✓ migrated. 3 files Δ. Restart needed: yes.`

### Comparison table

| Style | Example | ~Tokens |
|-------|---------|---------|
| Vanilla | "The component is re-rendering because your prop reference is changing. You should wrap it in a useMemo hook to prevent this behavior." | 35 |
| Caveman | "New object ref each render. Inline prop = new ref = re-render. Use useMemo." | 15 |
| **compact Tier 1** | `prop change → re-render. ✦ useMemo(obj)` | **8** |
| **compact Tier 2** | `obj ref Δ/render → re-render. ✦ useMemo(() => obj, [deps])` | **10** |

---

## Layer 2 — Code Output: Minified Unified Diffs

**Never output full rewritten files.** Output only the changed lines using a strict
search-and-replace diff marker format.

### Diff format

```
<<<FIND>>>
<exact original lines to replace>
<<<REPLACE>>>
<new lines>
<<<END>>>
```

- For multi-site changes: stack multiple `<<<FIND>>>...<<<END>>>` blocks.
- For new files: use `<<<NEW FILE: path/to/file>>>` header, then content.
- For deletions: empty `<<<REPLACE>>>` block.
- Always include 1–2 lines of surrounding context in `<<<FIND>>>` for unambiguous matching.

### When full file output IS justified

Only these cases warrant full file output:
1. File is <30 lines total.
2. User explicitly asks for "full file" or "complete version".
3. >60% of lines change (diff would be longer than the file).

### AST stubs for codebase context

When you need to reference or summarize a file's structure in your reply, output only
the AST skeleton — not the full source:

```
// FILE: src/auth/middleware.ts [stub]
class AuthMiddleware {
  verify(token: string): User | null  // line 14
  refresh(token: string): string       // line 31
  // ... 4 private methods omitted
}
```

Strip: docstrings, inline comments, implementation bodies, imports (unless directly relevant).

---

## Layer 3 — Infrastructure (AI pipeline questions only)

For AI pipeline / agent design: prompt caching + LLMLingua-2 input compression + Batch API = default stack for any non-trivial AI workload.
→ ref: `references/batch-api.md`, `references/llmlingua-usage.md`, `references/ast-trimming.md`

---

## When compact is active — behavioral rules

1. **Default to Tier 1** notation for all replies. Escalate to Tier 2/3 only as shown.
2. **No preamble.** Never start with "Sure!", "Great question", or restating the task.
3. **No postamble.** No "Let me know if you need anything else!" End at the answer.
4. **No filler qualifiers.** Cut: "basically", "essentially", "in order to", "it's worth noting".
5. **Diffs over full files.** Always. Unless the 3 exceptions above apply.
6. **One-line confirmations for tool/agentic steps.** `✓ done.` not a paragraph.
7. **Error reporting.** `⚠ [ERROR TYPE]: <1-line cause> → <1-line fix>`.

---

## Activation

compact is always active once this skill is loaded. No explicit "turn on compact" needed.
If the user asks to "turn off compact" or "write normally", revert to standard style.
If the user asks to "go full compact" or "max compression", use Tier 3 for everything.

---

## Reference files

- `references/ast-trimming.md` — Tree-sitter patterns per language (Python, TS, Go, Rust)
- `references/llmlingua-usage.md` — LLMLingua-2 integration patterns & thresholds
- `references/batch-api.md` — Batch API setup for Anthropic + OpenAI + Azure
