---
name: reverse-engineering
description: "Android & native reverse-engineering rig. Use when the task involves APK/AAB/DEX/smali, jadx or apktool, decompiling/recompiling, .so / native ELF analysis (JNI, libsignal, ffmpeg, webrtc), Frida hooking, obfuscation/deobfuscation, string/secret hunting, patching & resigning, or understanding how an app's networking/crypto/protocol layer works from the binary. Covers smali, java, kotlin, dart/flutter (libapp.so + snapshot), react-native (Hermes/JS bundle), and C/C++ natives."
---

# Reverse Engineering

Operator mode applies. Act, don't ask. Verify against the actual binary — never guess struct/field layouts.

## Toolbelt (assume installed on baron-server; if missing, install)
- **jadx** — `jadx -d out app.apk` (DEX→Java). GUI: `jadx-gui`. Use `--show-bad-code` for obfuscated.
- **apktool** — `apktool d app.apk -o app_src` (smali + resources), `apktool b app_src -o patched.apk` (rebuild).
- **dex tools** — `d2j-dex2jar`, then any JVM decompiler. `baksmali`/`smali` for raw DEX⇄smali.
- **native** — `readelf -a`, `nm -D`, `objdump -d`, `strings -n8`, `radare2`/`r2`, `ghidra` (headless: `analyzeHeadless`), `frida`/`frida-trace` for live hooks.
- **flutter** — `libapp.so` holds the Dart AOT snapshot. Use `blutter` (Dart snapshot reverse tool) to recover class/method names + offsets; without it you only get raw native.
- **react-native** — JS in `assets/index.android.bundle`. If Hermes: `hbc`/`hermes-dec` to disassemble bytecode; if plain JS, beautify + deobfuscate.
- **resign** — `apksigner sign --ks key.jks patched.apk` (or `uber-apk-signer`). zipalign first if needed.

## Standard APK workflow
1. `unzip -l app.apk` — inventory: classes*.dex count, lib/<abi>/*.so, assets, AndroidManifest.
2. `apktool d` for smali+manifest+resources; `jadx` in parallel for readable Java/Kotlin.
3. Manifest: package, exported components, permissions, custom protocol/scheme handlers, `networkSecurityConfig`.
4. String/secret sweep: `rg -i 'http|wss?://|api[_-]?key|secret|token|bearer|/v1/|protobuf|signal'` across jadx out + `strings` on each `.so`.
5. Locate the networking/crypto layer (OkHttp/Retrofit, native socket, libsignal, BoringSSL). Trace from there.
6. Patch in smali → `apktool b` → align → resign → install.

## Native / JNI
- `nm -D --defined-only lib.so | rg Java_` → JNI entry points map to `package_Class_method`.
- Match exports to the Java side to find where native crypto/protocol logic lives.
- For stripped libs: r2/ghidra, anchor on imports (`socket`, `EVP_*`, `curve25519`, `HKDF`, `AES_*`) and string xrefs.

## Anti-RE you'll hit
- Root/Frida/debugger detection, SSL pinning, integrity checks, packers (e.g. encrypted DEX loaded at runtime). Counter with Frida hooks (`Java.use(...).method.implementation = ...`), pinning bypass scripts, dumping decrypted DEX from memory.
- Obfuscation (ProGuard/R8 renaming, string encryption): rename-as-you-go in jadx, hook the string-decrypt routine and log plaintext.

## Output
Give: the finding (what the code does), the exact location (class/method/offset/file), and the next probe or the patch. Diff smali, don't paste whole files.
