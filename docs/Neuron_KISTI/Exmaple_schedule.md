## Job submission Example

```bash
cd /scratch/x3320a02/KISTAR_data-collection/regression
mkdir -p logs slurm

cat > slurm/train_haptic4_test.sbatch <<'EOF'
#!/bin/bash
#SBATCH -J haptic4_test
#SBATCH -p cas_v100_4
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=8
#SBATCH --gres=gpu:1
#SBATCH --time=00:30:00
#SBATCH --comment=pytorch
#SBATCH -o /scratch/x3320a02/KISTAR_data-collection/regression/logs/%x_%j.out
#SBATCH -e /scratch/x3320a02/KISTAR_data-collection/regression/logs/%x_%j.err

set -euo pipefail

cd /scratch/x3320a02/KISTAR_data-collection/regression

module purge
module load python/3.7.1

source activate reg

echo "[INFO] Host: $(hostname)"
echo "[INFO] PWD : $(pwd)"
echo "[INFO] Python: $(which python)"
python -u train_haptic4.py
EOF

```