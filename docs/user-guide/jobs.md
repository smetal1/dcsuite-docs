# Running Jobs

DC Suite clusters run the **SLURM** workload manager, so you submit and track
batch jobs the way you would on any HPC cluster — from the login node, or
through the DC Suite API.

## From the login node

SSH to the cluster ([Access & Networking](access-networking.md)) and use the
standard SLURM commands:

```bash
sinfo                      # partitions and node states
sbatch train.sbatch        # submit a batch job
squeue                     # your queued/running jobs
scancel <jobid>            # cancel a job
sacct -j <jobid>           # accounting for a finished job
```

A minimal batch script:

```bash
#!/bin/bash
#SBATCH --job-name=train
#SBATCH --gpus=1
#SBATCH --time=01:00:00
#SBATCH --output=train-%j.out

srun python train.py
```

## Through the API

You can submit and track jobs without SSH:

| Action | Endpoint |
| --- | --- |
| Submit a job to a cluster | `POST /v1/clusters/{id}/jobs` |
| List a cluster's jobs | `GET /v1/clusters/{id}/jobs` |
| Get one job | `GET /v1/clusters/{id}/jobs/{jobID}` |
| Cancel a job | `DELETE /v1/clusters/{id}/jobs/{jobID}` |

```bash
curl -sS -X POST https://your-dc-suite.example.com/v1/clusters/$CLUSTER/jobs \
  -H "Authorization: Bearer $DCS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{ "script": "#!/bin/bash\n#SBATCH --gpus=1\nsrun python train.py\n" }'
```

The response carries the SLURM job id; poll the get endpoint (or `squeue`) for
state.

## Common SLURM flags

| Flag | Meaning |
| --- | --- |
| `--job-name=NAME` | Label shown in `squeue`/`sacct`. |
| `--gpus=N` / `--gpus-per-node=N` | GPUs total / per node. |
| `--nodes=N` | Number of nodes. |
| `--ntasks=N` / `--ntasks-per-node=N` | MPI ranks total / per node. |
| `--cpus-per-task=N` | CPU cores per task (dataloader workers). |
| `--time=HH:MM:SS` | Wall-clock limit (jobs are killed at the limit). |
| `--partition=NAME` | Target a specific partition/queue. |
| `--output=FILE` / `--error=FILE` | Redirect stdout/stderr (`%j` = job id, `%x` = job name). |
| `--dependency=afterok:JOBID` | Start only after another job succeeds. |

## Multi-node distributed training

For multi-node GPU jobs, launch one task per GPU and let your framework read
the rendezvous info SLURM provides:

```bash
#!/bin/bash
#SBATCH --job-name=ddp
#SBATCH --nodes=2
#SBATCH --gpus-per-node=8
#SBATCH --ntasks-per-node=8
#SBATCH --cpus-per-task=8
#SBATCH --time=04:00:00
#SBATCH --output=ddp-%x-%j.out

export MASTER_ADDR=$(scontrol show hostnames "$SLURM_JOB_NODELIST" | head -n1)
export MASTER_PORT=29500

srun python -m torch.distributed.run \
  --nnodes="$SLURM_NNODES" \
  --nproc_per_node=8 \
  --rdzv_backend=c10d \
  --rdzv_endpoint="$MASTER_ADDR:$MASTER_PORT" \
  train.py
```

On profiles with high-bandwidth interconnect (NVLink/InfiniBand), NCCL uses it
automatically. If you need to confirm the fabric, set `NCCL_DEBUG=INFO` and
check the log for the transport in use.

## Array jobs (sweeps)

Run the same script over many inputs with one submission:

```bash
#!/bin/bash
#SBATCH --job-name=sweep
#SBATCH --array=0-9%4          # 10 tasks, at most 4 running at once
#SBATCH --gpus=1
#SBATCH --time=00:30:00
#SBATCH --output=sweep-%A_%a.out

srun python train.py --config "configs/run_${SLURM_ARRAY_TASK_ID}.yaml"
```

`%A` is the array job id, `%a` the task index.

## Interactive sessions

For debugging, grab an interactive allocation instead of a batch script:

```bash
srun --gpus=1 --time=01:00:00 --pty bash     # interactive shell on a GPU node
# ... or a quick one-off:
srun --gpus=1 nvidia-smi
```

## GPU visibility

SLURM sets `CUDA_VISIBLE_DEVICES` to the GPUs your job was granted — don't
override it, or you may step on another job's GPUs. Check what you got with
`nvidia-smi` inside the allocation.

## Tips

- **Right-size the cluster.** Match your `--gpus`/`--nodes` requests to the
  cluster's node groups. A job asking for more GPUs than the cluster has will
  sit pending forever with reason `Resources` in `squeue`.
- **Use shared storage** for inputs and outputs so results survive a single
  node, and copy final artifacts off before deleting the cluster (see
  [Storage & Data](storage.md)).
- **Checkpoint often** and write checkpoints to shared storage so a preemption
  or node failure doesn't cost the whole run.
- **Watch it live.** The [Observability](observability.md) page shows SLURM
  node/job counts and GPU utilization while your job runs.
- **Why is my job pending?** `squeue --start` estimates a start time;
  `scontrol show job <id>` shows the reason (`Resources`, `Priority`,
  `QOSMaxGRES`, …).

---

**Next:** [Templates](templates.md) — turn a bare cluster into a ready
environment.
