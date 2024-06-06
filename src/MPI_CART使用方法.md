date: 2023-03-13 15:50:12

tags: 

## MPI_Cart_Create

将一维的通信空间转为多维

接口

```fortran
subroutine MPI_Cart_shift(comm, direction, disp, rank_source, rank_dest)
        integer,intent(in)  ::comm, 		!通信域
        integer,intent(in)  ::direction,    !维度序号
        integer,intent(in)  ::disp,		   	!偏移量
  		integer,intent(out) ::rank_source,	!向本进程发送数据的进程 如果没有则为-1
  	    integer,intent(out) ::rank_dest		!本进程发送数据的目的进程 如果没有则为-1
end subroutine MPI_Cart_shift
```

## 测试

* 查看period对网络生成的影响

```fortran
program main
    use mpi
    integer ierr
    integer rank,size
    integer np_dim(2)
    logical period(2)
    integer mpi_world_cart
    integer src,dest
    np_dim=3
    period(1)=.false.
    period(2)=.true.
    call MPI_Init(ierr)
    call MPI_Cart_create( MPI_COMM_WORLD    &
                            , 2                 &
                            , np_dim            &
                            , period            &
                            , .false.           &
                            , mpi_world_cart    &
                            , ierr              &
                            )
    call MPI_Comm_rank(mpi_world_cart,rank,ierr)
    call MPI_Comm_size(mpi_world_cart,size,ierr)
    call MPI_Cart_shift(mpi_world_cart, 0, 1, src, dest,ierr)
    call MPI_Finalize(ierr)
    print *,rank,"of",size,"|",src,"of",dest,"|",(ierr==0)
end program main
```

输出

```shell
           6 of           9 |           3 of          -1 | T 
           1 of           9 |          -1 of           4 | T
           2 of           9 |          -1 of           5 | T
           0 of           9 |          -1 of           3 | T
           5 of           9 |           2 of           8 | T 以此条为例，5从2接收，发送到8
           4 of           9 |           1 of           7 | T
           3 of           9 |           0 of           6 | T
           7 of           9 |           4 of          -1 | T
           8 of           9 |           5 of          -1 | T
```

第一维度的period设置为.false.，导致网络在第一维度不是环状的，而是线性的；

第二维度的period设置为.true.，导致网络在第二维度是环状的，最后一个进程的下一个进程是第一个进程。

这种设置方式会产生一个类似圆柱形的结构，圆柱的底是头尾连接的，圆柱的高是头尾分离的。

因为MPI_Cart_shift选取的维度是第一维度，且period(1)==.false.，所以0\~2进程的src为-1，6\~8进程的dest为-1。

以下是period(1)==.true.时程序的输出。

```shell
           0 of           9 |           6 of           3 | T
           6 of           9 |           3 of           0 | T
           8 of           9 |           5 of           2 | T 8从5接受，发送到2
           7 of           9 |           4 of           1 | T
           1 of           9 |           7 of           4 | T
           3 of           9 |           0 of           6 | T
           2 of           9 |           8 of           5 | T
           4 of           9 |           1 of           7 | T
           5 of           9 |           2 of           8 | T
```

MPI_Cart_Create函数的reorder参数是用来指定创建出来的Cartesian拓扑是否可以被重新排序的。具体来说，如果reorder参数被设置为1，那么MPI库就可以为了提高性能而重新排列进程的拓扑结构。如果reorder参数被设置为0，那么MPI库就必须按照进程的原始顺序来创建Cartesian拓扑结构。

当reorder参数被设置为1时，MPI库可以为了提高性能而重新排列进程的拓扑结构。例如，假设原始的进程布局如下所示：

```text
0 1 2 3
4 5 6 7
8 9 10 11
```

如果reorder参数被设置为1，MPI库可以重新排列进程的拓扑结构，例如：

```text
0 4 8 9
1 5 6 10
2 3 7 11
```

这样做的目的是为了使相邻的进程在物理空间中更接近，从而减少通信延迟。但是需要注意的是，重新排序进程的拓扑结构可能会影响到原始程序的正确性，因此在使用该特性时需要小心谨慎。

