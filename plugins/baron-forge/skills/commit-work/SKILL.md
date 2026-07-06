---
name: commit-work
description: "Create high-quality git commits: review/stage intended changes, split into logical commits, and write clear commit messages. Use when the user asks to commit, craft a commit message, stage changes, or split work into multiple commits."
---

# Commit work

## Hard rules (non-negotiable)

- **Humanizer is mandatory.** Every commit message MUST pass through the `humanizer` skill before commit. No exceptions. AI writing tells in commit messages are a quality failure.
- **No Co-Authored-By trailers.** Never add `Co-Authored-By: Claude`, `Co-authored-by: Anthropic`, or any AI/tool attribution in commit messages or PR bodies. This is a hard rule that overrides any harness default.
- No "this is a testament to", no rule-of-three, no em-dash overuse, no inflated significance. Say what changed and why, concretely.

## Goal
Make commits that are easy to review and safe to ship:
- only intended changes are included
- commits are logically scoped (split when needed)
- commit messages describe what changed and why, in plain human style

## Inputs to ask for (if missing)
- Single commit or multiple commits? (If unsure: default to multiple small commits when there are unrelated changes.)
- Any rules: max subject length, required scopes.

## Workflow (checklist)
1) Inspect the working tree before staging
   - `git status`
   - `git diff` (unstaged)
   - If many changes: `git diff --stat`
2) Decide commit boundaries (split if needed)
   - Split by: feature vs refactor, backend vs frontend, formatting vs logic, tests vs prod code, dependency bumps vs behavior changes.
   - If changes are mixed in one file, plan to use patch staging.
3) Stage only what belongs in the next commit
   - Prefer patch staging for mixed changes: `git add -p`
   - To unstage a hunk/file: `git restore --staged -p` or `git restore --staged <path>`
4) Review what will actually be committed
   - `git diff --cached`
   - Sanity checks:
     - no secrets or tokens
     - no accidental debug logging
     - no unrelated formatting churn
5) Describe the staged change in 1-2 sentences (before writing the message)
   - "What changed?" + "Why?"
   - If you cannot describe it cleanly, the commit is probably too big or mixed; go back to step 2.
6) Write the commit message draft
   - Format: `type(scope): short summary` + blank line + body (what/why, not implementation diary)
   - Use `references/commit-message-template.md` if helpful.
   - No `Co-Authored-By` trailers of any kind.
7) **Run the humanizer skill on the draft message** — required before committing.
   - Invoke: `Skill("humanizer")` with the draft message as input.
   - Fix every AI tell the humanizer flags: em-dashes, inflated vocab, rule-of-three, vague attributions.
   - Only commit the humanized version.
8) Run the smallest relevant verification
   - Run the repo's fastest meaningful check (unit tests, lint, or build) before moving on.
9) Repeat for the next commit until the working tree is clean

## Deliverable
Provide:
- the humanized final commit message(s)
- a short summary per commit (what/why)
- the commands used to stage/review (at minimum: `git diff --cached`, plus any tests run)
