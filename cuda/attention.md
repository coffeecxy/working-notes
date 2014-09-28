编程的时候要注意的地方
===
CUDA C编程不同于传统的C编程，因为其需要有并行的特性，所以有些地方需要特别的注意

## 传入kernel的struct的问题
这个问题我弄了很久才搞明白。

一般情况下，我们传入kernel的参数都是一些基本的数据类型或者是这些基本数据类型的指针，比如说`double,double*`之类的,但是这样传参数的一个问题就是要传入的参数的个数有时候会太多了，比如我们需要完成一个矩阵的乘法，那么多半就需要使用如下的kernel，其中`A`是`M*P`的，`B`是`P*N`的，结果是`M*N`的。

```CUDA
__global__ void gpuMM(float *A, float *B, float *C, int N,int P,int M)
```
