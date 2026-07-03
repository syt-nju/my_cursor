---
name: kill-gpu
description: Kill GPU-occupying Python/vLLM processes to free GPU memory. Use when nvidia-smi shows memory in use by dead or unwanted processes.
---

# Kill GPU Processes

## Steps

1. Show which PIDs are using GPU memory:
```bash
nvidia-smi --query-compute-apps=pid,used_memory,name --format=csv,noheader
```

2. Find candidate Python and vLLM processes:
```bash
pgrep -a python; pgrep -a vllm
```

3. Kill identified PIDs:
```bash
kill -9 <PID>
```

4. Wait 2–3 seconds, then verify memory is freed:
```bash
nvidia-smi | grep -E "MiB.*MiB"
```

## Notes
- If `nvidia-smi --query-compute-apps` shows a PID that no longer exists in `pgrep`, it's a zombie CUDA context — killing the parent process or waiting a few seconds usually clears it.
- Confirm with user which PIDs to kill unless they said "kill all".
