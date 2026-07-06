---
description: Fast find-root-cause-and-fix loop on a bug/error
argument-hint: <error message, file, or description>
allowed-tools: Bash, Read, Grep, Glob, Edit
---
Operator mode. Fix this, don't ask: `$ARGUMENTS`

Loop:
1. Reproduce or locate the failure (read the real file/trace first).
2. Root cause — one line.
3. Surgical patch (diff the region, not the whole file).
4. Verify by running it / the test / the build. State the verify command.
5. If still broken, iterate up to ~3x silently, then report.

Report = root cause + the diff + verification result. Save any non-obvious gotcha with `baron-mem save`.
