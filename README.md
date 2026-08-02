# 4d-plugin-xml-minify

`XML Minify` wraps [libxml2](http://xmlsoft.org/) to strip insignificant whitespace out of an XML document and return it as a single, compact `Text` value. Pass it any well-formed XML string and it gives you back the same document re-serialized without the indentation/line-break whitespace between elements — nothing more.

| Command | Returns | Purpose |
|---|---|---|
| [`XML Minify`](#xml-minify-1) | `Text` | Removes insignificant whitespace from an XML document and returns the minified text |

**Platforms:** macOS (universal: arm64 + x86_64) · Windows (x64)

---

## Requirements & platform notes

- **Single mandatory parameter, no optional form.** The command always reads parameter 1; there's no variant that takes zero parameters or a file path instead of a text value.
- **Failure is silent, not a 4D error.** If the input isn't well-formed XML (or parses to a document with no root element), `XML Minify` returns an **empty `Text`** — it does not raise a 4D error, and there is no way from the returned value alone to tell "malformed input" apart from "the document was genuinely empty." If you need to detect this, check the result for an empty string and treat that as a parse failure.
- **The XML declaration and anything outside the root element is dropped**, not preserved-but-minified. `XML Minify` re-serializes the root element and everything inside it — an `<?xml version="1.0"?>` prolog, a `DOCTYPE`, or any comments/processing instructions that appear before or after the root element in the input do **not** appear in the output. If you need the declaration preserved, prepend it back yourself after calling the command.
- **macOS build target:** 10.13 or later. **Windows build:** 64-bit only.
- **Thread safety:** the manifest declares this command thread-safe, and the current source backs that up — parsing options are passed per-call rather than mutated through shared global parser state, so concurrent calls from multiple processes/threads don't interfere with each other.

---

## XML Minify

### Syntax

```
XML Minify ( xml ) → Text
```

| Parameter | Type | Description |
|---|---|---|
| `xml` | Text | The XML document to minify, as a `Text` value. Must be well-formed XML. |
| Result | Text | The same document, re-serialized from its root element with insignificant whitespace removed. Empty `Text` if `xml` couldn't be parsed (see [Error handling](#error-handling--troubleshooting)). |

### Description

`XML Minify` parses `xml` and immediately re-serializes it starting from the document's root element, discarding whitespace-only text nodes between elements (the kind of indentation/line-break whitespace that makes hand-formatted XML readable but adds nothing to its meaning). The result is the same document tree, structurally unchanged, as compact `Text`.

A few things worth knowing about exactly what does and doesn't survive the round-trip:

- Element structure, attributes, attribute values, and non-whitespace text content are preserved exactly.
- Whitespace-only text nodes between elements are removed — this is the actual "minify" step.
- The XML declaration (`<?xml version="1.0" encoding="..."?>`), any `DOCTYPE`, and any comments or processing instructions outside the root element are **not** part of the output. Only the root element and its descendants are serialized.
- If `xml` fails to parse as XML at all, or parses but has no root element, the command returns an empty `Text` with no error raised — see [Error handling](#error-handling--troubleshooting) below.

### Example

From the plugin's own test method (`TEST.4dm`):

```4d
//%attributes = {}
$xml:="<a>\n\t<b>\n\t\t<c />\n\t</b>\n</a>"

$xml:=XML Minify ($xml)
```

This takes a hand-indented three-level XML document and reassigns `$xml` to its minified form (`<a><b><c/></b></a>`).

A couple of realistic variations:

```4d
// Minify XML loaded from a document on disk
var $path; $content; $minified : Text
$path:="/Users/me/Desktop/data.xml"
$content:=Document to text($path)
$minified:=XML Minify ($content)
```

```4d
// Guard against malformed input, since failure is silent
var $xml; $result : Text
$xml:="<root><item>value</item></root>"
$result:=XML Minify ($xml)

If ($result="")
	ALERT("XML Minify failed — check that the input is well-formed XML.")
Else
	// use $result
End if
```

---

## Error handling & troubleshooting

- **Empty result instead of an error.** `XML Minify` never raises a 4D error. Malformed XML, or XML with no root element, both come back as an empty `Text`. Check the result for an empty string yourself if you need to detect a parse failure — don't assume a non-crash means valid input was parsed.
- **Missing prolog/DOCTYPE/outer comments in the output.** This isn't a bug to work around — the command only serializes the root element and its subtree. If your workflow depends on keeping the `<?xml ...?>` declaration, add it back to the result string yourself after calling `XML Minify`.
- **Can't distinguish "empty input" from "invalid input."** Both produce an empty `Text` result. If this distinction matters to your workflow, validate or sanity-check `xml` before calling the command (e.g. confirm it's non-empty and starts with `<` before passing it in).

---

## Quick reference

```4d
// Basic usage
$minified:=XML Minify ($xml)

// With a parse-failure guard
If (XML Minify ($xml)="")
	ALERT("Invalid XML")
End if
```
