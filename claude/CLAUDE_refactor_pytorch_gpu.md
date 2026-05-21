# GPU optimization (PyTorch, refactoring posture)

Applies when adapting existing PyTorch code for better GPU utilization. The goal is the thinnest adaptation that improves performance — research logic is not up for revision.

## Forbidden changes

Do not:
- Modify model architecture (layer counts, widths, activations, normalization placement)
- Change hyperparameters (learning rate, batch size, optimizer choice, scheduler, weight decay)
- Introduce distributed training where there is none (DDP, FSDP, DeepSpeed)
- Use `torch.compile` — research code rarely satisfies its constraints, and failures are confusing
- Restructure the training loop or refactor across module boundaries

If a candidate change feels like it might cross these lines, flag with `# CLAUDE-TODO:` rather than applying.

## Principles

1. **Profile when the bottleneck is unclear.** Well-known safe wins below do not require prior measurement. For anything else, state which bottleneck class is being targeted (compute, memory bandwidth, data loading, host-device sync) and justify it. Tools: `nvidia-smi` for utilization, `torch.profiler` for kernel traces, host-side timing with explicit `torch.cuda.synchronize()` around the measured region.

2. **Remove unnecessary host-device syncs in hot loops.** `.item()`, `.cpu()`, `.tolist()`, `print(tensor)`, and Python-side branches on tensor values all force synchronization. Accumulate on device; transfer once outside the loop.

3. **Pair `non_blocking=True` on `.to(device)` with `pin_memory=True` in the DataLoader.** Safe, well-known win, no measurement needed.

4. **Tune `num_workers` and `persistent_workers=True` only when data loading is the bottleneck.** Confirm via profiling that GPU utilization dips during data fetches before changing worker counts.

5. **Add `fused=True` to AdamW/Adam when available.** Numerically equivalent in standard cases, modest speedup, no risk to results.

6. **Use `channels_last` memory format for 4D vision tensors.** Safe for conv-heavy models; apply to model (`.to(memory_format=torch.channels_last)`) and inputs.

7. **Wrap eval/inference paths in `torch.inference_mode()`** (or `torch.no_grad()` if `inference_mode` causes downstream issues). A missing wrapper is a bug, not a design choice.

8. **Flag mixed precision as `# CLAUDE-TODO:`, do not apply.** AMP/autocast changes numerics and can affect reproducibility against published results. Mark candidate sites with the proposed `torch.autocast` block in a comment; the user decides whether to enable.

9. **Flag memory-vs-compute trades as `# CLAUDE-TODO:`, do not apply.** Gradient accumulation, gradient checkpointing, and batch size adjustments all touch effective training dynamics. Identify the opportunity in a comment.

10. **When in doubt, do nothing.** Each change must be defensible as a thinnest-possible adaptation. If a proposed optimization requires more than a few lines, it belongs in a comment.

## Reference

PyTorch performance tuning: https://pytorch.org/tutorials/recipes/recipes/tuning_guide.html — fetch dynamically when specific techniques need verification.
