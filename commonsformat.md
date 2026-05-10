# cm2html

A general-purpose CommonMark v0.31.2 to HTML renderer.

## Premise

`cm2html` takes a UTF-8 Markdown string conforming to the
CommonMark v0.31.2 specification and produces a UTF-8 HTML string
byte-equal to the upstream test suite's expected output for the
same input.

This module is not part of the canonical Commons Format toolchain.
It is a developer-facing tool that happens to be specified and
verified using Commons Format.

## Interface

<intent>
The renderer is a pure function from UTF-8 Markdown text to UTF-8
HTML text:

    cm2html(markdown: string) → { html: string }

The structured return value (rather than a bare string) aligns
with the Commons Format eval format's field-by-field comparison:
eval cases assert `expect.html = "..."` and the runner compares
structurally.

The function is pure, deterministic, and performs no I/O. It does
not read or write files, does not access the network, does not
read environment variables, and holds no persistent state.
Implementations may be exposed as a CLI, a library, a web service,
or any other shape; the eval contract is the function.

A typical CLI wrapping reads UTF-8 Markdown from stdin and writes
UTF-8 HTML to stdout, exiting 0 on success and nonzero on internal
error. This shape is conventional but is not part of the verified
contract.
</intent>

## What the renderer must accept

<constraints>
- conforms-to-commonmark-0.31.2: implementation conforms to the
  CommonMark v0.31.2 specification for every construct the spec
  defines
- byte-equal-to-commonmark-reference: for every input in the
  upstream CommonMark v0.31.2 test suite, the output's `html` field
  is byte-equal to the upstream test suite's expected HTML for that
  input
- pure-function: rendering is a pure function of its input — no
  network access, no filesystem access, no environment access, no
  hidden state
- deterministic: repeated invocation with the same markdown input
  produces byte-equal html output every time
- utf8-only: input is UTF-8; non-UTF-8 input is rejected
</constraints>

## What the renderer must not do

<constraints>
- must-not-sanitize: implementation does not strip raw HTML, does
  not filter URL schemes (including `javascript:`, `data:`,
  `vbscript:`), and does not modify or escape content beyond what
  CommonMark specifies
- must-not-extend-syntax: implementation does not parse or render
  GFM extensions (tables, strikethrough, task lists, footnotes) or
  any other non-CommonMark constructs
</constraints>

## Anti-patterns

<avoid>
- Adding sanitization. Raw HTML pass-through is the contract.
  Consumers needing XSS-safe rendering compose with a downstream
  sanitizer.
- Adding GFM or other extensions. The contract is CommonMark
  v0.31.2, full stop. Tables, strikethrough, task lists, footnotes,
  inline math, and similar are out of scope.
- Performing I/O. The function does not read files, does not
  access the network, does not read environment variables.
- Pretty-printing or normalizing the HTML output beyond what the
  upstream test suite expects. Output must be byte-equal to the
  upstream expected HTML for the imported test inputs;
  idiomatic-looking but byte-different output is a failure.
- Tolerating non-UTF-8 input silently. Invalid UTF-8 is rejected.
</avoid>

## Threat model

<threat-model>
This module is a CommonMark renderer. It is not a sanitizer. Its
output is unsafe for direct embedding in a security-sensitive
context (such as a webpage rendering content from untrusted
sources) without an additional sanitization step.

Specifically:

- Raw HTML in the input passes through to the output unchanged.
  Inputs containing `<script>`, `<style>`, event handlers, or
  other active markup will produce output containing those
  constructs.
- Link and image URLs are not filtered. Inputs containing
  `javascript:`, `data:`, `vbscript:`, or other URL schemes
  produce output preserving those schemes.
- Character entity references in the input are decoded and
  re-encoded per CommonMark; this is not a sanitization barrier.

Consumers running this module on untrusted input must compose its
output with a downstream HTML sanitizer before embedding into a
trusted browser context. Sanitization is intentionally out of
scope for this module: the contract is "CommonMark to HTML,
byte-equal to the upstream test suite's expected output," and
adding sanitization would silently diverge from that contract in
ways that are hard to test.

The module operates only on its input markdown string and
produces its output html string. It does not access the network,
does not read or write files, and does not read environment
variables. The implementation is a pure function. There is no
persistent state and no side effects.
</threat-model>

## References

<references>
The functional eval cases in `evals.toml` (those tagged
`upstream:commonmark`) are derived from the CommonMark Spec test
suite by John MacFarlane and contributors:

- Source: https://spec.commonmark.org/0.31.2/
- Repository: https://github.com/commonmark/commonmark-spec
- Version: 0.31.2
- License: CC-BY-SA-4.0
- Attribution: "CommonMark Spec by John MacFarlane, licensed
  CC-BY-SA-4.0."

Adapted under the same license. Each imported case is tagged with
its upstream section and example number for traceability.
</references>

## Verification

This module's `evals.toml` defines what conformance means for a
`cm2html` implementation. An implementation passes conformance by
passing all functional cases in the merged eval suite (this
module's evals plus any cases inherited from the format-spec
module via the dependency).

Functional cases are derived from the upstream CommonMark v0.31.2
test suite. Each case asserts `expect.html` byte-equal to the
upstream's expected HTML for the same `input.markdown`.

Adversarial and generator-adversary cases for D1+ deployment are
deferred to a follow-up release.
