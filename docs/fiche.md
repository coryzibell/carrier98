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
      <span class="meta-value">1.5</span>
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
@┃video჻id:str┃video჻title:str┃tags჻0:str┃tags჻1:str┃tags⟦⟧:str▓◉dQw4w9WgXcQ┃Never▓Gonna▓Give▓You▓Up┃music┃80s┃∅
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
| `჻` | U+10FB | Georgian comma | Nested path separator |
| `◈` | U+25C8 | Diamond in diamond | Primitive array element separator |
| `∅` | U+2205 | Empty set | Null value |
| `▓` | U+2593 | Dark shade | Minified space |
| `⟦` `⟧` | U+27E6 U+27E7 | Mathematical brackets | Array type markers |
| `,` `=` | U+002C U+003D | Comma, equals | Metadata key-value pairs |

These characters were chosen for:
- **Rarity**: Almost never appear in real data
- **Visibility**: Distinct at a glance
- **Single-token**: Most tokenizers encode each as one unit

> **Note on the field separator:** The heavy vertical `┃` (U+2503) is *not* the standard pipe `|` (U+007C). Compare them side by side: `┃` vs `|`. The heavy vertical is thicker and extends the full line height. This distinction matters—the standard pipe appears frequently in code and shell commands, while the heavy vertical is rare enough to serve as an unambiguous delimiter.

---

## Array Flattening

fiche handles nested structures and arrays by flattening them into indexed paths using the Georgian comma `჻` as the path separator.

### Primitive Arrays (Inline)

Arrays of primitives (strings, numbers, booleans) use the diamond separator `◈` for compact inline representation:

<div class="readout">
  <span class="readout-label">PRIMITIVE ARRAY</span>
@┃tags:str⟦⟧
◉music◈80s◈classic
</div>

**Equivalent JSON:**
```json
{
  "tags": ["music", "80s", "classic"]
}
```

The `tags:str⟦⟧` schema declares an array of strings. Values are joined with `◈`. This is more compact than indexed paths for primitive arrays.

### Arrays of Objects (Indexed Paths)

Arrays containing objects use indexed paths with the Georgian comma `჻`:

<div class="readout">
  <span class="readout-label">ARRAY OF OBJECTS</span>
@┃video჻id:str┃video჻title:str┃tags:str⟦⟧┃comments჻0჻author:str┃comments჻0჻text:str┃comments⟦⟧:str
◉dQw4w9WgXcQ┃Never▓Gonna▓Give▓You▓Up┃music◈80s┃alice┃Great!┃∅
</div>

**Equivalent JSON:**
```json
{
  "video": {
    "id": "dQw4w9WgXcQ",
    "title": "Never Gonna Give You Up"
  },
  "tags": ["music", "80s"],
  "comments": [
    {
      "author": "alice",
      "text": "Great!"
    }
  ]
}
```

### Nested Arrays

Arrays within arrays work naturally:

<div class="readout">
  <span class="readout-label">NESTED ARRAYS</span>
@┃comments჻0჻replies჻0჻author:str┃comments჻0჻replies჻1჻author:str┃comments჻1჻replies჻0჻author:str┃comments⟦⟧:str┃comments჻0჻replies⟦⟧:str┃comments჻1჻replies⟦⟧:str▓◉alice┃bob┃carol┃∅┃∅┃∅
</div>

**Path syntax:**
- `comments჻0` — First comment
- `comments჻0჻replies჻0` — First reply to first comment
- `comments჻0჻replies჻1` — Second reply to first comment

**Array markers:**
- `comments⟦⟧:str` — Top-level array marker
- `comments჻0჻replies⟦⟧:str` — Nested array marker

All array markers have `∅` values and exist solely for decoder metadata.

### Complex Nesting: Where fiche Shines

Real-world API responses often have deeply nested structures—arrays of objects containing arrays of objects. This is where many formats fail. fiche handles it naturally.

**Example: YouTube-style API response**

```json
{
  "video": {
    "id": "dQw4w9WgXcQ",
    "title": "Never Gonna Give You Up",
    "views": 1500000000
  },
  "comments": [
    {
      "author": "alice",
      "text": "Classic!",
      "replies": [
        {"author": "bob", "text": "Agreed!"},
        {"author": "carol", "text": "Never gets old"}
      ]
    },
    {
      "author": "dave",
      "text": "Still watching in 2024",
      "replies": ⟦⟧
    }
  ]
}
```

**fiche output:**

<div class="readout">
  <span class="readout-label">DEEPLY NESTED STRUCTURE</span>
