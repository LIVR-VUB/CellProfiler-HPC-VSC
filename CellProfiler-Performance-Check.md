Yes, absolutely — **you can monitor core usage in real-time or after the fact** while your SLURM job is running. Here's how to do it, step by step:

---

## ✅ 1. **While the Job is Running: Use `sstat` or `htop`**

### Option A: Monitor Live Job Stats with `sstat`

```bash
sstat -j <jobid> --format=AveCPU,AveRSS,MaxRSS,Elapsed
```

* `<jobid>` is the numeric SLURM job ID (you get this after `sbatch` submission).
* `AveCPU` shows average CPU usage
* `AveRSS` and `MaxRSS` show average and peak memory usage
* `Elapsed` shows how long the job has been running

> Example:

```bash
sstat -j 12345678 --format=AveCPU,AveRSS,MaxRSS,Elapsed
```

---

### Option B: Use `htop` on the Node (if you have SSH access to the compute node)

1. Find the node your job is running on:

```bash
squeue -u $USER
```

2. SSH into that node (if allowed):

```bash
ssh <nodename>
```

3. Run:

```bash
htop
```

Then press:

* `F2` to configure
* `F6` to sort by CPU usage
* Look for your job’s Python processes using up the cores

> If you don’t have direct access to compute nodes, stick to `sstat`.

---

## ✅ 2. **Inside Your Python Script: Print or Log CPU Info**

Add this at the top of `nf1_analysis.py`:

```python
import os
import multiprocessing

print(f"Total CPUs visible to job: {os.cpu_count()}")
print(f"MAX_WORKERS from env: {os.environ.get('MAX_WORKERS')}")
print(f"Number of plates to process: {len(plate_info_dictionary)}")
```

And right before launching `ProcessPoolExecutor`, log:

```python
from datetime import datetime
print(f"Launching ProcessPoolExecutor with {num_processes} workers at {datetime.now()}")
```

---

## ✅ 3. **Log CPU Allocation to Your SLURM Output**

Everything printed with `print()` in Python will go into your SLURM log file:

```bash
#SBATCH --output=logs/%x_%j.out
```

Check it with:

```bash
less logs/plate3_<jobid>.out
```

---

## ✅ 4. **After the Job Finishes: Use `sacct`**

```bash
sacct -j <jobid> --format=JobID,JobName%20,Elapsed,TotalCPU,MaxRSS,AveRSS
```

> Example:

```bash
sacct -j 12345678 --format=JobID,JobName%20,Elapsed,TotalCPU,MaxRSS,AveRSS
```

This gives:

* Total CPU time used
* Max and average memory
* Wall-clock time

---

## Summary Table

| Tool      | Use                         | Command Example    |
| --------- | --------------------------- | ------------------ |
| `sstat`   | Live job stats              | `sstat -j <jobid>` |
| `squeue`  | See node used               | `squeue -u $USER`  |
| `htop`    | Live CPU usage (if on node) | `htop`             |
| `sacct`   | After job finishes          | `sacct -j <jobid>` |
| `print()` | From within script          | View in log file   |

---

Would you like a diagnostic Python snippet that logs per-plate CPU start/end times as well? That can help trace bottlenecks.
