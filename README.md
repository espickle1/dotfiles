# dotfiles

Personal collection of `CLAUDE.md` templates for use with [Claude Code](https://claude.com/claude-code). Each file captures the conventions, guardrails, and working style for a recurring kind of task — drop the relevant one in at the root of a project (or a subdirectory) so Claude picks up the right context automatically.

## Contents

All templates live under [`claude/`](claude/).

### Protein structure prediction pipeline

- **[`CLAUDE_code_structure_agent_overall.md`](claude/CLAUDE_code_structure_agent_overall.md)** — Top-level context for a multi-agent pipeline (Agents 0–3) doing high-throughput protein structure prediction on phage receptor-binding proteins. Encodes the non-negotiable zone discipline (measurement → pattern → interpretation) that keeps Agents 0–2 deterministic.
- **[`CLAUDE_code_structure_agent_1_dev.md`](claude/CLAUDE_code_structure_agent_1_dev.md)** — Working context for developing Agent 1 (Boltz-2 structure prediction orchestrator). Same architectural rules, scoped to the agent under active development.

### Google Colab

- **[`CLAUDE_greenfield_colab.md`](claude/CLAUDE_greenfield_colab.md)** — Authoring a new Colab notebook from scratch. Run-All-from-fresh-runtime contract, jupytext percent format as source of truth, GPU tier declared up front, checkpoint long compute to Drive.
- **[`CLAUDE_refactor_notebook_to_Colab.md`](claude/CLAUDE_refactor_notebook_to_Colab.md)** — Remediating an existing Jupyter notebook so it survives Colab's ephemeral runtime. Surgical, not architectural — fix paths, consolidate installs, externalize secrets, enforce Restart-and-Run-All order.
- **[`CLAUDE_refactor_Python_to_Colab.md`](claude/CLAUDE_refactor_Python_to_Colab.md)** — Restructuring a working Python script into a Colab notebook. Canonical section order, one step per cell, isolate expensive operations, Config cell replaces argparse.

### Performance & infrastructure

- **[`CLAUDE_refactor_pytorch_gpu.md`](claude/CLAUDE_refactor_pytorch_gpu.md)** — Thinnest-possible adaptations to improve GPU utilization of existing PyTorch code. Strict forbidden-changes list (no architecture edits, no hyperparameter changes, no `torch.compile`); mixed precision and memory-vs-compute trades are flagged as `# CLAUDE-TODO:` rather than applied.
- **[`CLAUDE_refactor_Modal.md`](claude/CLAUDE_refactor_Modal.md)** — Adapting a working codebase to run heavy compute on [Modal](https://modal.com). Wrap-don't-rewrite posture: Modal is the orchestration layer, business logic stays portable and never imports `modal`.

## Usage

Copy or symlink the relevant file into a project as `CLAUDE.md`:

```powershell
# Windows (PowerShell)
Copy-Item claude\CLAUDE_greenfield_colab.md <project>\CLAUDE.md
```

```bash
# macOS / Linux
cp claude/CLAUDE_greenfield_colab.md <project>/CLAUDE.md
```

Combine templates when a project spans more than one context (e.g. a Colab notebook that also does PyTorch GPU work) — merge the principles sections and keep the strictest rules from each.

## License

Apache 2.0 — see [`LICENSE`](LICENSE).
