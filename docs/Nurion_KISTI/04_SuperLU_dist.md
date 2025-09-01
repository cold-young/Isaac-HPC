# Sparse Matrix Format (HB, CSR)
**Date**: 2024.05.29 (Wed) <br>
**Writer**: Chanyoung Ahn ([cold-young](https://github.com/cold-young))
___

## What is `Harwell-Boeing (HB)` format (`.rua / cua`)? 
- **Reference:** [Link](https://people.sc.fsu.edu/~jburkardt/data/hb/hb.html)
- It is common to use the same compressed column storage to represent the matrix in memory.

    <!-- <img src="../img/HPC_02_HB_Format.png" height=200><br> -->

### **`16tdm_matrix.rua`**
```python
1:Tridiagonal matirx                                                      tdm     
2:            17             1             2            10             4
#    # of data lines  # of 행 누적 nnz #nnz 열 idx  # of A marix lines # of b matrix lines
3:RUA                       16            16            46             0
#                     # row           # column    # nnz 수
4:(26I3)          (26I3)          (5E15.8)            (5E15.8)            
5:F                          1             0
6:  1  3  6  9 12 15 18 21 24 27 30 33 36 39 42 45 47
7:  1  2  1  2  3  2  3  4  3  4  5  4  5  6  5  6  7  6  7  8  7  8  9  8  9 10
8:  9 10 11 10 11 12 11 12 13 12 13 14 13 14 15 14 15 16 15 16
9: 2.00000000e+00-1.00000000e+00-1.00000000e+00 2.00000000e+00-1.00000000e+00
10:-1.00000000e+00 2.00000000e+00-1.00000000e+00-1.00000000e+00 2.00000000e+00
...
17:-1.00000000e+00 2.00000000e+00-1.00000000e+00-1.00000000e+00 2.00000000e+00
18: 2.00000000e+00
19: 2.00000000e+00 6.00000000e+00-9.00000000e+00 4.00000000e+00-9.00000000e+00
...
22: 9.00000000e+00
```

- **Line 1**: Title / KEY
- **Line 2**: 
  - Total number of data lines: `17` (5 - 22 rows: 17 lines)
  - Number of data lines for pointers: `1` (6 row: 1 line) 
  - Number of data lines for row or variable indices: `2` (7-8 row: 2 lines)
  - Number of data lines for numerical values of matrix entries: `10` (9-18 row: 10 lines)
  - **PHSCRD** Number of data lines for right hand side vectors, starting guesses, and solutions: `4` (19-22 row: 4 lines)
- **Line 3**: 
  - MATRIX TYPE: `RUA`(real, unsymmetric + square, assembled)
  - Number of rows or variable: `16` 
  - Number of columns or variable: `16`
  - Number of nonzero entires: `46` (the number of elements in 7-8 row)
  - Number of elemental matrix entries, "assembled":0 `0`
- **Line 4**:
  - (26I3): line break when write 26 element. Each element has 3 space. (6 row) / the accumulate number of row nnz
  - (26I3): line break when write 26 element. Each element has 3 space. (7-8 row) / nnz colmn idx
  - (5E15.8): line break when write 5 element. Each element has 15 space. (9-18 row) / A matrix
  - (5E15.8): line break when write 5 element. Each element has 15 space. (19-22 row) / b matrix 

- **Line 5** (PHSCRD>0):
  - describes the right hand side information: `F`  right side information
  - the number of right hand side: `1`
  - number of row indices: `0`

- 6 row: point data / 행렬 항목의 시작 위치
- 7-8 row: index data / 각 항목에 해당되는 행의 인덱스
- 9-18 row: A matrix
- 19-22 row: b matrix

> All line length should be limited to 80 characters

## Examples in SuperLU library

```shell
cds # cd scratch/$USERID
cd ~/examples

# Example: pddrive2.out & test.out
mpiicc -I$INC_SUPERLUD -L$LIB_SUPERLUD -lsuperlu_dist dreadhb.c dcreate_matrix.c dcreate_matrix_perturbed.c 
pddrive2.c -o pddrive2.out

# Example: test.out (monitor b matrix)
mpiicc -I$INC_SUPERLUD -L$LIB_SUPERLUD -lsuperlu_dist dreadhb.c dcreate_matrix.c dcreate_matrix_perturbed.c test.c -o test.out

# Test exampels
mpirun -np 4 ./OUTFILENAME.out -r 2 -c 2 tdm 16_new.rua 
```
### `dreadhb.c` # 
- Read double precision matrix stored in Harwell-Boeing format. 

### `dcreate_matrix.c` 
- Functions for reading the matrix with various formats
- `DCREATE_MATRIX_POSTFIX` read the matrix from data file in different formats (.rua, .rb, .mtx, .dat, .ddatnh, and .bin)

### `decreate_matrix_perturbed.c`
 * `DCREATE_MATRIX_PERTURBED` 
   * read the matrix from data file in Harwell-Boeing format, and distribute it to processors in a distributed compressed row format. It also generate **the distributed true solution X** and **the right-hand side RHS**.
   * `iam`: `iam` means current processor's rank (ID)

* `DCREATE_MATRIX` read the matrix from data file in Harwell-Boeing format, and distribute it to processors in a distributed compressed row format. It also generate the distributed true solution X and the right-hand.
  
### `pddrive2.c` and `test.c`
* These examples illustrate how to use PDGSSVX to solve systems repeatedly w/ the same sparsity pattern of matrix A.

## `test.c`: SuperLU-dist Benchmark 
* [Code Documentation](https://portal.nersc.gov/project/sparse/superlu/superlu_dist_code_html/index.html)
* Add Debug part in `pddrive2.c`
```cpp
    for(ii=0;ii<4;ii++) {
        printf("iam = %3d, x[%2d] = %15.10f\n", iam, ii, b[ii]);
        printf("iam = %d, A.nnz_loc = %d\n", iam, Astore_temp->nnz_loc);
        printf("iam = %d, A.fst_row = %d\n", iam, Astore_temp->fst_row);
        printf("iam = %d, A.m_loc = %d\n", iam, Astore_temp->m_loc);
    }
```


```cpp
// pdgssvx Function
pdgssvx(&options, &A, &ScalePermstruct, b, ldb, nrhs, &grid,
            &LUstruct, &SOLVEstruct, berr, &stat, &info);

```
- [**Reference**](https://portal.nersc.gov/project/sparse/superlu/superlu_dist_code_html/pdgssvx_8c.html)
- `pdgssvx` solves a system of linear equations $A \times X = B$, by using Gaussian elimination with "static pivoting" to compute the LU factorization of A.
  
  <img src="../img/HPC_util_03.png" height=200>

## CSR(Compressed Sparse Row) Format
- **Reference:** [Link](https://gaussian37.github.io/math-la-sparse_matrix/)
<br>

- `Sparse Matrix`: A matrix has less non-zero element. (<>`dense matrix`) 
- **CSR Format**:
  - `Data`: an array containing non-zero elements
  - `Rol`: an array containing column index of non-zero elements
  - `Row`: an array containing the number of non-zero elements before *the nth* row (accumulated value)
