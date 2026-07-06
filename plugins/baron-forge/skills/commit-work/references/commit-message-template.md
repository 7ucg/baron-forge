# Commit message template

```text
<type>(<scope>): <summary>

<What changed.>
<Why it changed.>
```

Rules:

- Summary imperative and specific: "Add", "Fix", "Remove", "Refactor" — not "enhance", "pivotal", "comprehensive".
- Body: what/why, not implementation diary. No em-dashes, no rule-of-three, no "this is a testament to".
- **Run humanizer before committing.** AI tells in commit messages are a quality failure.
- **No `Co-Authored-By` trailers** of any kind — no Claude, no Anthropic, no AI attribution.
- If breaking: `!` in header and/or `BREAKING CHANGE:` footer.
