# Environment Setting for PaScaL_TDMA(KOR.)
**Date**: 2024.05.21 (Tue) <br>
**Writer**: Chanyoung Ahn ([cold-young](https://github.com/cold-young))

- KOR version 작성 후, ENG version 작성 예정.
- Reference: [Nurion-guide](https://docs-ksc.gitbook.io/nurion-user-guide-eng)
___

## Prerequisites

1. My KSC - `Nurion` or `Neuron` / terminal
2. See `module av`, `load` compiler/libraries

```shell
craype-network-opa
craype-mic-knl
intel/oneapi_21.2
impi/oneapi_21.2
xccels_lib/MUMPS_5.6.2
xccels_lib/STARUMPACK_7.1.4
xccels_lib/superlu_dist_8.1.2
```

```shell
# When start
module load craype-network-opa craype-mic-knl \
intel/oneapi_21.2 impi/oneapi_21.2 \
xccels_lib/MUMPS_5.6.2 xccels_lib/STRUMPACK_7.1.4 \
xccels_lib/superlu_dist_8.1.2 

# .. Or Add these commands in .bashrc
# vim .bashrc
module add craype-mic-knl
module add intel/oneapi_21.2
module add impi/oneapi_21.2

module add xccels_lib/MUMPS_5.6.2
module add xccels_lib/STRUMPACK_7.1.4
module add xccels_lib/superlu_dist_8.1.2

alias interactive='qsub -I -l select=1:ncpus=68:mpiprocs=68:ompthreads=1 -l walltime=4:00:00 -q debug -A etc'
###
source .bash_rc


# Check 
module list
```

* We should submit any job and `interactive` alias in `scratch/USERID`.

## SuperLU Example
```shell
cds # cd scratch/$USERID
cd ~/examples

# Example: pddrive2.out
mpiicc -I$INC_SUPERLUD -L$LIB_SUPERLUD -lsuperlu_dist dreadhb.c dcreate_matrix.c dcreate_matrix_perturbed.c pddrive2.c -o pddrive2.out

# Example: test.out
mpiicc -I$INC_SUPERLUD -L$LIB_SUPERLUD -lsuperlu_dist dreadhb.c dcreate_matrix.c dcreate_matrix_perturbed.c test.c -o test.out

# Example: pddrive2.out with MKL_parallel link 
mpiicc -I$INC_SUPERLUD -L$LIB_SUPERLUD -lsuperlu_dist -L/apps/compiler/intel/oneapi_21.2/mkl/2021.2.0/lib/intel64/ -mkl=parallel dreadhb.c dcreate_matrix.c dcreate_matrix_perturbed.c pddrive2.c -o pddrive2.out
```

```shell
# Test exampels
mpirun -np 4 ./OUTFILENAME.out -r 2 -c 2 tdm 16_new.rua 
```
- `-np`: the number of processors  
- `-r`: the number of processor grid row
- `-c`: the number of processor grid column 


## PaScaL_TDMA Example
- Reference: [Docs](https://xccels.github.io/PaScaL_TDMA/)
```shell
cd examples
mpirun -np 2 ./ex1_single.out

cd ~/run
mpirun -np 8 ./convection_3D.out ./PARA_INPUT.inp
```


### [Makefile Tutorial](../C_lang/Makefile.md)