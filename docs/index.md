---
layout: default
title: carrier98
---

# carrier98

**Data modulation for a new era.**

carrier98 is an LLM-to-LLM wire protocol for structured data. It modulates serialized machine state into a parser-inert, copy-paste safe visual stream. It uses a 96-character safe-set (Box Drawing + Geometric Shapes) wrapped in Egyptian delimiter frames to ensure data survives the hostile environment of context windows and log files.

---

## Philosophy

The carrier wave carries the signal. Like DTMF tones encoding digits for telephone switches, carrier98 encodes structured data for LLM pipelines. It's data modulation - not different from hitting 0-9 on a phone keypad, just highly advanced.

**Design principles:**

1. **Parser-inert** - No regex, JSON parser, shell, or syntax highlighter will grab it by accident
2. **Display-safe** - Renders visibly on any terminal, editor, or browser (no invisible chars, no replacement boxes)
3. **Copy-paste debuggable** - See it in a log, copy it, decode it
4. **Self-describing** - Schema embedded in the payload
5. **Dense** - Smaller than JSON, smaller than TOON

> *"The image translators work for the construct program - but there's way too much information to decode the Matrix. I don't even see the code. All I see is blonde, brunette, redhead."*

---

## The Name

**98** = 96 alphabet characters + 2 frame delimiters

Also a nod to Windows 98 - a time when machines were learning to communicate with each other.

---

## Wire Format

```
𓍹{payload}𓍺
```

| Component | Value | Description |
|-----------|-------|-------------|
| Frame start | `𓍹` (U+13379) | Egyptian hieroglyph, opening quotation mark |
| Frame end | `𓍺` (U+1337A) | Egyptian hieroglyph, closing quotation mark |
| Payload | Base-96 encoded | Binary data in carrier98 alphabet |

### Why Hieroglyphs?

These aren't arbitrary ancient characters - they're actual quotation marks from 5,000 years ago. Rope coils the Egyptians used to frame quoted text. Perfect semantic match for framing protocol data.

- **Stable**: Unicode 5.2 (2009), 15+ years proven
- **Parser-inert**: No parser on Earth looks for hieroglyphs
- **Visually distinct**: You see them, you know it's carrier98

---

## Alphabet

96 visually-safe Unicode characters:

| Range | Block | Count | Examples |
|-------|-------|-------|----------|
| U+2501–U+257B | Box Drawing (heavy/double) | 44 | `━` `┃` `┏` `╔` `║` |
| U+2588–U+259F | Block Elements | 11 | `█` `▀` `▄` `▌` `▐` |
| U+25A0–U+25FF | Geometric Shapes (solid) | 41 | `■` `▲` `●` `◆` `◉` |

**Properties:**
- Visually distinct (no confusables like 0/O or 1/l)
- Cross-platform safe (renders on Windows, macOS, Linux)
- No blanks (nothing looks like whitespace)
- Size-independent (readable at any font size)
- Monospace-friendly (consistent width in terminals)

**Excluded:**
- ASCII (parser-visible)
- Light box drawing (invisible at small sizes)
- Outline shapes (look empty)
- Emoji (inconsistent rendering)
- CJK (width issues)
- C0/C1 control characters
- Private use areas

---

## Binary Format

```
[compression][header][values]
```

### Compression Prefix (1 byte)

| Byte | Algorithm |
|------|----------|
| 0x00 | None |
| 0x01 | Brotli |
| 0x02 | LZ4 |
| 0x03 | Zstd |

### Header

```
[flags: u8]
[root_key?: varint string]    // if FLAG_HAS_ROOT_KEY
[row_count: varint]
[field_count: varint]
[field_types: nibble-packed]
[field_names: varint strings]
[null_bitmap?: bytes]         // if FLAG_HAS_NULLS
```

**Flags:**

| Bit | Flag | Description |
|-----|------|-------------|
| 0 | TYPED_VALUES | Per-value type tags (reserved) |
| 1 | HAS_NULLS | Null bitmap present |
| 2 | HAS_ROOT_KEY | Root key in header |

**Field types (4-bit tags):**

| Tag | Type | Encoding |
|-----|------|----------|
| 0 | U64 | Varint |
| 1 | I64 | Zigzag varint |
| 2 | F64 | 8 bytes IEEE 754 |
| 3 | String | Varint length + UTF-8 |
| 4 | Bool | Packed bits |
| 5 | Null | No bytes (bitmap only) |
| 6 | Array | Varint count + elements |
| 7 | Any | Type tag + value (reserved) |

### Values

Row-major order. No delimiters between values - the schema defines boundaries.

- **Integers:** LEB128 varint (MSB continuation bit)
- **Signed integers:** Zigzag encoding, then varint
- **Floats:** 8 bytes, IEEE 754 double, little-endian
- **Strings:** Varint length + UTF-8 bytes
- **Arrays:** Varint count + elements
- **Nulls:** No bytes (tracked in bitmap)

---

## Encoding Pipeline

```
Structured Data → IR → Binary → Compress → Base96 → Frame
```

**Intermediate Representation (IR)** is the pivot point. Parsers (JSON, YAML, CSV) produce IR. The binary packer consumes IR. This decouples input formats from the wire format.

---

## Example

**Input:**
```json
{"users":[{"id":1,"name":"alice"},{"id":2,"name":"bob"}]}
```

**Binary structure:**
```
Flags: 0x04 (HAS_ROOT_KEY)
Root key: "users"
Row count: 2
Field count: 2
Field types: [U64, String]
Field names: ["id", "name"]
Values: 1, "alice", 2, "bob"
```

**Wire format:**
```
𓍹╣◟╥◕◝▰◣◥▟╺▖◘▰◝▤◀╧𓍺
```

---

## Benchmarks

**What the LLM sees** (characters in context window):

| Payload | JSON | carrier98+brotli | Context Reduction |
|---------|------|------------------|-------------------|
| 1 KB | 1,055 | 403 | 62% smaller |
| 10 KB | 14,080 | 1,075 | 92% smaller |
| 100 KB | 245,081 | 7,885 | **97% smaller** |

At 100KB, you're packing 245,081 characters into 7,885. That's a **31x compression** in context space.

**Wire overhead:**

carrier98 trades bandwidth for context density. The display96 alphabet uses ~3 bytes UTF-8 per character, so wire size is ~1.5x larger than JSON+brotli. But the model doesn't see wire bytes - it sees characters. And those characters are 97% fewer.

| Payload | JSON + server brotli | carrier98 + server brotli | Wire Overhead |
|---------|---------------------|---------------------------|---------------|
| 1 KB | 346 bytes | 529 bytes | 1.5x |
| 10 KB | 794 bytes | 1,359 bytes | 1.7x |
| 100 KB | 6,022 bytes | 9,154 bytes | 1.5x |

**The trade-off:** Pay 1.5x bandwidth, get 31x context density.

---

## Limitations

1. **Root primitives**: Must be object or array (not `42` or `"hello"`)
2. **Null array elements**: Not supported (use object fields for nullable values)
3. **Single-element arrays**: Unwrap to single value
4. **Sparse arrays**: Normalized with nulls

---

## Implementations

| Language | Repository | Status |
|----------|------------|--------|
| Rust | [base-d](https://github.com/coryzibell/base-d) | Reference implementation |

---

## License

MIT OR Apache-2.0
