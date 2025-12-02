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
      <span class="meta-value">1.3</span>
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
@users┃id:int┃name:str┃active:bool▓◉1┃alice┃true▓◉2┃bob┃false▓◉3┃carol┃true
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
| `◈` | U+25C8 | White diamond containing black | Array element separator (flat) |
| `①②③...` | U+2460+ | Circled numbers | Nested depth levels |
| `∅` | U+2205 | Empty set | Null value |
| `▓` | U+2593 | Dark shade | Minified space |
| `[` `]` | U+005B U+005D | Square brackets | Metadata annotation |
| `,` `=` | U+002C U+003D | Comma, equals | Metadata key-value pairs |

These characters were chosen for:
- **Rarity**: Almost never appear in real data
- **Visibility**: Distinct at a glance
- **Single-token**: Most tokenizers encode each as one unit

> **Note on the field separator:** The heavy vertical `┃` (U+2503) is *not* the standard pipe `|` (U+007C). Compare them side by side: `┃` vs `|`. The heavy vertical is thicker and extends the full line height. This distinction matters—the standard pipe appears frequently in code and shell commands, while the heavy vertical is rare enough to serve as an unambiguous delimiter.

---

## Nesting with Circled Numbers

fiche handles nested data using circled number delimiters. Each number represents a depth level:

<div class="readout">
  <span class="readout-label">NESTED STRUCTURE</span>
@people┃name:str┃height:str┃films:@▓◉Luke▓Skywalker┃172①A▓New▓Hope①Empire▓Strikes▓Back▓◉C-3PO┃167①A▓New▓Hope
</div>

**Depth markers:**
- `◉` — Root record (depth 0)
- `①` — First nesting level
- `②` — Second nesting level
- `③④⑤...` — Deeper levels as needed

Unicode provides circled numbers ①-⑳ (1-20), with extended ranges ㉑-㊿ (21-50) for deeper structures.

### Multi-Level Nesting

<div class="readout">
  <span class="readout-label">TWO-LEVEL NESTING</span>
@films┃title:str┃director:str┃characters:@▓◉A▓New▓Hope┃George▓Lucas①Luke▓Skywalker②Tatooine②Jedi①Leia▓Organa②Alderaan②Rebel▓Leader▓◉Empire▓Strikes▓Back┃Irvin▓Kershner①Luke▓Skywalker②Dagobah②Jedi
</div>

The structure reads naturally: characters (`①`) belong to films, attributes (`②`) belong to characters.

### Why Circled Numbers?

We tested multiple approaches for nesting:

| Approach | Example | Haiku | Sonnet |
|----------|---------|-------|--------|
| Repeated arrows | `↳↳↳` | ✗ Failed | ✓ Passed |
| Circled numbers | `①②③` | ✓ Passed | ✓ Passed |

**Circled numbers won because:**
- Semantic meaning is baked into the symbol (`②` *means* depth 2)
- No counting required—models parse instantly
- Single token per depth marker
- Works on smaller, cheaper models (Haiku) without any format explanation

> **The test:** We gave Haiku raw fiche data with nested structures—no format explanation, no schema documentation. It parsed the data correctly on first attempt, even identifying relationships across nesting levels.

---

## Model Accuracy: fiche vs TOON