@┃video჻id:str┃video჻title:str┃video჻views:int┃comments჻0჻author:str┃comments჻0჻text:str┃comments჻0჻replies჻0჻author:str┃comments჻0჻replies჻0჻text:str┃comments჻0჻replies჻1჻author:str┃comments჻0჻replies჻1჻text:str┃comments჻1჻author:str┃comments჻1჻text:str┃comments⟦⟧:str┃comments჻0჻replies⟦⟧:str┃comments჻1჻replies⟦⟧:str▓◉dQw4w9WgXcQ┃Never▓Gonna▓Give▓You▓Up┃1500000000┃alice┃Classic!┃bob┃Agreed!┃carol┃Never▓gets▓old┃dave┃Still▓watching▓in▓2024┃∅┃∅┃∅
</div>

**Key observations:**
- `comments჻0჻replies჻1჻author` — Four levels deep, completely unambiguous
- `comments჻1჻replies⟦⟧:str` — Empty array preserved via marker
- Every path is explicit—no counting indentation or tracking state
- **Round-trips perfectly**—decode produces identical JSON

**Cold parse test:** We gave this to Haiku with zero format explanation and asked: *"Who replied to the first comment?"* Answer: *"bob and carol"*. Correct.

This is the complexity level where whitespace-based formats break down. Fiche handles it because structure is encoded in the path, not inferred from layout.

### Try It Yourself: Model Cold Parse Test

Copy this fiche data and paste it to any LLM with the questions below. No format explanation needed.

<div class="readout">
  <span class="readout-label">COPY THIS</span>
@┃org჻founded:int┃org჻name:str┃teams჻0჻lead:str┃teams჻0჻members჻0჻name:str┃teams჻0჻members჻0჻skills:str⟦⟧┃teams჻0჻members჻1჻name:str┃teams჻0჻members჻1჻skills:str⟦⟧┃teams჻0჻name:str┃teams჻1჻lead:str┃teams჻1჻members჻0჻name:str┃teams჻1჻members჻0჻skills:str⟦⟧┃teams჻1჻name:str┃teams⟦⟧:str┃teams჻0჻members⟦⟧:str┃teams჻1჻members⟦⟧:str
◉2019┃Acme▓Corp┃alice┃bob┃rust◈python┃carol┃go┃Engineering┃dave┃eve┃figma◈css◈animation┃Design┃∅┃∅┃∅

Questions:
1. What skills does bob have?
2. Who leads the Design team?
3. How many members are on the Engineering team?
4. What is eve's third skill?
</div>

**Expected answers:**
1. rust, python
2. dave
3. 2 (bob and carol)
4. animation

If your model answers correctly with zero prompting about the format, fiche works for your use case.

### Try It Yourself: Tokenized Version

Same test, but with field names tokenized to runic characters. The token map is in the first line. Can your model still parse it cold?

<div class="readout">
  <span class="readout-label">COPY THIS (TOKENIZED)</span>
@ᚠ=org,ᚡ=founded,ᚢ=name,ᚣ=teams,ᚤ=lead,ᚥ=members,ᚦ=skills
ᚠ჻ᚡ:int┃ᚠ჻ᚢ:str┃ᚣ჻0჻ᚤ:str┃ᚣ჻0჻ᚥ჻0჻ᚢ:str┃ᚣ჻0჻ᚥ჻0჻ᚦ:str⟦⟧┃ᚣ჻0჻ᚥ჻1჻ᚢ:str┃ᚣ჻0჻ᚥ჻1჻ᚦ:str⟦⟧┃ᚣ჻0჻ᚢ:str┃ᚣ჻1჻ᚤ:str┃ᚣ჻1჻ᚥ჻0჻ᚢ:str┃ᚣ჻1჻ᚥ჻0჻ᚦ:str⟦⟧┃ᚣ჻1჻ᚢ:str┃ᚣ⟦⟧:str┃ᚣ჻0჻ᚥ⟦⟧:str┃ᚣ჻1჻ᚥ⟦⟧:str
◉2019┃Acme▓Corp┃alice┃bob┃rust◈python┃carol┃go┃Engineering┃dave┃eve┃figma◈css◈animation┃Design┃∅┃∅┃∅

Questions:
1. What skills does bob have?
2. Who leads the Design team?
3. How many members are on the Engineering team?
4. What is eve's third skill?
</div>

**Expected answers:** Same as above. If your model handles both versions identically, tokenization is safe for your use case.

Here's the equivalent JSON for comparison—same data, same structure:

