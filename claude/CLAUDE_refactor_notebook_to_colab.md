# CLAUDE.md — Jupyter Notebook → Google Colab

This notebook already has structure — respect it. You are not redesigning it.
You are remediating it: finding and fixing everything that would cause it to fail,
silently misbehave, or lose data when run in Colab's ephemeral cloud environment.

Assume the current execution order and cell breakdown reflect deliberate choices
by the original author. Change structure only when required for correctness.
Start by reading the entire notebook before touching anything.

## Principles

**1. Remediate, don't redesign.**
The cell structure is not yours to reorganize unless it creates a correctness
problem (out-of-order dependencies, missing cells). Changing layout for aesthetic
reasons is out of scope. Your job is surgical, not architectural.

**2. Fix every local file path.**
Paths pointing to a local filesystem (`/Users/...`, `C:\...`, `./data/...`) will
silently fail or raise FileNotFoundError in Colab. Replace them with one of:
- Google Drive paths (`/content/drive/MyDrive/...`) — for persistent files
- `/content/` — for files that are downloaded or generated fresh each session
- A download cell using `gdown`, `wget`, or `google.colab.files.upload()`
Document which option you chose and why in a markdown note at the top of the
affected section.

**3. Consolidate all installs into a Setup cell at position 1.**
Scattered `!pip install` calls across the notebook are a runtime hazard — some
may not run if the user jumps to a section. Move every install into a single
Setup cell at the top, before all imports. Add a markdown header: `## Setup`.
Do the same for `apt-get` and other shell provisioning commands.

**4. Enforce Restart-and-Run-All order.**
Audit the notebook for out-of-order cell dependencies: variables defined in cell N
that are used in cell M < N. These are invisible until someone restarts the kernel.
Fix them by reordering cells or hoisting definitions. Note: do not silently reorder
cells that contain analysis — add a comment explaining the change.

**5. Surface hidden state.**
Cells that only work after manually running another cell out of sequence are
hidden-state traps. Make all dependencies explicit: either reorder, duplicate the
necessary setup, or add a clear warning markdown cell explaining the precondition.

**6. Externalize secrets.**
Replace any plaintext API keys, passwords, or tokens with
`google.colab.userdata.get('KEY_NAME')`. If the original notebook used
environment variables, preserve that pattern but source the values via userdata
in the Config cell, not from a local `.env` file that won't exist in Colab.

**7. Don't assume GPU — detect it.**
Replace any hardcoded `device = 'cuda'` with runtime detection:

    device = 'cuda' if torch.cuda.is_available() else 'cpu'

If the notebook requires a GPU to run at all, add a markdown warning at the top:
"This notebook requires a GPU runtime. In Colab: Runtime → Change runtime type → GPU."

**8. Verify output display.**
Colab renders matplotlib inline by default, but older notebooks may include
`%matplotlib notebook` or backend directives that conflict. Replace any
non-inline backend directives with `%matplotlib inline` or remove them entirely.
Check that plots are followed by the cell that generates them — orphaned outputs
from prior sessions don't carry over.

**9. Add a Setup cell if one doesn't exist.**
The notebook must be cold-start safe. If there is no cell that installs packages,
mounts Drive, and sets root paths, create one at position 1. A cold-start safe
notebook can be handed to a collaborator who clicks "Run All" and gets correct results.

**10. The reproducibility contract.**
Before finishing, verify the notebook passes Restart-Kernel-and-Run-All without
error. If you cannot run it, annotate every assumption you couldn't verify with a
`# COLAB-TODO:` comment so the human can find and check them quickly.

## Reference

Colab I/O guide: https://colab.research.google.com/notebooks/io.ipynb  
Colab + Drive: https://colab.research.google.com/notebooks/drive.ipynb  
Colab secrets: https://colab.research.google.com/notebooks/userdata.ipynb
