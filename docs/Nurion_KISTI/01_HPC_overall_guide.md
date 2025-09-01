# NURION GUIDE (KOR.)
**Date**: 2024.05.21 (Tue) <br>
**Writer**: Chanyoung Ahn ([cold-young](https://github.com/cold-young))

- KOR version 작성 후, ENG version 작성 예정.
- Reference: [Nurion-guide](https://docs-ksc.gitbook.io/nurion-user-guide-eng)

## NURION 
- KISTI 슈퍼컴퓨터 5호기, 리눅스 기반 초병렬 클러스터 시스템

### 계산노드 
1. **KNL (manycore) node**
     1. **Processor**
       - Intel Xeon Phi 7250 processor / 8,305 nodes  <br>
       - Perfonrmance/CPU: 3.0464 TFLOPS <br>
       - Core/CPU: 68 <br>
       - Node/CPU: 1 <br>
      
     1. **Main Memory** <br>
       - Memory/Node: 96 GB 
  
1. **SKL (CPU-only) node**
  - Intel Xeon Gold 6148 (Skylake) / 132 nodes

### Login node
#### SSH
```shell
ssh -l <UserID> nurion.ksc.re.kr
ssh -l <UserID> nurion.ksc.re.kr -p 22
```

#### MyKSC
- https://my.ksc.re.kr
- Add `ttyd(terminal)` app

### Send & Receive Files
- FTP Client 
```shell
ftp nurion-dm.ksc.re.kr
# or
sftp [USERID@]nurion-dm.ksc.re.kr [-p22]
```

### Monitor SRU Time
```shell
isam
```

### File system & quater policy
- Provide two file systems, `/home01/$USER` and `/scratch/$USER`(`$ cds`).
- Performance of Home directory is limited. We should use scratch directory when use any computation..
- `/home01/$USER`: 64GB 
- `/scratch/$USER`: 100TB / Delete files have not been used for 15 days

```shell
lfs quota -h /home01 # check file system disk
```

## How to set module & compiler
1. **Compiler & modulus Setting**
   - ```module avail```: view all module list
   - ```module load [module name] [module name]``` .. 
   - ```module list```: view install module list
   - ```module purge```: remove all used module `

2. **Serial Program Compile**
- Intel Compiler
   - `icc` or `icpc`: C / C++ Language
   - `ifort`: F77 / F90 Language
   - **Compiler Option**
     - `-O`[1|2|3]: Object optimization
     - `-ip`, `-ipo`: 프로시저 간 최적화
     - `-qopt_report=[0|1|2|3|4]`:벡터 진단 정보의 양을 조절
     - `xCORE-AVX512`, `xMIC-AVX512`: support CPU with 512bits register (for SKL node) / support MIC with 512bits register (for KNL node)
     - `fast`: `-O3 -ipo -no-prec-div -static`, `-fp-model fast=2` macro
     - `-shared/-shared-inel/-i_dynamic`: link shared libraries
     - `-g -fp`: provide debugging information
     - `-qopenmp`: Use multi-thread code based on OpenMP
     - `-openmp_report=[0|1|2]`: Change OpenMP parallelization level
     - `-fPIC, fpic`: Compile with PIC (Position Independent Code)

    - **Recommend Options**
      - **SKL**: `-O3 -fPIC -xCORE-AVX512`
      - **KNL**: `-O3 -fPIC -xMIC-AVX512`
      - **SKL & KNL**: `-O3 -fPIC -xCOMMON-AVX512`  
  
3. **Parallel Program Compile**
- **OpenMP Compiler**
  - `icc`, `icpc`, `ifort`: C / C++ / F77/F90 `-qopenmp`
  - `gcc`, `g++`, `gfortran`: C / C++ / F77/F90 `-fopenmp`

  **Example**:
    ```shell
    modle load craype-mic-knl intel/18.0.3
    icc -o test_omp.ext -qopenmp -O3 -fPIC -xMIC-AVX512 test_omp.c
    ```

- **MPI Compiler**
  
  <img src="../img/HPC_01.png" weight=70%>

  **Example**:
    ```shell
    modle load craype-mic-knl intel/18.0.3 impl/18.0.3
    mpiicc -o test_omp.ext -O3 -fPIC -xMIC-AVX512 test_omp.c
    mpirun -np 2 ./text_mpi.exe
    ```

## Portable Batch System (PBS) Scheduler
- One node for one user
- `/scratch/$USER`  

### Monitoring
#### 1. Submit Batch Job 
- script examples `/apps/shell/home/job_examples`
- PBS scheduler options
  - `PBS -V`: remain current environment variable
  - `PBS -N`: name current job
  - `PBS -q`: queue for job 

  **MPI (IntelMPI) Example (mpi.sh)**:
    ```shell
    #!/bin/sh
    #PBS -N IntelMPI_job
    #PBS -V
    #PBS -q normal
    #PBS -A
    #PBS -l select=4:ncpus=64:mpiprocs=64
    #PBS -l walltime=04:00:00

    cd $PBS_O_WORKDIR

    module purge
    module load craype-mic-knl intel/18.0.3 impi/18.0.3
    mpirun ./test_mpi.exe
    ``` 
    - 4 nodes, 64 process per a node (MPI process of total: 256)
  
#### 2. Submit Interactive Job
- Not use #PBS but use `-I -A` option.
    ```shell
    qsub -I -X -l select=1:ncpus=68:ompthread=1 -l walltime=12:00:00 -q normal -A {PBS option name}
    ``` 
#### 3. Job Mornitoring
  ```shell
  showq # view queue

  pbs_queue_check # view available queue list 
  ```

  ```shell
  cp ./EXAMPLE.sh scratch/$USER_ID
  cds # scratch/$USER_ID
  
  qsub EXAMPLE.sh
  qstat -u [USER_ID] # see my queue(only R state)
  ```
