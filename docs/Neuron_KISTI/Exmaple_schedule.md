## Job submission Example

### Sbatch Exmaple with DDP (Torchrun) - Experimental
```bash
#!/bin/bash
#SBATCH -J JOB_NAME
#SBATCH -p amd_h200nv_8 
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1         
#SBATCH --cpus-per-task=16
#SBATCH --gres=gpu:2
#SBATCH --time=00:40:00
#SBATCH --comment=pytorch
#SBATCH -o /scratch/YOUR_ID/logs/%x_%j.out
#SBATCH -e /scratch/YOUR_ID/logs/%x_%j.err
#SBATCH --mail-type=END
#SBATCH --mail-user=YOUR_EMAIL

set -euo pipefail
ulimit -n 65535 || true

export OMP_NUM_THREADS=1
export MKL_NUM_THREADS=1
export OPENBLAS_NUM_THREADS=1
export NUMEXPR_NUM_THREADS=1

PY=/home01/[$USER]/.conda/envs/reg/bin/python
WORKDIR=/scratch/[$USER]/[$YOUR_PROJECT]
TRAIN_SCRIPT=[$YOUR_TRAIN_FILE].py

echo "[INFO] Host: $(hostname)"
echo "[INFO] Job : ${SLURM_JOB_ID:-NA}"
echo "[INFO] Using PY: $PY"
$PY -V

nvidia-smi
echo "[INFO] CUDA_VISIBLE_DEVICES=${CUDA_VISIBLE_DEVICES:-<unset>}"

# ---- decide nproc_per_node robustly ----
if [[ -n "${CUDA_VISIBLE_DEVICES:-}" ]]; then
  NPROC_PER_NODE=$(echo "$CUDA_VISIBLE_DEVICES" | awk -F',' '{print NF}')
else
  # 2) Slurm GPU 
  NPROC_PER_NODE="${SLURM_GPUS_ON_NODE:-${SLURM_GPUS_PER_NODE:-2}}"
fi
export NPROC_PER_NODE
echo "[INFO] NPROC_PER_NODE=$NPROC_PER_NODE"

# ---- quick torch sanity ----
$PY - <<'PY'
import os, torch
print("[INFO] torch", torch.__version__)
print("[INFO] cuda available:", torch.cuda.is_available())
print("[INFO] device_count:", torch.cuda.device_count())
print("[INFO] CUDA_VISIBLE_DEVICES:", os.environ.get("CUDA_VISIBLE_DEVICES"))
if torch.cuda.is_available() and torch.cuda.device_count() > 0:
    for i in range(torch.cuda.device_count()):
        print(f"[INFO] cuda:{i} ->", torch.cuda.get_device_name(i))
PY
echo "[INFO] which python: $(which python || true)"
echo "[INFO] which torchrun: $(which torchrun || true)"
"$PY" -c "import torch, sys; print('torch', torch.__version__); print('python', sys.executable)"
"$PY" -c "import torch.distributed.run as r; print('torch.distributed.run OK')"

# ---- stage DATA into node-local ----
BASE_TMP="${SLURM_TMPDIR:-/tmp}"
DATA_SRC=/scratch/[$USER]/[$YOUR_DATASET]
DATA_DST="$BASE_TMP/[$YOUR_TMP_DATASET]"

echo "[INFO] ---- STAGE DATA START ----"
date
[[ -d "$DATA_SRC" ]] || { echo "[ERR] DATA_SRC missing: $DATA_SRC"; exit 1; }
[[ -n "$DATA_DST" ]] || { echo "[ERR] DATA_DST empty"; exit 1; }
[[ "$DATA_DST" == *"/YOUR_DATASET" ]] || { echo "[ERR] DATA_DST looks unsafe: $DATA_DST"; exit 1; }

mkdir -p "$DATA_DST"
time rsync -a --delete "$DATA_SRC"/ "$DATA_DST"/
date
echo "[INFO] ---- STAGE DATA END ----"

export DATA_DIR="$DATA_DST"
echo "[INFO] DATA_DIR=$DATA_DIR"
ls -l "$DATA_DST" | head -n 5 || true

cd "$WORKDIR"
echo "[INFO] PWD=$(pwd)"
echo "[INFO] ---- TRAIN START ----"
date
export PYTHONUNBUFFERED=1

time $PY -m torch.distributed.run \
  --standalone --nnodes=1 --nproc_per_node="$NPROC_PER_NODE" [$YOUR_FILE].py

date
echo "[INFO] ---- TRAIN END ----"
```


### PytorchDDP (Official Guide)
1. Single Node with 2GPU
```bash
#!/bin/bash -l
#SBATCH -J PytorchDDP
#SBATCH -p cas_v100_4
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --gres=gpu:2
#SBATCH --comment pytorch
#SBATCH --time 1:00:00
#SBATCH -o %x.o%j
#SBATCH -e %x.e%j

# Configuration
traindata='{path}'
master_port="$((RANDOM%55535+10000))"

# Load software
conda activate pytorchDDP

# Launch one SLURM task, and use torch distributed launch utility
# to spawn training worker processes; one per GPU
srun -N 1 -n 1 python main.py -a config \
                                --dist-url "tcp://127.0.0.1:${master_port}" \
                                --dist-backend 'nccl' \
                                --multiprocessing-distributed \
                                --world-size $SLURM_TASKS_PER_NODE \
                                --rank 0 \
                                $traindata
```

2. Multi Node with 2GPU
```bash
#!/bin/bash -l
#SBATCH -J PytorchDDP
#SBATCH -p cas_v100_4
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=1
#SBATCH --gres=gpu:1
#SBATCH --comment pytorch
#SBATCH --time 10:00:0
#SBATCH -o %x.o%j
#SBATCH -e %x.e%j

# Load software list
module load {module name}
conda activate {conda name}

# Setup node list
nodes=$(scontrol show hostnames $SLURM_JOB_NODELIST) # Getting the node names
nodes_array=( $nodes )
master_node=${nodes_array[0]}
master_addr=$(srun --nodes=1 --ntasks=1 -w $master_node hostname --ip-address)
master_port=$((RANDOM%55535+10000))
worker_num=$(($SLURM_JOB_NUM_NODES))

# Loop over nodes and submit training tasks
for ((  node_rank=0; node_rank<$worker_num; node_rank++ )); do
          node=${nodes_array[$node_rank]}
          echo "Submitting node # $node_rank, $node"
          # Launch one SLURM task per node, and use torch distributed launch utility
          # to spawn training worker processes; one per GPU
          srun -N 1 -n 1 -w $node python main.py -a $config \
                                                             --dist-url tcp://$master_addr:$master_port \
                                                             --dist-backend 'nccl' \
                                                            --multiprocessing-distributed \
                                                            --world-size $SLURM_JOB_NUM_NODES \
                                                            --rank $node_rank &

          pids[${node_rank}]=$!
done

# Wait for completion
for pid in ${pids[*]}; do
          wait $pid
done
```