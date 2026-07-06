# Batch API Reference

Route non-interactive AI workloads through Batch APIs for a flat 50% cost reduction.
Batch requests process asynchronously (results in minutes to hours, not seconds).

---

## Decision: Batch vs Real-time

```
user waiting for response?
├── yes → real-time API (standard)
└── no  → batch API (50% cheaper)
    ├── automated code review
    ├── bulk test generation
    ├── structural refactoring passes
    ├── CI/CD lint + suggestion pipelines
    ├── embedding generation at scale
    ├── data extraction / classification jobs
    └── nightly analysis runs
```

---

## Anthropic Batch API

**Cost:** 50% of standard per-token pricing.
**Throughput:** up to 10,000 requests per batch.
**Latency:** results available within 24h (typically minutes for small batches).

```python
import anthropic

client = anthropic.Anthropic()

# 1. Create batch
batch = client.beta.messages.batches.create(
    requests=[
        {
            "custom_id": f"review-{i}",
            "params": {
                "model": "claude-sonnet-4-20250514",
                "max_tokens": 1024,
                "messages": [
                    {"role": "user", "content": f"Review this code:\n{code_snippet}"}
                ]
            }
        }
        for i, code_snippet in enumerate(code_snippets)
    ]
)

batch_id = batch.id
print(f"Batch submitted: {batch_id}")

# 2. Poll for completion
import time

while True:
    status = client.beta.messages.batches.retrieve(batch_id)
    if status.processing_status == "ended":
        break
    print(f"Processing... {status.request_counts.processing} remaining")
    time.sleep(30)

# 3. Fetch results
results = {}
for result in client.beta.messages.batches.results(batch_id):
    if result.result.type == "succeeded":
        results[result.custom_id] = result.result.message.content[0].text
    else:
        results[result.custom_id] = f"ERROR: {result.result.error}"
```

---

## OpenAI Batch API

**Cost:** 50% of standard per-token pricing.
**Throughput:** up to 50,000 requests per batch, 100MB file limit.
**Latency:** 24h window (usually much faster).

```python
import json
from openai import OpenAI

client = OpenAI()

# 1. Write JSONL file
tasks = []
for i, code_snippet in enumerate(code_snippets):
    tasks.append({
        "custom_id": f"review-{i}",
        "method": "POST",
        "url": "/v1/chat/completions",
        "body": {
            "model": "gpt-4o",
            "max_tokens": 1024,
            "messages": [
                {"role": "user", "content": f"Review this code:\n{code_snippet}"}
            ]
        }
    })

with open("batch_tasks.jsonl", "w") as f:
    for task in tasks:
        f.write(json.dumps(task) + "\n")

# 2. Upload + create batch
with open("batch_tasks.jsonl", "rb") as f:
    uploaded = client.files.create(file=f, purpose="batch")

batch = client.batches.create(
    input_file_id=uploaded.id,
    endpoint="/v1/chat/completions",
    completion_window="24h"
)

batch_id = batch.id

# 3. Poll + retrieve
import time

while True:
    status = client.batches.retrieve(batch_id)
    if status.status == "completed":
        break
    time.sleep(30)

output = client.files.content(status.output_file_id)
results = {}
for line in output.text.strip().split("\n"):
    item = json.loads(line)
    results[item["custom_id"]] = item["response"]["body"]["choices"][0]["message"]["content"]
```

---

## Prompt Caching (Anthropic)

Cache static context to cut repeated-input costs by up to 90%.

```python
response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    system=[
        {
            "type": "text",
            "text": LARGE_STATIC_CONTEXT,   # system prompt + codebase stubs
            "cache_control": {"type": "ephemeral"}  # ← marks for caching
        }
    ],
    messages=[
        {"role": "user", "content": user_message}
    ]
)

# Check cache hit
usage = response.usage
cache_read = getattr(usage, "cache_read_input_tokens", 0)
cache_write = getattr(usage, "cache_creation_input_tokens", 0)
print(f"Cache read: {cache_read}t | Cache write: {cache_write}t")
```

**Cache lifetime:** 5 minutes (Anthropic ephemeral). Re-used on every call within window.
**Minimum size:** 1024 tokens to be eligible for caching.
**Cost:** cache writes billed at 1.25x normal; reads at 0.1x normal → break-even after ~1.4 reads.

---

## Combined stack: Batch + Cache + Compact output

```python
def run_bulk_review_pipeline(
    files: list[tuple[str, str]],  # (custom_id, code)
    static_system_prompt: str,
    compressor,                     # LLMLingua-2 instance
    ast_extractor                   # Tree-sitter extractor
) -> dict[str, str]:

    requests = []
    for custom_id, code in files:
        # Compress code context via AST stub
        stub = ast_extractor.extract(code)

        requests.append({
            "custom_id": custom_id,
            "params": {
                "model": "claude-sonnet-4-20250514",
                "max_tokens": 512,
                "system": [
                    {
                        "type": "text",
                        "text": static_system_prompt,
                        "cache_control": {"type": "ephemeral"}  # cached across all batch items
                    }
                ],
                "messages": [
                    {"role": "user", "content": f"Review:\n{stub}"}
                ]
            }
        })

    # Fire batch (50% cost reduction)
    batch = client.beta.messages.batches.create(requests=requests)
    # ... poll and collect as above
```

---

## Cost model summary

Baseline cost (naïve, no optimization) = **100%**

| Optimization | Reduction applied to |
|---|---|
| Compact output style (Tier 1–2) | Output tokens: −65% |
| Unified diffs (no full rewrites) | Output tokens: −80% on code |
| AST stub injection | Input tokens: −60% |
| LLMLingua-2 history compression | Input tokens: −50% |
| Prompt caching (static context) | Input tokens: −90% on cached portion |
| Batch API routing | Total cost: −50% financial |

**Compounded effective cost: ~5–10% of baseline** on a long agentic pipeline using all layers.
