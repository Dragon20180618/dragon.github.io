date: 2022-08-25 09:17:16

tags: openblas

# OpenBlas-Make

## environment: 
* redhat
* gcc version 4.8.5 20150623 (Red Hat 4.8.5-36) (GCC)
## build
```shell
make CC=gcc FC=gfortran make_NB_JOBS=1
make install PREFIX=<install_path>
```
if you don't make with suffix `make_NB_JOBS`, and has a old lapack in you machine,

openblas will generate before included lapack built, And won't call the old lapack lib.
## test
```c
#include <cblas.h>
#include <stdio.h>

int main()
{
  int i=0;
  double A[6] = {1.0,2.0,1.0,-3.0,4.0,-1.0};         
  double B[6] = {1.0,2.0,1.0,-3.0,4.0,-1.0};  
  double C[9] = {.5,.5,.5,.5,.5,.5,.5,.5,.5}; 
  cblas_dgemm(CblasColMajor, CblasNoTrans, CblasTrans,3,3,2,1,A, 3, B, 3,2,C,3);

  for(i=0; i<9; i++)
    printf("%lf ", C[i]);
  printf("\n");
  return 0;
}
```
```shell
$ gcc -I ~/OpenBLAS/include  -L ~/OpenBLAS/lib -lopenblas main.cpp -o solver
$ ./solver
11.000000 -9.000000 5.000000 -9.000000 21.000000 -1.000000 5.000000 -1.000000 3.000000
```