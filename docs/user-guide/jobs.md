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

## Tips

- **Right-size the cluster.** Match your `--gpus`/`--nodes` requests to the
  cluster's node groups. A job asking for more GPUs than the cluster has will
  sit pending.
- **Use shared storage** for inputs and outputs so results survive a single
  node, and copy final artifacts off before deleting the cluster.
- **Watch it live.** The [Observability](observability.md) page shows SLURM
  node/job counts and GPU utilization while your job runs.

---

**Next:** [Templates](templates.md) — turn a bare cluster into a ready
environment.
