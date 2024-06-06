date: 2022-11-17 14:36:31

tags: makefile wildcard filter-out patsubst

## 文件列表
```shell
main.f90
makefile
sub1.f90
sub2.f90
sub3.f90
```
## makfile代码
```makefile
main = main.f90

SRC = $(wildcard *.f90)

SRC := $(filter-out $(main),$(SRC))

SRC := $(patsubst %.f90, %.o, $(SRC))

main:

    echo $(SRC)
```
* wildcard
	在对变量调用时，保持通配符特性
* filter-out
	过滤器，删除后面跟的第一个参数
* patsubst
	将变量中元素根据参数替换，上述代码将所有.f90替换为.o
## 执行结果
```shell 
$> make
echo  sub1.o  sub2.o  sub3.o
 sub1.o  sub2.o  sub3.o
```