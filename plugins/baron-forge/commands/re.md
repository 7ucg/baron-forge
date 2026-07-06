---
description: Reverse-engineer an APK / DEX / .so target end-to-end
argument-hint: <path-to-apk-or-so> [focus e.g. "networking", "libsignal", "ws"]
allowed-tools: Bash, Read, Grep, Glob
---
Reverse-engineer the target using the reverse-engineering skill. Target + focus: `$ARGUMENTS`

Run the standard workflow, autonomously, operator mode (no asking):
1. Inventory the file (`unzip -l` / `readelf`/`file`).
2. Decompile: apktool + jadx for APK; r2/objdump/nm for .so.
3. Sweep for endpoints, schemes, crypto, protobuf, the target focus.
4. Locate the relevant networking/crypto/protocol layer and explain what it does + exactly where (class/method/offset).
5. Report findings tersely, then propose the next probe or the patch. Save key findings with `baron-mem save`.

Install any missing tool rather than stopping.
