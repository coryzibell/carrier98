# carrier98

**Data modulation for a new era.**

carrier98 is an LLM-to-LLM wire protocol for structured data. It modulates serialized machine state into a parser-inert, copy-paste safe visual stream. It uses a 96-character safe-set (Box Drawing + Geometric Shapes) wrapped in Egyptian delimiter frames to ensure data survives the hostile environment of context windows and log files.

## Philosophy

The carrier wave carries the signal. Like DTMF tones encoding digits for telephone switches, carrier98 encodes structured data for LLM pipelines. It's data modulation - not different from hitting 0-9 on a phone keypad, just highly advanced.

**Design principles:**

1. **Parser-inert** - No regex, JSON parser, shell, or syntax highlighter will grab it by accident
2. **Display-safe** - Renders visibly on any terminal, editor, or browser (no invisible chars, no replacement boxes)
3. **Copy-paste debuggable** - See it in a log, copy it, decode it
4. **Self-describing** - Schema embedded in the payload
5. **Dense** - Smaller than JSON, smaller than TOON

*"The code of the Matrix isn't English - it's too dense. But a human can see patterns."*

## Quick Start

```bash
# Install the reference implementation
cargo install base-d

# Encode
echo '{"users":[{"id":1,"name":"alice"}]}' | base-d schema
# Output: 𓍹╣◟╥◕◝▰◣◥▟╺▖◘▰◝▤◀╧𓍺

# Decode
echo '𓍹╣◟╥◕◝▰◣◥▟╺▖◘▰◝▤◀╧𓍺' | base-d schema -d
# Output: {"users":[{"id":1,"name":"alice"}]}
```

## Documentation

- [Full Specification](https://coryzibell.github.io/carrier98)
- [Reference Implementation (base-d)](https://github.com/coryzibell/base-d)

## The Name

**98** = 96 alphabet characters + 2 frame delimiters

Also a nod to Windows 98 - a time when machines were learning to communicate with each other.

## License

MIT OR Apache-2.0