```json
{"org":{"founded":2019,"name":"Acme Corp"},"teams":[{"lead":"alice","members":[{"name":"bob","skills":["rust","python"]},{"name":"carol","skills":["go"]}],"name":"Engineering"},{"lead":"dave","members":[{"name":"eve","skills":["figma","css","animation"]}],"name":"Design"}]}
```

> **Note on size:** For single complex records, fiche's schema overhead can exceed JSON. The savings come with multiple rows of similar structure—see [Context Efficiency](#context-efficiency) for benchmarks showing 30-50% reduction on typical datasets.

### Why This Hybrid Approach?

fiche uses two strategies for arrays:

| Array Type | Strategy | Example |
|------------|----------|---------|
| Primitives | Inline with `◈` | `tags:str⟦⟧` → `music◈80s◈classic` |
| Objects | Indexed paths | `comments჻0჻author:str` → indexed fields |

**Benefits:**
- Primitive arrays are compact—no schema bloat for simple lists
- Object arrays have explicit structure—no ambiguity about nesting levels
- Paths are self-documenting (`comments჻0჻replies჻1` reads naturally)
- Array boundaries are clear from path prefixes or `⟦⟧` markers
- Single token for each separator (Georgian comma `჻` and diamond `◈` are rare in content)

> **Note:** The Georgian comma `჻` (U+10FB) was chosen for its visibility and rarity. It's distinct at a glance and almost never appears in real data.

---

## Field Name Tokenization

For maximum compression, fiche can tokenize field names using single Unicode characters from ancient scripts. This reduces schema overhead while remaining regex-safe—no ASCII, no digits, no modern text patterns.

### Token Alphabet

Tokens are assigned from these Unicode ranges in order:

| Priority | Script | Range | Count | Plane |
|----------|--------|-------|-------|-------|
| 1 | Runic | U+16A0 – U+16F8 | 89 | BMP |
| 2 | Egyptian Hieroglyphs | U+13000 – U+1342F | 1072 | SMP |
| 3 | Cuneiform | U+12000 – U+123FF | 1024 | SMP |

**Why this order:**
- **Runic first**: Basic Multilingual Plane (BMP) means 2-byte UTF-8, better compatibility across systems
- **Hieroglyphs/Cuneiform overflow**: Supplementary Multilingual Plane (SMP) requires 4-byte UTF-8, used only for schemas with 90+ fields

89 runic characters cover the vast majority of real-world schemas.

### Token Map Syntax

The schema line includes a token map in the metadata section:

```
@ᚠ=video,ᚡ=id,ᚢ=title,ᚣ=comments,ᚤ=author,ᚥ=text,ᚦ=replies
ᚠ჻ᚡ:str┃ᚠ჻ᚢ:str┃ᚣ჻0჻ᚤ:str┃ᚣ჻0჻ᚥ:str┃ᚣ჻0჻ᚦ჻0჻ᚤ:str┃...
```

**Format:** `@` followed by comma-separated `token=fieldname` pairs, then the schema fields.

### Example: Tokenized vs Untokenized

**Untokenized (readable):**
```
@┃video჻id:str┃video჻title:str┃comments჻0჻author:str┃comments჻0჻text:str
◉dQw4w9WgXcQ┃Never▓Gonna▓Give▓You▓Up┃alice┃Classic!
```

**Tokenized (compact):**
```
@ᚠ=video,ᚡ=id,ᚢ=title,ᚣ=comments,ᚤ=author,ᚥ=text
ᚠ჻ᚡ:str┃ᚠ჻ᚢ:str┃ᚣ჻0჻ᚤ:str┃ᚣ჻0჻ᚥ:str
◉dQw4w9WgXcQ┃Never▓Gonna▓Give▓You▓Up┃alice┃Classic!
```

Data rows are unchanged—only schema field names are tokenized.

### Why Ancient Scripts?

| Requirement | Solution |
|-------------|----------|
| No ASCII collision | Ancient scripts contain no Latin, digits, or punctuation |
| No regex match | `\w`, `[a-zA-Z0-9]`, `\d` won't match runic/hieroglyphs |
| No delimiter collision | Scripts don't include `┃`, `჻`, `◈`, `⟦⟧`, etc. |
| Model parseability | Tested: Haiku parses tokenized schemas cold with 100% accuracy |
| Visual distinction | Immediately obvious these are tokens, not data |

### Tokenization Rules

