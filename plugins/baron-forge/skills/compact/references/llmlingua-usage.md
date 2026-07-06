# LLMLingua-2 Integration Reference

Use LLMLingua-2 to compress accumulated prose context (conversation history, bash logs,
file read dumps) before re-injecting into LLM API calls. Target: 40–60% token reduction
with minimal information loss.

---

## Install

```bash
pip install llmlingua
# Requires: torch, transformers (installs automatically)
# Model download: ~250MB on first use (microsoft/llmlingua-2-bert-base-multilingual-cased-meetingbank)
```

---

## Basic usage

```python
from llmlingua import PromptCompressor

compressor = PromptCompressor(
    model_name="microsoft/llmlingua-2-bert-base-multilingual-cased-meetingbank",
    use_llmlingua2=True,
    device_map="cpu"  # use "cuda" if GPU available
)

compressed = compressor.compress_prompt(
    context,
    rate=0.5,           # keep 50% of tokens — tune per use case
    force_tokens=["\n"] # always preserve newlines for structure
)

print(compressed["compressed_prompt"])
print(f"Ratio: {compressed['ratio']:.2f}x | Saved: {compressed['saving']}t")
```

---

## Rate calibration guide

| Context type | Recommended rate | Why |
|--------------|-----------------|-----|
| Bash history / terminal logs | 0.3–0.4 | High redundancy: echoed commands, repeated paths |
| File read dumps (prose docs) | 0.4–0.5 | Moderate redundancy |
| Conversation history | 0.5–0.6 | Lower redundancy: preserve user intent signals |
| Error tracebacks | 0.6–0.7 | High signal: don't over-compress |
| Code diffs/patches | 0.8+ | Very high signal: barely compress |

---

## Agentic loop integration pattern

In a multi-turn coding agent, compress accumulated history at each turn:

```python
class CompactAgent:
    def __init__(self, llm_client, compressor):
        self.client = llm_client
        self.compressor = compressor
        self.history = []          # raw turn history
        self.static_context = ""   # system prompt + codebase stubs (cache these)
        self.COMPRESS_THRESHOLD = 4000  # tokens before compressing history

    def turn(self, user_message: str) -> str:
        self.history.append({"role": "user", "content": user_message})

        # Compress history if it's grown large
        history_text = self._serialize_history()
        if self._token_estimate(history_text) > self.COMPRESS_THRESHOLD:
            compressed = self.compressor.compress_prompt(
                history_text,
                rate=0.55,
                force_tokens=["\n", "```"]
            )
            history_text = compressed["compressed_prompt"]

        # Build final prompt: static (cached) + compressed history
        messages = self._build_messages(history_text, user_message)

        response = self.client.messages.create(
            model="claude-sonnet-4-20250514",
            messages=messages,
            system=self.static_context  # hits cache after first call
        )

        reply = response.content[0].text
        self.history.append({"role": "assistant", "content": reply})
        return reply

    def _token_estimate(self, text: str) -> int:
        return len(text) // 4  # rough: 1 token ≈ 4 chars
```

---

## What LLMLingua-2 preserves vs strips

**Preserves (high mutual information):**
- Named entities: function names, variable names, error codes
- Conditional markers: if/else/when/unless
- Negations: not, never, failed, missing
- Numbers and specific values
- Structural markers: colons, brackets, delimiters

**Strips (low mutual information):**
- Filler connectives: "as a result of", "in order to", "it should be noted"
- Repeated context: second mention of same file path, same error
- Verbose bash output: progress bars, dots, repeated header lines
- Redundant confirmation lines: "Running...", "Done.", "OK"

---

## Combining with AST trimming

Full pipeline for a coding agent context window:

```python
def build_compressed_context(
    conversation_history: list[dict],
    source_files: dict[str, str],
    bash_logs: list[str],
    compressor: PromptCompressor
) -> str:

    # 1. AST-trim all source files (see ast-trimming.md)
    stubs = {path: extract_stub(src, detect_lang(path))
             for path, src in source_files.items()}
    code_context = format_stubs(stubs)

    # 2. Compress bash logs aggressively
    log_text = "\n".join(bash_logs)
    compressed_logs = compressor.compress_prompt(log_text, rate=0.35)["compressed_prompt"]

    # 3. Compress conversation history moderately
    hist_text = serialize_history(conversation_history[:-1])  # exclude latest turn
    compressed_hist = compressor.compress_prompt(hist_text, rate=0.55)["compressed_prompt"]

    # 4. Assemble: static stubs first (cacheable), then compressed dynamic context
    return f"""
=== CODEBASE STUBS ===
{code_context}

=== COMPRESSED SESSION HISTORY ===
{compressed_hist}

=== COMPRESSED BASH LOGS ===
{compressed_logs}
"""
```

---

## Failure modes to watch for

| Symptom | Cause | Fix |
|---------|-------|-----|
| LLM misses a key variable name | rate too aggressive | Raise rate to 0.6+ or add to `force_tokens` |
| Model forgets earlier decision | history compressed too hard | Keep last 2 turns uncompressed |
| Code suggestion breaks syntax | code compressed (don't do this) | Never compress code — only prose/logs |
| Slow first call | Model downloading | Pre-warm compressor at agent startup |