We benchmarked fiche against [TOON](https://github.com/toon-format/toon)'s published results using the same GitHub repositories dataset (top repositories by stars).

### Haiku Retrieval Accuracy

TOON's benchmark showed Haiku struggling with their whitespace-based format:

| Format | TOON Benchmark | fiche Benchmark |
|--------|----------------|-----------------|
| Accuracy | 59.8% (125/209) | **100%** (10/10 complex queries) |
| Format explanation | Required | None (cold parse) |

We tested fiche with 10 complex retrieval questions including aggregations, sorting, filtering, ratio calculations, and counting—all answered correctly by Haiku with zero format explanation.

### Why the Difference?

**TOON** uses whitespace indentation for structure. Smaller models struggle to:
- Track indentation depth accurately
- Distinguish significant whitespace from formatting
- Parse collapsed/minified content (impossible with TOON)

**fiche** uses explicit Unicode delimiters (`◉`, `┃`, `▓`, `①②③`). Models can:
- Count visible characters reliably
- Parse structure without inferring from spacing
- Handle minified single-string format identically to expanded

### Token Efficiency Comparison

Using TOON's GitHub repos benchmark data (50 records):

| Format | Tokens | vs JSON |
|--------|--------|---------|
| JSON | 6,757 | baseline |
| TOON | ~8,744 | +29% worse |
| fiche | 5,918 | **-12.4% better** |

On flat tabular data, fiche outperforms both JSON and TOON. TOON's strength is mixed nested structures—but fiche handles those too with circled number depth markers.

### The Full Picture

| Capability | fiche | TOON |
|------------|-------|------|
| Flat tabular | -12% tokens | +6% overhead |
| Nested structures | ✓ (①②③ depth) | ✓ (indentation) |
| Deep nesting (5+ levels) | ✓ stable | degrades |
| Minifiable | ✓ single string | ✗ whitespace required |
| Haiku accuracy | 100% cold | 59.8% |
| Human readability | good | better |

fiche fills an unclaimed niche: **nested + minifiable + token-efficient + small-model-friendly**.

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

### Header Metadata

When JSON has scalar fields alongside an array, fiche extracts them as header metadata:

```
@{root_key}[{key}={value},{key}={value}]┃{field}:{type}...
```

<div class="readout">
  <span class="readout-label">API RESPONSE WITH METADATA</span>
@students[class=Year▓1,school_name=Springfield▓High]┃id:str┃name:str┃grade:int▓◉A1┃alice┃95▓◉B2┃bob┃87▓◉C3┃carol┃92
</div>

**Equivalent JSON:**
```json
{
  "school_name": "Springfield High",
  "class": "Year 1",
  "students": [
    {"id": "A1", "name": "alice", "grade": 95},
    {"id": "B2", "name": "bob", "grade": 87},
    {"id": "C3", "name": "carol", "grade": 92}
  ]
}
```

**Rules:**
- Metadata keys are bare (no spaces)
- Metadata values use `▓` for spaces
- Keys sorted alphabetically for deterministic output
- Only extracted when JSON has scalar fields + exactly one array of objects

This pattern is common in API responses (`{count, next, results: [...]}`) where pagination or context metadata wraps the main data.

---

## Examples

### Simple Record Set

<div class="readout">
  <span class="readout-label">FICHE FORMAT</span>
@crew┃id:int┃name:str┃role:str▓◉1┃Glenn┃Pilot▓◉2┃Carpenter┃Pilot▓◉3┃Johnson┃Computer
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
@missions┃name:str┃crew:str[]▓◉Mercury-Atlas▓6┃Glenn▓◉Apollo▓11┃Armstrong◈Aldrin◈Collins
</div>

### With Nulls

<div class="readout">
  <span class="readout-label">FICHE FORMAT</span>
@telemetry┃timestamp:int┃altitude:float┃notes:str▓◉1621234567┃408.5┃∅▓◉1621234568┃∅┃Signal▓lost▓◉1621234569┃412.1┃Reacquired
</div>

### Embedded Content

fiche handles embedded JSON, code, or any content without escaping:

<div class="readout">
  <span class="readout-label">FICHE FORMAT</span>
@logs┃level:str┃message:str▓◉error┃Failed▓to▓parse▓{"key":▓"value"}▓◉info┃User▓said▓"hello,▓world"▓◉debug┃Multiline▓content▓works
</div>

The heavy pipe `┃` delimiter is rare enough that typical content passes through unchanged.

---

## Context Efficiency

| Content Type | JSON | fiche | Reduction |
|--------------|------|-------|-----------|
| 10 simple records | 450 bytes | 280 bytes | 38% |
| 100 records | 4,200 bytes | 2,100 bytes | 50% |
| Nested with arrays | 890 bytes | 520 bytes | 42% |
| **SWAPI people (5 records, nested)** | **1,117 bytes** | **725 bytes** | **35%** |

### Real-World Benchmark: Star Wars API

Tested against actual SWAPI data with nested arrays (films, vehicles, starships per character):

<div class="readout">
  <span class="readout-label">SWAPI IN FICHE</span>
@people┃name:str┃height:str┃mass:str┃films:@┃vehicles:@┃starships:@◉Luke▓Skywalker┃172┃77①film/1①film/2①film/3①film/6①vehicle/14①vehicle/30①starship/12①starship/22◉C-3PO┃167┃75①film/1①film/2①film/3①film/4①film/5①film/6∅∅◉Darth▓Vader┃202┃136①film/1①film/2①film/3①film/6∅①starship/13
</div>

Note the `▓` (U+2593) replacing spaces in names—this prevents whitespace mangling in terminals and parsers while remaining visually distinct. Models read it as a space naturally.

**Result:** 35% reduction, parsed correctly by Haiku with zero format explanation.

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

# JSON → fiche (minified single line)
echo '{"users":[{"id":1,"name":"alice"}]}' | base-d fiche -m

# fiche → JSON (works with both formats)
echo '@users┃id:int┃name:str▓◉1┃alice' | base-d fiche -d

# Pretty-print JSON output
base-d fiche -d -p < data.fiche
```

### Library

```rust
use base_d::{encode_fiche, encode_fiche_minified, decode_fiche};

let json = r#"{"users":[{"id":1,"name":"alice"}]}"#;
let fiche = encode_fiche(json)?;           // multi-line
let minified = encode_fiche_minified(json)?; // single line
let restored = decode_fiche(&fiche, false)?;
```

---

## Reference

**Specification version:** 1.0

**Implementation:** [base-d](https://github.com/coryzibell/base-d) (Rust)

**Related:** [carrier98](/) — opaque wire format for maximum density
