# Neuron - KISTI Documentation

* contribution: [Chanyoung Ahn](https://cold-young.github.io/)

This document outlines how to use NVIDIA Isaac Sim on HPC systems, particularly KISTI's Neuron system.

* Overall Guideline: [Link](https://docs-ksc.gitbook.io/neuron-user-guide-eng/system/user-environment)
* Neuron system main page: [Link](https://docs-ksc.gitbook.io/neuron-user-guide/undefined/running-jobs-through-scheduler-slurm)

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
salloc -p amd_a100nv_8 -N 1 --gres=gpu:1 -c 8 --mem=32G -t 02:00:00 --comment pytorch # for learning isaacsim (slow version)
salloc -p eme_h200nv_8 -N 1 --gres=gpu:2 -c 16 --mem=32G -t 00:30:00 --comment pytorch
# Refer to the Table of SBATCH option name per application 

srun ./(실행파일) (실행옵션)

exit 

# Delete the job
scancel [JOBID]

# Job monitoring
sinfo
squeue -u <YOUR ID>

```

# See other tutorial documents! 

- [build_your_env](./build_your_env.md)
- [Run model with multiple GPU](./Exmaple_schedule.md)