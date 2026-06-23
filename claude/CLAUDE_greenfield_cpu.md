# Greenfield authoring — plain Python, CPU-local

Scope: authoring new plain `.py` scientific-computing code on a CPU-only local machine. No CUDA is available locally — the office workstation has no GPU, and the Apple Silicon laptops are treated as CPU because all GPU work runs on Modal/Colab. This is the **upstream** artifact in the suite: code written here is later wrapped by Modal or decomposed into a Colab notebook, so it must leave clean seams for that adaptation rather than bake in any one platform.

Environment is **uv**. Success criterion: the script runs end-to-end from a fresh `uv` environment, on CPU, in a smoke configuration (small N, few steps). The bar is "it ran end-to-end from a clean env on CPU" — not "the part I touched worked."

**1. uv owns the environment.** Declare dependencies in `pyproject.toml` and run through `uv run`. Never `pip install` into the ambient interpreter or assume a global/conda environment — the lockfile is also what a Modal image build mirrors.

**2. CPU-portable by construction.** Resolve the device once — `device = "cuda" if torch.cuda.is_available() else "cpu"` — and route every tensor and model through it. Never hardcode `.cuda()` or a `"cuda"` string, and do not add an `mps` branch: locally this always yields CPU, and CPU (not MPS) is the faithful proxy for the CUDA you actually deploy to.

**3. Repo-relative paths via pathlib.** Resolve files from a known anchor with `pathlib`; no absolute paths, no OS-specific separators. The same file must run unedited across Windows, macOS, and Linux.

**4. Config at the top.** Paths, hyperparameters, N, step counts, and seeds live in one config block or dataclass that compute reads from — not as literals buried in function bodies. This is the surface Modal parameterizes and a Colab notebook exposes as its parameters cell.

**5. Compute in functions; the module stays import-safe.** Put logic in functions and guard execution with `if __name__ == "__main__":`. Importing the module must not trigger computation — the Modal wrapper imports it.

**6. Scale by config, not by editing code.** The same file reaches a 30-second smoke run and a full run by changing config values or flags, never by commenting blocks in and out. This is what makes the end-to-end CPU smoke run a repeatable check rather than a one-off.

**7. No platform idioms.** No notebook magics (`%autoreload`, `!pip`), no `google.colab` or `IPython` imports, no `modal` decorators. Plain Python keeps this the clean upstream the converters can consume.

**8. Seeds are explicit.** Set and surface RNG seeds (Python, NumPy, torch) from config rather than relying on implicit global state — the one general practice the research context makes non-negotiable for reproducible runs.

Reference: uv — https://docs.astral.sh/uv
