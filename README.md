# ECMAScript Proposal: `TOON` (Token-Oriented Object Notation)

**Stage:** 0  
**Champions:** *Seeking Champion*  
**Proposal Drafter:** Tiago Bertolo ([@bertolo1988](https://github.com/bertolo1988))  
**TOON Format Creator & Specification Author:** Johann Schopplich ([@johannschopplich](https://github.com/johannschopplich), [toon-format](https://github.com/toon-format))  

---

## Attribution & Acknowledgements

The **TOON (Token-Oriented Object Notation)** format and specification were designed and authored by **Johann Schopplich**.

- Official Website & Documentation: [toonformat.dev](https://toonformat.dev)
- Specification Repository: [github.com/toon-format/spec](https://github.com/toon-format/spec)
- Reference Implementation: [github.com/toon-format/toon](https://github.com/toon-format/toon)

*Note: This proposal repository is an effort to bring native `TOON` serialization and parsing capabilities into the ECMAScript standard. The proposal author (@bertolo1988) is solely drafting this proposal for TC39 consideration, and full credit for the underlying format, grammar, and design belongs to Johann Schopplich and the TOON community.*

---

## Status

This proposal is currently **Stage 0** in the [TC39 Process](https://tc39.es/process-document/).

## Motivation

With the proliferation of Large Language Models (LLMs), agentic workflows, and structured prompting, JavaScript and TypeScript runtimes (Node.js, Deno, Bun, and browsers) are the primary environments orchestrating interactions with model inference APIs.

In these systems, serializing structured data to send in prompts and parsing model outputs back into runtime objects is a dominant operation:
1. **Token Inefficiency of JSON:** While `JSON` is natively built into ECMAScript (`JSON.stringify` / `JSON.parse`), standard JSON is verbose. In collections of uniform objects, keys are duplicated for every single record, and punctuation delimiters (`{`, `}`, `"`, `:`, `,`) inflate token counts significantly (often by 30% to 60%).
2. **Cost and Context Window Limits:** LLM token usage incurs direct API costs and consumes limited context windows.
3. **Userland Bottlenecks:** Parsing and stringifying alternative data formats in userland JavaScript introduces performance and memory overhead. Having native, engine-level support (V8, JavaScriptCore, SpiderMonkey) for a token-efficient notation unlocks zero-overhead serialization for AI workloads.

**TOON** addresses this by providing a compact, human- and LLM-friendly serialization format that maps losslessly to the JSON data model, with special syntactic support for tabular collections of objects.

---

## Overview of TOON

TOON maintains full parity with the JSON data model (objects, arrays, strings, numbers, booleans, and null) while offering a significantly more compact syntax:

### Example: Uniform Array of Objects

**JSON (Verbose, high token count):**
```json
{
  "users": [
    { "id": 1, "name": "Alice", "role": "admin" },
    { "id": 2, "name": "Bob", "role": "member" }
  ]
}
```

**TOON (Compact, tabular representation):**
```
users[2]{id,name,role}:
  1,Alice,admin
  2,Bob,member
```

Features:
- **Lossless round-tripping** with the JSON data model (`TOON -> JSON -> TOON`).
- **Tabular syntax** for uniform arrays of objects.
- **Minimal delimiter noise:** strings only require quotes when ambiguous (containing delimiters, whitespace, or special characters).
- **Deterministic:** canonical output formatting ensures consistent tokenization and caching.

---

## Proposed API

The proposal introduces a built-in `TOON` namespace object in the global scope, mirroring the ergonomics and conventions of the global `JSON` object:

### `TOON.parse(text [, reviver])`

Parses a string containing TOON-formatted text and constructs the corresponding ECMAScript value or object.

```javascript
const toonText = `users[2]{id,name,role}:
  1,Alice,admin
  2,Bob,member`;

const data = TOON.parse(toonText);
// Result:
// {
//   users: [
//     { id: 1, name: "Alice", role: "admin" },
//     { id: 2, name: "Bob", role: "member" }
//   ]
// }
```

### `TOON.stringify(value [, replacer [, space [, options]]])`

Serializes an ECMAScript value into a TOON-formatted string.

```javascript
const payload = {
  users: [
    { id: 1, name: "Alice", role: "admin" },
    { id: 2, name: "Bob", role: "member" }
  ]
};

const toonString = TOON.stringify(payload);
```

Options may include serialization hints (e.g., controlling tabular threshold or indentation style).

---

## Comparison with Existing Formats

| Feature | JSON | YAML | CSV | TOON |
|---|---|---|---|---|
| **ECMAScript Native** | Yes (`JSON.*`) | No | No | **Proposed (`TOON.*`)** |
| **Token Efficiency** | Low | Medium | High | **High** |
| **Nested Structure Support** | Yes | Yes | No | **Yes** |
| **Lossless JSON Model** | Yes | Often | No | **Yes** |
| **Tabular Array Compression**| No | No | Yes | **Yes** |
| **Determinism / Simplicity** | High | Low | Medium | **High** |

---

## Open Questions & Discussion Points

1. **Options Parameter:** How should options (e.g., tabular detection heuristics, delimiters) be structured in `TOON.stringify` without diverging too far from `JSON.stringify` signature?
2. **Raw TOON / Streaming:** Should companion features like `TOON.rawTOON` or streaming parse APIs be included, or deferred to later proposals?
3. **Standard Grammar:** Coordinating formal ECMA-262 grammar specifications with the upstream [TOON specification](https://github.com/toon-format/spec).

---

## Next Steps

1. Solicit feedback from the JavaScript community and LLM tooling developers on [es.discourse.group](https://es.discourse.group).
2. Seek a champion among TC39 committee delegates to present at an upcoming TC39 plenary meeting for **Stage 1**.
