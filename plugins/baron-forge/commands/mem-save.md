---
description: Save a note/decision/fact to persistent global memory
argument-hint: <text to remember>
allowed-tools: Bash(baron-mem:*)
---
Save this to persistent memory, then confirm in one line:

!`baron-mem save "$ARGUMENTS"`

If `$ARGUMENTS` reads like a single keyed fact (a config value, repo, endpoint, flag), also `baron-mem set` an appropriate KEY for it.