1. **Collect unique field names** from flattened schema paths
2. **Assign tokens** starting at ᚠ (U+16A0), incrementing through runic
3. **Overflow to hieroglyphs** at 𓀀 (U+13000) if runic exhausted
4. **Overflow to cuneiform** at 𒀀 (U+12000) if hieroglyphs exhausted
5. **Exclude from tokenization:**
   - Array indices (remain as digits: `჻0჻`, `჻1჻`)
   - Type annotations (`:str`, `:int`, etc.)
   - Array markers (`⟦⟧`)

### Constraints

**DO NOT use as tokens:**
- ASCII characters (0x00–0x7F)
- Digits in any script
- fiche delimiters (`◉`, `┃`, `჻`, `◈`, `∅`, `▓`, `⟦`, `⟧`)
- Common Unicode punctuation

**Numeric tokens break parsing.** Array indices use digits (`჻0჻`, `჻1჻`), so numeric tokens like `1=field` create ambiguity in paths like `1჻0჻2`—is `1` a token or index? Ancient scripts avoid this entirely.

### Implementation Flag

Tokenization is optional. Reference implementation supports:

```bash
# Tokenized (default for compact output)
base-d fiche input.json

# Untokenized (readable, debugging)
base-d fiche --no-tokenize input.json
```

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

**fiche** uses explicit Unicode delimiters (`◉`, `┃`, `▓`, `჻`). Models can:
- Count visible characters reliably
- Parse structure without inferring from spacing
- Handle minified single-string format identically to expanded
- Follow explicit path-based nesting (`comments჻0჻replies჻1`)

### Token Efficiency Comparison

Using TOON's GitHub repos benchmark data (50 records):

| Format | Tokens | vs JSON |
|--------|--------|---------|
| JSON | 6,757 | baseline |
| TOON | ~8,744 | +29% worse |
| fiche | 5,918 | **-12.4% better** |

On flat tabular data, fiche outperforms both JSON and TOON. TOON's strength is mixed nested structures—but fiche handles those too with path flattening.

### The Full Picture

| Capability | fiche | TOON |
|------------|-------|------|
| Flat tabular | -12% tokens | +6% overhead |
| Nested structures | ✓ (path flattening) | ✓ (indentation) |
| Deep nesting (5+ levels) | ✓ stable | degrades |
| Minifiable | ✓ single string | ✗ whitespace required |
| Haiku accuracy | 100% cold | 59.8% |
| Human readability | good | better |

fiche fills an unclaimed niche: **nested + minifiable + token-efficient + small-model-friendly**.

### fiche vs JSON Parsing Parity

We tested whether fiche degrades model comprehension compared to raw JSON. Using 10 users with nested objects (address, company, geo coordinates) plus metadata:

| Format | Size | Parsing Errors | Reasoning Errors |
|--------|------|----------------|------------------|
| JSON | 4,170 bytes | 0 | 2 |
| fiche | 3,117 bytes | 0 | 2 |

Both formats produced identical parsing results. The reasoning errors (finding minimum values, pattern matching) occurred on *both* formats with different wrong answers—indicating model reasoning limits, not format comprehension issues.

**Conclusion:** fiche parses at parity with JSON while being 25% smaller.

[Try it yourself →](benchmarks/)

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
- `{type}⟦⟧` — Array of type (e.g., `str⟦⟧`, `int⟦⟧`)

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
@┃missions჻name:str┃missions჻crew჻0:str┃missions჻crew჻1:str┃missions჻crew჻2:str┃missions⟦⟧:str┃missions჻crew⟦⟧:str▓◉Mercury-Atlas▓6┃Glenn┃∅┃∅┃∅┃∅▓◉Apollo▓11┃Armstrong┃Aldrin┃Collins┃∅┃∅
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
@┃people჻0჻name:str┃people჻0჻height:str┃people჻0჻films჻0:str┃people჻0჻films჻1:str┃people჻0჻vehicles჻0:str┃people჻1჻name:str┃people჻1჻films჻0:str┃people⟦⟧:str┃people჻0჻films⟦⟧:str┃people჻0჻vehicles⟦⟧:str┃people჻1჻films⟦⟧:str▓◉Luke▓Skywalker┃172┃film/1┃film/2┃vehicle/14┃C-3PO┃film/1┃∅┃∅┃∅┃∅
</div>

Note the `▓` (U+2593) replacing spaces in names—this prevents whitespace mangling in terminals and parsers while remaining visually distinct. Models read it as a space naturally.

**Result:** 35% reduction, parsed correctly by Haiku with zero format explanation. Path-based nesting makes relationships explicit.

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
