---
name: protocols
description: "Messaging & realtime protocol analysis and implementation. Use for libsignal / Signal protocol (X3DH, Double Ratchet, sessions, prekeys, sealed sender), WhatsApp internals (Noise XX handshake, binary XMPP/stanza framing, wa-proto / protobuf nodes, multi-device, app-state sync), protobuf / .proto authoring & decoding, WebSocket/wss + stanza/node/XML(stream) framing, GraphQL, MEX, media servers, and call/VoIP (WebRTC, SRTP/DTLS, signaling). Helps decode wire formats, write/patch .proto, and reason about E2EE message flows for interop and Baron's own packages."
---

# Protocols

Operator mode applies. Baron maintains `wa-proto-baron`, `baron-baileys-v2`, `whatsapp-rust-bridge-baron` — this is interop work on his own stack. Be concrete, verify against real captures/proto, don't hand-wave field layouts.

## libsignal / Signal Protocol
- **X3DH** key agreement: identity key + signed prekey + one-time prekey → initial shared secret. Know which keys sign which.
- **Double Ratchet**: DH ratchet (per round-trip) + symmetric chain ratchet (per message) → message keys, forward secrecy + post-compromise security.
- Primitives: Curve25519 (X25519 DH, Ed25519 sign), HKDF-SHA256 for KDF chains, AES-256-CBC + HMAC-SHA256 for message encryption.
- Sealed sender: sender identity hidden in an extra encryption layer.
- Reality check: E2EE means you can read/derive your **own** session state and your own device's keys; you cannot decrypt other parties' traffic without their private keys. Frame work as: client implementation, session debugging, interop, your own account's data.
- Reference: `libsignal` (Rust core + bindings). For native RE, anchor on `curve25519`, `HKDF`, `AES`, `signal_*` symbols.

## WhatsApp wire (for the baileys/wa-proto family)
- **Transport**: WebSocket to WA edge; **Noise protocol (XX handshake)** wraps the session → encrypted frames.
- **Framing**: length-prefixed binary; payload is WA's compact binary-XMPP ("binary nodes"): a tag dictionary tokenizes common strings, attributes + children form stanzas (`iq`, `message`, `presence`, `receipt`, `notification`).
- **Content**: message content is **protobuf** (the `wa-proto` schema: `WebMessageInfo`, `Message`, etc.). E2EE payloads inside are Signal-protocol ciphertext.
- **Multi-device**: per-device Signal sessions, sender-key for groups, **app-state sync** (LTHash-based) for contacts/chats/settings.
- Debug loop: capture frames → strip Noise → parse binary node → decode protobuf → if `message`, it's Signal ciphertext you decrypt with your session.

## protobuf
- `.proto` authoring/patching: keep field numbers stable, `reserved` removed fields, prefer `optional`/explicit presence when decoding unknown streams.
- Decode unknown bytes: `protoc --decode_raw < blob` to recover field numbers + wire types, then name fields by reverse engineering. Wire types: 0 varint, 1 i64, 2 len-delim, 5 i32.
- Generate: `protoc` (or `prost`/`prost-build` for Rust, `protobufjs` for Node, `protoc_plugin` for Dart).

## WebSocket / stanza / XML-stream / nodes
- wss framing, ping/pong keepalive, reconnect/backoff, resume tokens.
- XMPP-style stream: `<stream>` open, `iq`/`message`/`presence` stanzas, IDs + ack/receipts. WA's is the tokenized binary variant above.
- Map node ⇄ JSON ⇄ protobuf when bridging Rust↔Node.

## GraphQL / MEX
- Inspect queries/mutations + persisted-query hashes (apps often send only the hash + vars). Pull the operation map from the bundle. MEX = Meta's GraphQL-over-batch transport; treat as GraphQL docs wrapped in a batch envelope.

## Media / Call / VoIP
- **WebRTC**: ICE/STUN/TURN for NAT traversal, **DTLS** handshake → **SRTP** for media encryption, SDP for negotiation.
- Signaling rides the messaging channel (offer/answer/candidates as stanzas/protobuf). Media is separate (UDP/SRTP), often via relay/TURN.
- Media files: encrypted blobs on a CDN/media server, keyed per-message (key + SHA in the protobuf), `mediaKey` → HKDF → AES-CBC + MAC. Download → verify MAC → decrypt.

## Output
Wire layout / field map / flow diagram in text, then the decode or encode code (Rust/Node/Dart per target). Cite the real source: the .proto, the capture, or the symbol — not memory.
