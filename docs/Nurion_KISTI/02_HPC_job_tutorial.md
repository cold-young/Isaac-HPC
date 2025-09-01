# HPC Job Tutorials (KOR.)
**Date**: 2024.05.30 (Thur) <br>
**Writer**: Chanyoung Ahn ([cold-young](https://github.com/cold-young))

- Reference: [Nurion-guide](https://docs-ksc.gitbook.io/nurion-user-guide-eng)

## Submit Job 
- Reference: [Link](https://docs-ksc.gitbook.io/nurion-user-guide/undefined/running-jobs-through-scheduler-pbs)

```shell
qsub -I -X -l select=1:ncpus=64:ompthreads=1 -l walltime=02:00:00 -q {QUEUE_NAME} -A {PBS OPTIONS}
```
- `select`: The number of using nodes
- `ncpus`: The number of using cores
- `ompthreads`: The number of using threads 
- `walltime`: runtime

### Common shell commands
```shell
motd # See basic notice and help
pbs_status # watch all nodes status 

qstat -u #view my jobs
qstat -i -w -T -u [USER_ID]
qstat -xf {JOBNUMBER}.pbs # view log 
```
- `qstat -u support`error! why cannot see my jobs?

### Submit NURION Interactive job

### Write job script 
```shell
cd 
cd job_examples

mpiifort hello.f90 -o hello.x
vim mpi.sh
```
- Compiler Option
```shell
-O3 -fPIC -xMIC-AVX512 #KNL Node Recommand
```

- *mpi.sh*
```shell
#!/bin/sh
#PBS -V
#PBS -N mpi_job
#PBS -q debug
#PBS -A etc
#PBS -l select=2:ncpus=32:mpiprocs=64
#PBS -l walltime=04:00:00

cd $PBS_O_WORKDIR

module purge
module load craype-mic-knl intel/18.0.3 impi/18.0.3

mpirun ./hello.x
```


## Send/Download files 

```shell
sftp [USER_ID]@nurion-dm.ksc.re.kr # Nurion
sftp [USER_ID]@neuron-dm.ksc.re.kr # neuron

# OTP PW: application
# PW: USER_ID PW
```

```shell
# In ftp directory
cd; ls; mkdir 

# In local directory
lcd; lls; lmkdir 

# Send local > nurion
put [LOCAL_FILE]
# mput [] [] ..

# Download from nurion
get [NURION_FILE]
#mget [] [] .. 
```