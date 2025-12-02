---
layout: fiche
title: fiche
---

<div class="doc-header">
  <div class="classification">MODEL-READABLE</div>
  <h1>fiche</h1>
  <p class="subtitle">structured data format for human-machine collaboration</p>
  <div class="doc-meta">
    <div class="meta-field">
      <span class="meta-label">Document</span>
      <span class="meta-value">FICHE-SPEC-001</span>
    </div>
    <div class="meta-field">
      <span class="meta-label">Revision</span>
      <span class="meta-value">1.0</span>
    </div>
    <div class="meta-field">
      <span class="meta-label">Status</span>
      <span class="meta-value"><span class="stamp stamp-go">APPROVED</span></span>
    </div>
  </div>
</div>

## Abstract

**fiche** is a model-readable structured data format. Where carrier98 is opaque—maximum density, the model shuttles data without parsing—fiche is transparent. The model reads the structure. The human reads the structure. Both work from the same document.

Named for microfiche: data compressed onto film, readable by machine and eye alike. The format that archived the space program.

<div class="readout">
  <span class="readout-label">EXAMPLE OUTPUT</span>
@users┃id:int┃name:str┃active:bool
◉1┃alice┃true
◉2┃bob┃false
◉3┃carol┃true
</div>

---

## Design Philosophy

Before computers verified trajectories, humans calculated them by hand. The machine checked the work. Both needed to read the same numbers.

fiche follows this principle: **one document, two readers.**

- The model parses structure with minimal tokens
- The human scans data without decoding
- No escaping—quotes, braces, newlines are just content
- Schema declared once, values positional

> When Katherine Johnson calculated orbital mechanics for John Glenn's flight, she used the same notation the IBM 7090 used. The format served both. fiche serves both.

---

## Delimiter Specification

| Symbol | Unicode | Name | Purpose |
|--------|---------|------|---------|
| `@` | U+0040 | At sign | Schema line start |
| `◉` | U+25C9 | Fisheye | Row start marker |
| `┃` | U+2503 | Heavy vertical | Field separator |
| `◈` | U+25C8 | White diamond containing black | Array element separator |
| `∅` | U+2205 | Empty set | Null value |

These characters were chosen for:
- **Rarity**: Almost never appear in real data
- **Visibility**: Distinct at a glance
- **Single-token**: Most tokenizers encode each as one unit

> **Note on the field separator:** The heavy vertical `┃` (U+2503) is *not* the standard pipe `|` (U+007C). Compare them side by side: `┃` vs `|`. The heavy vertical is thicker and extends the full line height. This distinction matters—the standard pipe appears frequently in code and shell commands, while the heavy vertical is rare enough to serve as an unambiguous delimiter.

---

## Format Structure

### Schema Declaration

```
@{root_key}┃{field}:{type}┃{field}:{type}...
```

The schema line begins with `@`, optionally followed by a root key (the JSON wrapper object name), then field definitions separated by `┃`.

**Supported types:**
- `int` — Integer values
- `float` — Floating point values
- `str` — String values
- `bool` — Boolean values (`true`/`false`)
- `{type}[]` — Array of type (e.g., `str[]`, `int[]`)

### Data Rows

```
◉{value}┃{value}┃{value}...
```

Each row begins with `◉`, followed by values in schema order, separated by `┃`.

---

## Examples

### Simple Record Set

<div class="readout">
  <span class="readout-label">FICHE FORMAT</span>
@crew┃id:int┃name:str┃role:str
◉1┃Glenn┃Pilot
◉2┃Carpenter┃Pilot
◉3┃Johnson┃Computer
</div>

**Equivalent JSON:**
```json
{"crew":[
  {"id":1,"name":"Glenn","role":"Pilot"},
  {"id":2,"name":"Carpenter","role":"Pilot"},
  {"id":3,"name":"Johnson","role":"Computer"}
]}
```

### With Arrays

<div class="readout">
  <span class="readout-label">FICHE FORMAT</span>
@missions┃name:str┃crew:str[]
◉Mercury-Atlas 6┃Glenn
◉Apollo 11┃Armstrong◈Aldrin◈Collins
</div>

### With Nulls

<div class="readout">
  <span class="readout-label">FICHE FORMAT</span>
@telemetry┃timestamp:int┃altitude:float┃notes:str
◉1621234567┃408.5┃∅
◉1621234568┃∅┃Signal lost
◉1621234569┃412.1┃Reacquired
</div>

### Embedded Content

fiche handles embedded JSON, code, or any content without escaping:

<div class="readout">
  <span class="readout-label">FICHE FORMAT</span>
@logs┃level:str┃message:str
◉error┃Failed to parse {"key": "value"}
◉info┃User said "hello, world"
◉debug┃Line contains
newlines
</div>

The heavy pipe `┃` delimiter is rare enough that typical content passes through unchanged.

---

## Context Efficiency

| Content Type | JSON | fiche | Reduction |
|--------------|------|-------|-----------|
| 10 simple records | 450 chars | 280 chars | 38% |
| 100 records | 4,200 chars | 2,100 chars | 50% |
| Nested with arrays | 890 chars | 520 chars | 42% |

fiche achieves 30-50% context reduction over JSON for typical structured data. For maximum compression, use carrier98.

---

## Escape Hatch

When data contains fiche delimiters (rare), wrap the field in carrier98 encoding:

```
◉normal value┃𓍹carrier98_encoded_value𓍺┃another value
```

The hieroglyph delimiters `𓍹...𓍺` signal encoded content. Decode the carrier98 payload to recover the original value.

---

## Relationship to carrier98

| Property | fiche | carrier98 |
|----------|-------|-----------|
| Model reads structure | Yes | No |
| Human reads structure | Yes | No |
| Context reduction | 30-50% | 90-97% |
| Use case | Working data | Shuttle data |
| Parsing required | Minimal | Full decode |

**Use fiche when:** The model needs to understand and transform the data.

**Use carrier98 when:** The model passes data through unchanged—maximum density, minimum tokens.

They are siblings. Same family, different jobs.

---

## Implementation

### CLI

```bash
# JSON → fiche
echo '{"users":[{"id":1,"name":"alice"}]}' | base-d fiche

# fiche → JSON
echo '@users┃id:int┃name:str
◉1┃alice' | base-d fiche -d

# Pretty-print JSON output
base-d fiche -d -p < data.fiche
```

### Library

```rust
use base_d::{encode_fiche, decode_fiche};

let json = r#"{"users":[{"id":1,"name":"alice"}]}"#;
let fiche = encode_fiche(json)?;
let restored = decode_fiche(&fiche, false)?;
```

---

## Reference

**Specification version:** 1.0

**Implementation:** [base-d](https://github.com/coryzibell/base-d) (Rust)

**Related:** [carrier98](/) — opaque wire format for maximum density
