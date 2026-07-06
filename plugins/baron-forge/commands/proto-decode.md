---
description: Decode an unknown protobuf/binary blob or wire capture
argument-hint: <path-to-blob-or-hex> [context e.g. "wa message node"]
allowed-tools: Bash, Read
---
Use the protocols skill. Decode: `$ARGUMENTS`

1. If protobuf-ish: `protoc --decode_raw` to recover field numbers + wire types.
2. Map fields to meaning from context (WA node / Signal / media). Wire types: 0 varint, 1 i64, 2 len-delim, 5 i32.
3. If it's a framed wire capture (ws/Noise/stanza): identify framing → strip → decode inner protobuf.
4. Output a clean field map (number → type → inferred name → value) and, if useful, a `.proto` stub. Don't invent field names — mark inferred ones as inferred.
