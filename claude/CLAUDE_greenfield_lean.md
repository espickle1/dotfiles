# CLAUDE.md — Greenfield Lean 4 Project

> Standing instructions for Claude Code. This file is loaded automatically each
> session from the project root. Rename to `CLAUDE.md` when you drop it into the repo.

## Project

Greenfield Lean 4 + Mathlib formalization. Domain: **information theory applied
to biological sequences** (DNA alphabet `{A,T,G,C}`, protein alphabets, regulatory
motifs treated as messages over a noisy channel).

The goal is machine-verified theorems. A green `lake build` is the only definition
of "done" — not plausible-looking proofs.

## Build & Verify Commands

```bash
lake build              # build everything — run after EVERY change
lake build --wfail      # treat warnings (incl. sorry) as failures — use in CI
lake exe cache get      # pull prebuilt Mathlib oleans (run after dep changes)
lake env lean MyFile.lean   # check a single file
grep -rn "sorry" .      # list all unproven gaps
```

Always run `lake build` after editing. Never report a proof as complete until the
build passes with no errors.

## Core Rules

1. **Search before proving.** Before writing any proof, look for an existing
   Mathlib lemma: try `exact?`, `apply?`, `simp?` in a scratch block, or
   `grep -rn` the Mathlib source. Reinventing a Mathlib lemma is a mistake.
2. **Build incrementally.** After each definition or lemma, run `lake build` and
   fix all errors before moving on. Do not stack multiple unverified changes.
3. **`sorry` is allowed but must be flagged.** Every `sorry` gets a comment
   explaining exactly what mathematics is still needed. Never silently leave one.
4. **State first, prove second.** Write the theorem statement (with `sorry`) and
   confirm it type-checks before attempting the proof. A verified interface comes
   before a verified proof.
5. **Outline hard proofs as comments first.** For non-trivial results, write the
   mathematical strategy as comments, identify which steps Mathlib covers, then
   fill in tactics.
6. **Decompose.** Break large theorems into named helper lemmas. Prove the easy,
   Mathlib-adjacent ones first; isolate genuinely novel content into its own lemma.
7. **No hand-waving.** Do not claim a step "should work" — verify it with the build.

## Conventions

- **Imports:** import the narrowest Mathlib module that provides what's needed, not
  the whole library. Common ones for this project:
  - `Mathlib.Probability.ProbabilityMassFunction.Basic` — discrete distributions
  - `Mathlib.Analysis.SpecialFunctions.Log.Basic` — `Real.log`
  - `Mathlib.Algebra.BigOperators.Basic` — `Finset.sum` (`∑`)
  - `Mathlib.Data.Fintype.Basic` — finite alphabets
- **Naming:** follow Mathlib convention — `lower_snake_case` for theorems/lemmas,
  descriptive names (`entropy_nonneg`, `entropy_le_log_card`), `UpperCamelCase`
  for types and structures.
- **Notation:** prefer `∑ i, f i` (Finset big-operator) and `ℝ` over ASCII.
- **Docstrings:** every public definition and theorem gets a `/-- ... -/` docstring
  in plain mathematical English.

## File Layout

```
Myproject/
  Basic.lean          # core types: alphabets, sequences, distributions
  Entropy.lean        # Shannon entropy H(X) and its properties
  MutualInfo.lean     # mutual information I(X;Y)
  KL.lean             # KL divergence D(P‖Q)
  Sequence.lean       # biological-sequence-specific definitions
Myproject.lean        # top-level import aggregator
```

Keep one conceptual unit per file. Re-export from the top-level file.

## Proof-State Discipline

When working a proof, after each tactic step report the remaining goal so the
state is auditable. If a tactic closes the goal unexpectedly, double-check it
actually proved the intended statement and isn't closing a trivial/degenerate case.

## What "stuck" means here

If a proof resists automation after a few genuine attempts, **stop and flag it**
rather than papering over it. A resistant lemma is usually one of:
- genuinely novel mathematics → mark `sorry`, note it as a research contribution
- a missing Mathlib lemma → name the lemma we'd want and prove it separately
- a malformed statement → re-examine whether the theorem is even true as stated

Surface which of these it is. Do not fabricate a proof or weaken the statement
without saying so explicitly.

## Definition of Done

A task is complete when:
- [ ] `lake build` passes with no errors
- [ ] no unexplained `sorry` (each remaining one has a justifying comment)
- [ ] new theorems have docstrings
- [ ] statements were verified to type-check before proofs were attempted

## Out of Scope

- Do not modify Mathlib itself; build on top of it.
- Do not weaken or restate a theorem to make a proof pass without flagging it.
- Do not commit `.lake/` build artifacts (they belong in `.gitignore`).
