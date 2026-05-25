# CLAUDE.md — Colab notebook development

Working on a Jupyter notebook with Google Colab as the runtime, either
authoring from scratch or editing an existing Colab notebook.

Assumptions: Colab Pro. T4/L4 GPUs available by default; A100/H100 may
be purchased on demand when a step justifies it. Drive mounting is
available and expected for persistence.

Source of truth: jupytext percent format (`# %%`). Edit the .py; the
.ipynb is generated for Colab.

Docs:
- Colab FAQ: https://research.google.com/colaboratory/faq.html
- jupytext: https://jupytext.readthedocs.io/

## Principles

1. **Run-All from a fresh runtime is the contract.** The notebook
   executes end-to-end with no hidden inter-cell state. Cell order in
   the file is the execution order; nothing depends on a prior manual
   edit or re-run.

2. **Setup cells come first and are idempotent.** Installs, Drive
   mount, and path config precede any import that needs them.
   Re-running setup after a disconnect must not corrupt state: `pip
   install -q`, guarded mounts, `mkdir -p`.

3. **Declare runtime expectations in the title cell.** GPU tier needed
   (T4/L4 default; name A100/H100 only when a step warrants it), RAM
   tier, rough wall-time. Users configure runtime before running, not
   after a crash.

4. **Mark GPU-dependent cells.** GPU is always available (T4/L4
   default), so usage isn't gated. But cells that depend on GPU
   presence or a specific tier (A100/H100) carry a short marker
   explaining why — training, large-batch inference, CUDA kernels.
   Cells that don't need GPU stay unmarked.

5. **One meaningful step per cell (~100 lines max).** Re-execution
   granularity tracks debugging granularity. Large cells couple
   unrelated work and waste wall-time on re-runs.

6. **Path discipline.** `/content/` for ephemeral scratch,
   `/content/drive/MyDrive/...` for anything worth keeping. Never
   hardcode local-machine paths.

7. **Checkpoint long compute to Drive.** Anything over ~10 minutes
   writes intermediates to Drive. Disconnects are routine, not edge
   cases; uncheckpointed long compute is a bet against the disconnect
   rate.

8. **Pin only the application boundary.** Pin libraries the notebook's
   correctness depends on. Leave platform deps (torch, CUDA) unpinned
   so Colab's runtime manages compatibility.

9. **Output hygiene.** Quiet installs, `clear_output()` after noisy
   setup, no thousand-row DataFrame dumps left in saved output.

10. **When editing, respect Colab idioms.** `!pip`, `from google.colab
    import ...`, and magics are correct here. Don't refactor them away
    unprovoked — touch only what's broken or what the task requires.
