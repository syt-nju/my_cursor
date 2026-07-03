---
name: setup-training-env
description: Set up a model training/inference environment from scratch (uv venv + inference engine + training smoke test). Use when preparing to train or serve a model on a new machine/project, or when python/torch/vllm imports fail. Tool-agnostic flow; uv-first.
---

# Set Up a Model Training Environment

A generic, project-decoupled procedure for standing up a training/inference environment. **uv-first.** The end state is a reproducible `.sh` script + a note in the project's key docs.

## Required inputs (ask the user first)

- **Model path/name** — REQUIRED. Drives vllm version and transformers-compatibility decisions.
- **Inference engine** — sglang or vllm. If unspecified, use whichever the system already has.
- **Any special requirements** — user requests take top priority; follow them over these defaults.

## Fixed decisions (do not re-litigate)

- Build the venv with **uv**, place it on the **root local disk** (not a network mount like cephfs — faster reads), named **after the project** (e.g. `/<project-name>`).
- Create it with **`--system-site-packages`** so the venv inherits the system's big packages (torch/vllm/etc. — no re-download); install *only* conflicting packages inside the venv to override them (e.g. downgrade transformers). Do NOT hand-symlink individual packages.

## Steps

1. **Probe the system.** Establish what you're building on:
```bash
nvidia-smi
python -c "import torch, transformers; print('torch', torch.__version__, '| tfms', transformers.__version__)" 2>&1
python -c "import vllm; print('vllm', vllm.__version__)" 2>&1
python -c "import sglang; print('sglang', sglang.__version__)" 2>&1
```

2. **Create the venv** on the local root disk, project-named, inheriting system packages:
```bash
uv venv /<project-name> --python=3.12 --system-site-packages
source /<project-name>/bin/activate
```

3. **Pick the inference engine** — sglang or vllm per the user; default to whichever is already importable from step 1.

4. **Pin versions to the model.** Adjust vllm to a build that supports the target model. Resolve transformers conflicts — some training frameworks don't support transformers 5.x, so downgrade inside the venv only:
```bash
uv pip install "transformers<5"   # example; match the framework's requirement
```
Only install what the system lacks or what must be overridden — the rest is inherited.

5. **Verify — stage 1 (CUDA reachable):**
```bash
python -c "import torch; assert torch.cuda.is_available(); print('CUDA OK', torch.cuda.device_count(), 'gpus')"
```

6. **Verify — stage 2 (inference engine serves):** launch the chosen engine on the model and confirm the process comes up and answers one request. This is the intermediate milestone.

7. **Verify — final goal (training runs):** write a minimal training demo for the model and confirm it completes **one step** without error.

8. **Persist.** Freeze the working setup into a single `.sh` script at the project root (so the env is portable across machines/projects), and add a short "environment activation" note to CLAUDE.md / key docs (`source /<project-name>/bin/activate`).

## Notes
- Prefer overriding a single conflicting package in the venv over rebuilding the whole stack — `--system-site-packages` makes the system packages the default, the venv the override layer.
- Watch the torch ↔ flash-attn ABI match: pin torch so flash-attn's prebuilt wheel applies, otherwise it compiles from source.
- If the machine needs an outbound proxy for pip / ModelScope / HF, export it before installs (bake it into the `.sh` script).
- Distributed/multi-machine is out of scope — that's handled by ray once the single-node env passes.
