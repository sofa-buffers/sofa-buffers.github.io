# SofaBuffers — Organization Landing Page 🛋️

> **Structured Objects For Anyone** — a compact, streaming-first binary serialization format for structured data.

This repository hosts the [SofaBuffers organization GitHub Pages site](https://sofa-buffers.github.io) — a single, dependency-free `index.html` (plain HTML, CSS and JS) that introduces the project.

## About SofaBuffers

SofaBuffers fills the gap between heavyweight schema formats and lightweight-but-limited ones:

- **JSON** is human-readable but carries significant overhead.
- **MessagePack / BSON / CBOR** are efficient but lack message definitions.
- **Protocol Buffers** are powerful but generate unnecessarily heavy code.
- **FlatBuffers** aren't tightly packed for slow interfaces.

It is tightly packed (varint + zig-zag encoding), fully **streamable** on both serialization and deserialization, self-describing via a TLV wire format, and runs everywhere from tiny MCUs to cloud services.

## Links

- 🏠 **Organization:** https://github.com/sofa-buffers
- 📖 **Documentation & spec:** https://github.com/sofa-buffers/documentation
- ⚙️ **Code generator:** https://github.com/sofa-buffers/generator
- **Core libraries:** [C/C++](https://github.com/sofa-buffers/corelib-c-cpp) · [C++](https://github.com/sofa-buffers/corelib-cpp) · [Rust](https://github.com/sofa-buffers/corelib-rs) · [Go](https://github.com/sofa-buffers/corelib-go) · [Python](https://github.com/sofa-buffers/corelib-py) · [TypeScript](https://github.com/sofa-buffers/corelib-ts) · [Java](https://github.com/sofa-buffers/corelib-java) · [C#](https://github.com/sofa-buffers/corelib-cs)

## Developing the page

It's a static site — no build step. Open `index.html` in a browser, or serve the folder:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

> *Built for embedded developers and cloud engineers 🚀*
> *"Because JSON is too fat, and Protobuf is too Google."*

Released under the [MIT License](LICENSE).
