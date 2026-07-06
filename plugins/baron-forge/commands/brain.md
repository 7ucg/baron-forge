---
description: Build or rebuild the project brain (.baron-brain.md). Scans the current project and writes a compact map of layout, entry points, commands, conventions, and gotchas — injected at every session start.
argument-hint: "[rebuild|show]"
allowed-tools: Bash, Read, Grep, Glob, Write, Agent
---
$ARGUMENTS

Run the brain-builder agent on this project:

!`bash "${CLAUDE_PLUGIN_ROOT}/bin/baron-brain" show 2>/dev/null && echo "--- existing brain above ---" || echo "no brain yet"`

Dispatch the brain-builder agent to scan the project and write `.baron-brain.md` at the git root:

Use the Agent tool with `subagent_type: "baron-forge:brain-builder"` and prompt: "Build/update the project brain for the project at the current git root. Scan fully, write .baron-brain.md."
