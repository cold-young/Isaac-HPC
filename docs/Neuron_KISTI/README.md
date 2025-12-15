# Neuron - KISTI Documentation

* contribution: [Chanyoung Ahn](https://cold-young.github.io/)

This document is for using NVIDIA Isaac Sim on HPC systems (especially, KISTI's Neuron system).

* Overall Guideline: [Link](https://docs-ksc.gitbook.io/neuron-user-guide-eng/system/user-environment)

### Login
* Use myksc 
* Or ssh
```shell
 ssh -l <user ID> neuron01.ksc.re.kr -p 22
```

### General Policy

* Home directory `/home01`: 64GB, # of files < 100K 
* Scratch directory `/scratch`: 100TB, # of files < 4M. Files not accessed in 15 days will be automatically deleted! 

* Check your directory 
  ```shell
  neuroninfo
  ```

* Short command for scratch directory
  ```shell
    cds # scratch/$USER_ID

  ```

### Just Test in Debug Node! 
```shell
ssh glogin01
ssh gdebug01

nvidia-smi
apptainer exec --nv myenv.sif python -c "import torch; print(torch.cuda.is_available())"

./(실행파일) (옵션)
```

### Interactive job
* Cannot use in debug node
```shell
salloc --partition=cas_v100_4 --nodes=2 --ntasks-per-node=2 --gres=gpu:2 --comment python
salloc --partition=ivy_v100_2 -N 2 -n 4 --tasks-per-node=2 --gres=gpu:2 --comment={SBATCH option name} 
# Refer to the Table of SBATCH option name per application 

srun ./(실행파일) (실행옵션)

exit 

# Delete the job
scancel [JOBID]

# Job monitoring
sinfo
squeue -u <YOUR ID>

```

## Build own's Environment
### Conda
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
