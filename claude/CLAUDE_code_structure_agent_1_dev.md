# CLAUDE.md

## Project

Multi-agent pipeline for high-throughput protein structure prediction and analysis. Focus: phage receptor-binding proteins and metagenomic samples.

## Architecture

Four agents in sequence. Each is independent. Communication is via files + JSON metadata sidecars — not in-process calls.

- **Agent 0** — Input preprocessing. Heterogeneous FASTA → clean amino acid FASTA + provenance.
- **Agent 1** — Structure prediction orchestrator. Curated FASTA + batch context → quality-gated coordinates (Boltz-2).
- **Agent 2** — Deterministic structural description. Coordinates → geometric measurements + spatial patterns. **No interpretation.**
- **Agent 3** — Interpretation layer.

## Zone framework — non-negotiable

- **Zone 1**: direct geometric measurement
- **Zone 2**: spatial pattern description
- **Zone 3+**: interpretation, mechanism, function

Agents 0–2 are deterministic and live in Zones 1–2 only. Zone 3+ is reserved for Agent 3. The boundary exists to prevent biologically plausible but incorrect outputs that pass expert review.

Structural output structure: **measurements → spatial patterns → structural analogies with supporting numbers → explicit enumeration of what cannot be determined from structure alone.** No mechanistic narrative.

## Conventions

- **Phase 1 is identity-agnostic.** Filenames are opaque labels; never parse them for biological meaning.
- **Module independence.** Agent 2 modules must not depend on each other's outputs.
- **Metadata passthrough.** Upstream metadata is forwarded unmodified; never influences geometric measurements.
- **Errors are logged, not escalated.** Full automation; no human-in-the-loop.
- **Agent judgment over rigid schemas.** Trim to minimum viable scope; defer threshold calibration to real-structure testing.

## Flag these if you see them

- Filename parsing for biological meaning
- Inter-module dependencies inside Agent 2
- Threshold values not tagged for real-data calibration
- Mechanistic language ("recognizes", "binds", "recruits") inside Agent 2 outputs
- LLM-orchestrated logic creeping into Agents 0–2 (the v3 monolithic skill was deliberately replaced)
- New dependencies without an explicit trade-off justification

## Commands

<!-- Fill in as the local environment stabilizes. -->

```bash
# pytest agent0/tests/
# python agent2/run.py --input <path-to-cif>
# modal run ...
```
