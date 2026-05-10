# cm2html

A general-purpose CommonMark v0.31.2 → HTML renderer, specified
as a Commons Format module.

This repository ships only words: a module specification
(`commonsformat.md`), a verification suite (`evals.toml`), module
metadata (`commonsformat.toml`), and a license. There is no
executable code in this repository. Consumers generate their own
implementation by running this module's spec through a Commons
Format orchestrator with the code generator of their choice.

## Module structure

- `commonsformat.toml` — module metadata
- `commonsformat.md` — canonical prose with intent, constraints,
  threat model, and references
- `evals.toml` — verification suite (652 functional cases seeded
  from the upstream CommonMark v0.31.2 test corpus)
- `LICENSE` — Creative Commons Attribution-ShareAlike 4.0
  International

## Status

v0.1.0 — initial release. Functional cases only; deployable at D0.
Adversarial and generator-adversary cases for D1+ deployment are
planned for a follow-up release.

## Attribution

The functional eval cases are derived from the CommonMark Spec by
John MacFarlane, licensed CC-BY-SA-4.0. See the `<references>`
section in `commonsformat.md` for full attribution.

## License

CC-BY-SA-4.0 (whole module).
