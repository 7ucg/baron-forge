# AST Trimming Reference — Tree-sitter Patterns

Use these patterns to extract signature-only stubs from source files before injecting
into LLM context. Goal: reduce code context by 60–80% while preserving navigational utility.

---

## Install

```bash
pip install tree-sitter tree-sitter-languages
```

---

## Universal stub extractor (Python)

```python
from tree_sitter_languages import get_language, get_parser

def extract_stub(source: str, lang: str) -> str:
    """Return AST stub: classes + function signatures only. No bodies, no comments."""
    language = get_language(lang)
    parser = get_parser(lang)
    tree = parser.parse(source.encode())
    lines = source.splitlines()
    keep = set()

    def walk(node):
        if node.type in SIGNATURE_NODES.get(lang, []):
            # Keep the signature line(s) only
            keep.add(node.start_point[0])
            for child in node.children:
                if child.type in HEADER_CHILD_NODES.get(lang, []):
                    keep.add(child.start_point[0])
        for child in node.children:
            walk(child)

    walk(tree.root_node)
    result = []
    for i, line in enumerate(lines):
        if i in keep:
            result.append(line)
    return "\n".join(result)
```

---

## Language-specific node types

### Python

```python
SIGNATURE_NODES = {
    "python": ["class_definition", "function_definition", "decorated_definition"]
}
HEADER_CHILD_NODES = {
    "python": ["identifier", "parameters", "type", "return_type"]
}
```

**Strips:** docstrings (`expression_statement` with string), function bodies, comments.

**Example input:**
```python
class DataLoader:
    """Loads data from various sources."""

    def fetch(self, url: str, timeout: int = 30) -> dict:
        """Fetch data from URL."""
        resp = requests.get(url, timeout=timeout)
        resp.raise_for_status()
        return resp.json()

    def _validate(self, data: dict) -> bool:
        if not data:
            raise ValueError("empty")
        return True
```

**Stub output:**
```python
class DataLoader:
    def fetch(self, url: str, timeout: int = 30) -> dict  # line 4
    def _validate(self, data: dict) -> bool               # line 10
```

Token reduction: ~78%

---

### TypeScript / JavaScript

```python
SIGNATURE_NODES = {
    "typescript": [
        "class_declaration", "method_definition", "function_declaration",
        "interface_declaration", "type_alias_declaration", "export_statement"
    ]
}
HEADER_CHILD_NODES = {
    "typescript": ["identifier", "formal_parameters", "type_annotation", "type_parameters"]
}
```

**Example stub:**
```typescript
// FILE: src/auth/service.ts [stub]
interface AuthConfig { ... }           // line 1
type TokenPair = { ... }               // line 7
class AuthService {
  constructor(config: AuthConfig)      // line 12
  async login(creds: Credentials): Promise<TokenPair>   // line 18
  async refresh(token: string): Promise<string>          // line 29
  private validate(token: string): boolean               // line 41
}
```

---

### Go

```python
SIGNATURE_NODES = {
    "go": ["function_declaration", "method_declaration", "type_declaration", "interface_type"]
}
```

**Example stub:**
```go
// FILE: pkg/cache/redis.go [stub]
type RedisCache struct { ... }                    // line 12
func NewRedisCache(addr string) *RedisCache       // line 21
func (c *RedisCache) Get(key string) ([]byte, error)   // line 31
func (c *RedisCache) Set(key string, val []byte, ttl time.Duration) error  // line 44
```

---

### Rust

```python
SIGNATURE_NODES = {
    "rust": ["function_item", "impl_item", "struct_item", "trait_item", "enum_item"]
}
```

**Example stub:**
```rust
// FILE: src/storage/mod.rs [stub]
pub struct Storage { ... }                        // line 8
pub trait Storable { ... }                        // line 15
impl Storage {
    pub fn new(path: &str) -> Result<Self, Error>  // line 24
    pub fn read(&self, key: &str) -> Option<Vec<u8>>  // line 31
    pub fn write(&mut self, key: &str, val: &[u8]) -> Result<(), Error>  // line 40
}
```

---

## Context injection pattern

When injecting stubs into an LLM prompt, use this wrapper:

```
=== CODEBASE CONTEXT (stubs only) ===
[FILE: src/auth/service.ts — 847 lines → stub shown]
<stub content>

[FILE: src/db/queries.go — 312 lines → stub shown]
<stub content>
=== END CONTEXT ===
```

Always annotate: original line count + stub line count so the LLM knows scale.

---

## Threshold guide

| File size | Strategy |
|-----------|----------|
| <50 lines | Inject full file (stub overhead not worth it) |
| 50–200 lines | Stub: classes + public functions only |
| 200–1000 lines | Stub: public API surface only |
| >1000 lines | Stub + module-level docstring summary, 3 lines max |
