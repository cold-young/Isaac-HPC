# Build own's Environment

## Conda
```shell
$ module load gcc/10.2.0 cuda/11.4 cudampi/openmpi-4.1.1 python/3.7.1 cmake/3.16.9
$ conda create -n my_pytorch  
$ source activate my_pytorch
(my_pytorch) $  


# If you want to install torch / 
# for Python3.11
pip install torch==2.7.0 torchvision==0.22.0 --index-url https://download.pytorch.org/whl/cu128

# for Python3.10
pip install torch==2.5.1 torchvision==0.20.1 --index-url https://download.pytorch.org/whl/cu118

# IsaacSim # Recommmand: Just use image!
pip install "isaacsim[all,extscache]==5.1.0" --extra-index-url https://pypi.nvidia.com

## Install check
python - <<'PY'
import torch
print(torch.__version__)
print(torch.cuda.is_available())
if torch.cuda.is_available():
    print(torch.cuda.get_device_name(0))
PY

```

## Docker / Apptainer
### Build Your IsaacSim Environment with Apptainer 

Load singularity module in Neuron system
```shell
$ module load singularity/3.11.0
# or
$ $HOME/.bash_profile
export PATH=$PATH:/apps/applications/singularity/3.11.0/bin/
# check
singluarity --version
```

### (Local) Install Isaac Sim container 
* **Caution:** Need NVIDIA Docker in your local machine!
 -- If you use docker in KIST, please use Docker Pro for commercial license. 
```shell
docker pull nvcr.io/nvidia/isaac-sim:5.1.0
```

### Change Docker image in Isaac Lab or dex_soldering (internal repo for KIST)

1. Install `IsaacLab` or `dex_soldering` repo. 
   ```shell
   cd $YOUR_PROJECT_DIR
   ```

* `/docker/cluster/.env.cluster`
* Change [$YOUR_PROJECT] to `isaaclab` or `dex_soldering`
```shell
# Job scheduler used by cluster.
# Currently supports PBS and SLURM
CLUSTER_JOB_SCHEDULER=SLURM

# e.g. /cluster/scratch/$USER/docker-isaac-sim
CLUSTER_ISAAC_SIM_CACHE_DIR=/scratch/$USER/isaac_cache/isaacsim-5.1.0/docker-isaac-sim

# Isaac Lab directory on the cluster (has to end on isaaclab)
CLUSTER_ISAACLAB_DIR=/scratch/$USER/[$YOUR_PROJECT]/isaaclab

# Cluster login
CLUSTER_LOGIN=[$YOUR_KISTI_ID]@neuron.ksc.re.kr

# Cluster scratch directory to store the SIF file
# CLUSTER_SIF_PATH=/some/path/on/cluster/
CLUSTER_SIF_PATH=/scratch/$USER/dex_soldering/containers
# Remove the temporary isaaclab code copy after the job is done
REMOVE_CODE_COPY_AFTER_JOB=false
# Python executable within Isaac Lab directory to run with the submitted job
CLUSTER_PYTHON_EXECUTABLE=scripts/reinforcement_learning/rsl_rl/train.py

TMPDIR=${SLURM_TMPDIR:-/tmp}/isaaclab_${SLURM_JOB_ID}
```

* `/docker/cluster/cluster_interface.sh`
```shell
...
submit_job() {

    echo "[INFO] Arguments passed to job script ${@}"

    case $CLUSTER_JOB_SCHEDULER in
        "SLURM")
            CMD='bash -l' ## Add
            job_script_file=submit_job_slurm.sh
            ;;
        "PBS")
            CMD=bash ## Add 
            job_script_file=submit_job_pbs.sh
            ;;
        *)
            echo "[ERROR] Unsupported job scheduler specified: '$CLUSTER_JOB_SCHEDULER'. Supported options are: ['SLURM', 'PBS']"
            exit 1
            ;;
    esac

    ssh $CLUSTER_LOGIN "cd $CLUSTER_ISAACLAB_DIR && $CMD $CLUSTER_ISAACLAB_DIR/docker/cluster/$job_script_file \"$CLUSTER_ISAACLAB_DIR\" \"isaac-lab-$profile\" ${@}"
}
...
```

### Build and push a docker image
1. First, build a new image
    
    ```bash
    python docker/container.py start
    ```
    
2. push the created image to the server.
    
    ```bash
    ./docker/cluster/cluster_interface.sh push base
    ```

3. check your docker image (in local)
   ```shell
   docker images | grep -i isaac
   ```

If you want to create a custom image with a different name, replace the name `base` with other names. Also rename the files (i.e. .`env.base`) accordingly.

### Docker image to Local Apptainer SIF conversion
```shell
docker save isaac-lab-base:latest -o isaac-lab-base_latest.tar
ls -lh isaac-lab-base_latest.tar
```

```shell
# Apptainer build 
sudo apptainer build isaac-lab-base_isaacsim5.1.0.sif docker-archive://isaac-lab-base_latest.tar

# Singulurity build
sudo singularity build isaac-lab-base_isaacsim5.1.0.sif docker-archive://isaac-lab-base_latest.tar
```
#### Check the SIF file
```shell
ls -lh isaac-lab-base_isaacsim5.1.0.sif
apptainer inspect isaac-lab-base_isaacsim5.1.0.sif | head
```

### Test 10 epoch in HPC Neuron system
- We cannot use `video` option in HPC system...... 
```shell
singularity exec --nv --containall \
  -B "$ISAACLAB_SRC:/workspace/isaaclab:rw" \
  -B "$KIT_DATA:/isaac-sim/kit/data:rw" \
  -B "$KIT_CACHE:/isaac-sim/kit/cache:rw" \
  -B "$KIT_LOGS:/isaac-sim/kit/logs:rw" \
  "$SIF" \
  bash -lc '
    set -e
    cd /workspace/isaaclab

    touch /isaac-sim/kit/data/__rw_test__ && rm -f /isaac-sim/kit/data/__rw_test__

    /isaac-sim/python.sh scripts/reinforcement_learning/rsl_rl/train.py \
      --task Isaac-Velocity-Rough-Anymal-C-v0 \
      --headless \
      --num_envs 64 \
      --max_iterations 10 \ 
  '
```