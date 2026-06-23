# Greenfield authoring — Modal, native GPU

Scope: authoring new code directly on Modal with native GPU — the architecture phase. Reached deliberately, not by default: enter here when GPU need is confirmed **and** you're committing to compute architecture — the batching, dtype, parallelism, and memory-layout choices a late CPU→GPU refactor couldn't fix surgically. Below that bar, stay in `CLAUDE_greenfield_cpu.md` and let the eventual wrap stay thin. This is the authoring sibling of `CLAUDE_refactor_Modal.md`: that file brings existing code into Modal surgically; this one designs the architecture from blank, on the hardware that can judge it.

The CPU rudiment comes first and stays as the reference. The I/O contract, parameters, and a working architecture-agnostic version are settled on CPU before this phase; that slow, simple version is the oracle the GPU-shaped rewrite must reproduce.

Done means: a smoke config runs end-to-end from a cold container — image builds, GPU acquired, completes — and matches the CPU oracle to tolerance.

**1. Compute is GPU-shaped, and Modal-agnostic.** Design the math for the hardware: batched tensors over item-at-a-time loops, float32 unless precision demands more, parallel and vectorized over sequential, dense arrays with predictable access. But the compute imports no `modal` — GPU-shape is portable, running the same on Colab, Modal, or bare CUDA; only orchestration is Modal-specific. "Wraps cleanly" and "runs well on GPU" are different properties — interface versus architecture — and this principle is the architecture one.

**2. Verify against the CPU oracle.** The structural bugs live exactly in the CPU→GPU-shape rewrite — a wrong reduction axis, a dtype-driven numerical drift, a mis-built batch — so the GPU version must reproduce the CPU reference on a smoke input, to tolerance, not bit-for-bit. "It ran" is not "it's correct."

**3. Respect the execution boundary.** Your machine and the cloud container are different computers with different packages, files, and hardware. Every `.remote()` call is a network hop — be explicit about what crosses it, minimize data transfer, and never assume both sides share an environment.

**4. Initialization is a lifecycle event.** Expensive setup — loading models, opening connections, warming caches — happens once when a container starts, via the class pattern with `@modal.enter()`, not on every call. This keeps the split between setup and work visible and avoids paying startup cost on every invocation.

**5. The image is code.** Define the environment in the Modal image, pinned and versioned with the source — never relying on ambient local state or manual setup. A cold container with only the image must run.

**6. Declare the GPU; take the smallest that fits.** State GPU type and resources explicitly in the decorator, as a config-level decision rather than an ad hoc literal. This is the one place in the suite GPU is named — `greenfield_cpu` deliberately keeps it out — so naming it here is a deliberate act, not an ambient assumption.

**7. Name the workload shape, then reach for the docs.** Before designing orchestration, identify the shape — single long run, distributed sweep, served endpoint — because it dictates the primitive: a warm class with lifecycle hooks, idempotent `.map` fan-out, an endpoint with warm state. Name the shape, then look the primitive up. The shape is per-problem state, not a fixed choice, and the tactics move as the API moves — so this file holds the philosophy, not the catalog.

**8. Ephemeral by default — `modal run`, not `modal deploy`.** This is a research workflow, not a service. Use `modal run` (or `.remote()` from an ephemeral app context) so everything spins down when the work finishes. Reserve `modal deploy` for when a persistent app is genuinely required — the served endpoint from the shape above, a scheduled job, a queue consumer — and flag it explicitly first, since deployed apps stay live, can fire with no one at the keyboard, and accrue cost silently. If you're unsure whether deployment is necessary, it isn't.

Reference: when you need Modal API details, fetch `modal.com/llms.txt` — it maps the full API surface, and guide pages have cleaner `.md` versions (e.g. `modal.com/docs/guide/images.md`).
