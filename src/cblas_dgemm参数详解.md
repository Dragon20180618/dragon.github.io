date: 2022-08-22 14:34:18

tags:

# cblas_dgemm参数详解

[dragon](mailto: wauqas@gmail.com) 22-8-15

## 代码

```c
#include <cblas.h>
#include <stdio.h>

int main()
{
  int i = 0;
  double A[6] = {1.0, 10.0, 20.0, 30.0, 40.0, 50.0};
  double B[6] = {1.0, 10.0, 20.0, 30.0, 40.0, 60.0};
  double C[9] = {.5, .5, .5, .5, .5, .5, .5, .5, .5};
  cblas_dgemm(CblasColMajor, CblasNoTrans, CblasTrans, \
              3, 3, 2, 1, A, 3, B, 3, 0, C, 3);
  for (int j = 0; j < 3; j++)
  {
    for (i = 0; i < 3; i++)
      printf("%lf ", C[i*3+j]);
    printf("\n");
  }
  return 0;
}
```

## 解释

```c
void cblas_dgemm(
    	OPENBLAS_CONST enum CBLAS_ORDER Order, //行列主序
        OPENBLAS_CONST enum CBLAS_TRANSPOSE TransA, //矩阵A转置
        OPENBLAS_CONST enum CBLAS_TRANSPOSE TransB, //矩阵B转置
        OPENBLAS_CONST blasint M, //op(A)的行数
        OPENBLAS_CONST blasint N, //op(B)的列数
        OPENBLAS_CONST blasint K, //op(A)的列数和op(B)的行数
        OPENBLAS_CONST double alpha, //A的缩放
        OPENBLAS_CONST double *A, //matrix A
        OPENBLAS_CONST blasint lda, //A的第一维度,跟主序有关
       	OPENBLAS_CONST double *B, //matrix B
        OPENBLAS_CONST blasint ldb, //B的第一维度
        OPENBLAS_CONST double beta, //C 的缩放
        double *C, //matrix C
        OPENBLAS_CONST blasint ldc //C的第一维度
);
```

最终结果为$C=alpha*op(A)*op(B)+beta*C$

