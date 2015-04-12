# cuda runtme api

使用C来进行CUDA GPU的编程的时候,有两套API可以使用.
一套是cuda driver api,这套api中提供全部都是标准C函数,但是使用起来是比较麻烦的.
所以cuda还提供了一套cuda runtime api,这套api会对c的语法进行一定的修改,也就是使用的cuda c,但是这套api使用起来会方便很多.

按照NVIDIA官方的说法,使用这两套api能达到相同的效果,所以基本我们都不会使用driver api,都是使用runtime api.

## CUDA C RUNTIME

CUDA的runtime（也就是提供cuda函数的一个library，要么是动态的DLL，要么是静态的lib）包含了很多的用于不同目的的函数接口.

这两个库都存在于`$CUDA_PATH/lib/arch`目录下,其中`CUDA_PATH`环境变量表示的是cuda toolkit安装的根目录,arch表示系统架构,要么是win32,要么是x64.
它们的名字是`cudart.lib`和`cuda_static.lib`,分别是动态链接和静态链接的时候使用的.

可以发现,`cuda_static.lib`比较大,因为其是会被静态链接到最后的可执行文件中.
而`cudart.lib`比较小,因为运行的时候还会使用对应的dll文件.

## 运行时初始化

使用过其他库的都应该知道，要使用一个库的功能，这个库一般都会提供一个初始化函数，然后其他的所有函数都需要在调用了这个函数之后才被调用，当不使用这个库的功能了之后，要调用一个对应的destroy函数来清除这个库使用的资源.

cudart这个库不需要显式的调用这样一个函数，当第一个cudart提供的函数被调用的时候，其会自动的被初始化。

要注意的时候，初始化的时候每个device会被创建一个context，然后这个进程的**所有的thread**在访问这个device的时候都是用这个context.

## 异步的并发执行
在host中的代码执行是串行的，然后在这些代码中，会有对kernel的调用，这样device就也要开始执行代码了。

要注意的是，对于kernel的调用，其不是等到分配给device的代码运行完了才返回，而是在调用了之后的某个时刻就返回了。

实际上，下面的调用都是异步的。

+ 调用kernel
+ 在一个device中，进行memory copy
+ 从host copy数据到device，如果数据大小为64kb及以下
+ 在device中的memset函数
+ 以async开头的函数

我们一般都想让CPU和GPU并行的运行,这样才能达到更高的系统利用效率,因为如果CPU和GPU之间有相互的等待的话,那么就还有优化的空间.

在上面描述的情况中,我们假设CPU使用的是单线程,实际上,通过使用多线程编程技术,我们可以让多核的cpu也同时运行.在下一部分中的cuda+openmp就是介绍这个技术的.

kernel的并行运行

当一个device的
`concurrentKernels`属性为1的时候，表示其支持同时运行几个kernel。意思是host在一个kernel还没有返回的时候，可以又发出一个kernel的调用，它们会同时运行。


一个典型的情况是将一个kernel发送几次，这样就可以同时运行几个了。

当然，我们需要使用一些同步的方法来让所有的thread

### Page_locked host memory
我们知道在现代的操作系统中，为了更好的使用有限的内存服务于多个进程，所以将内存进行了分页,page。而 page_locked的host memory是强制的让OS将一块内存不进行分页然后给我们的应用使用。对于这个不分页的内存，在某些device中，可以使用一个叫做mapped momory的技术，让这部分内存和device的global memory完成一个双向的binding，这样的话就免去了先将host memory中的数据copy到device memory中，完成了算法之后又copy回来的这种操作。

但是一个不幸的事情是，OS能分配给一个应用的page_locked memory是有限的，因为系统为了保证其他的应用和自身的运行，必须让大部分的内存变成page的（因为page的内存可以被缓存，而且可以在内存不够的时候交换到硬盘上），所以说，虽然这是一个很好的技术，但是在当前的OS架构上面，其应用只能是有限的。

在提供的API中，`cudaHostAlloc`和`cudaFreeHost`是用来完成page locked memory的操作的。

### Multi-device system
为了增加运算能力，我们可以使用了GPU的阵列，这样在一个系统上面就有了多个device，使用下面的代码可以知道系统上面有多少个device。
```
int deviceCount;
cudaGetDeviceCount(&deviceCount);
```
使用cudaGetDeviceProperties可以得到一个device的属性。
```
cudaDeviceProp deviceProp;
cudaGetDeviceProperties(&deviceProp, device);
```

device memory的分配和释放以及kernel的launch都是在当前的device中进行的，使用`cudaSetDevice()`函数，可以设置当前使用的device。

### Compute mode
对于tesla的GPU，其可以运行在不同的模式下面。

默认模式：OS上面的所有线程都可以使用这个device，也就是他们都可以创建一个context

Exclusive-process模式：OS上只能有一个进程可以在这个device上新建一个context，但是这个进程的所有线程都可以同时对这个context进行操作。

Exclusive-process-and-thread: OS上只能有一个进程可以在这个device上新建一个context，当在一个时刻，只能有一个线程操作这个context。

### Function type qualifiers

因为程序可以运行在host和device中，所以需要指定一个函数运行的地方和被调用的地方。
#### `__global__`
`__global__`是用来声明一个函数是kernel的，其表示这个函数是运行在device中，然后要从host中调用的。

使用global声明的函数的返回值必须是void，所以如果我们想要返回值的话，那么就以指针的方式传入要返回的值。要注意的是，所有的输入参数都是从global memory中传入的，也就是说我们必须先将要交给device处理的数据从host的memory中传入device的global memory中。

#### `__device__`
`__device__`表示这个声明的函数运行在device中，然后从device中调用这个函数。这个意思就是从global函数中可以调用device的函数，从device中也可以调用device函数。

其用处就是当我们要运行在device中的并行算法比较大的时候，可以分成多个部分，每个部分写成一个device声明的函数，这样就看起来结构比较清楚了。
#### `__host__`
使用`__host__`声明的函数表示其运行在host中，也要从host中调用，这个其实就是我们的C的函数。这个是默认的声明方式。也就是如果都没有给出来的话，那么就是`__host__`了。

还有一个就是从device调用然后在host中运行的函数，当是CUDA的架构是不支持这种调用的。

### Variable type qualifier
#### `__shared__`
前面已经说过了，在一个block中的所有thread有一个shared memory，这个memory可以被所有的thread访问，其中定义的变量的life time只有这个block的。



## 主要的API函数的使用介绍



    cudaError_t cudaGetDeviceCount (int *count)

很直观的，这个函数返回的是系统上面的device的个数，也就是那些可以进行CUDA运算的GPU的个数，注意count实际是一个输出参数。
函数返回了一个cudaError_t，看了其定义之后，发现其是一个enum，这个enum中包含了很多个域，也就是所有的那些调用之后需要看到其调用时候成功的函数都会返回中数据类型的。

`cudaError_t cudaGetDeviceProperties(cudaDeviceProp *prop, int device)`

也是很直观的一个函数，device表示要得到哪一个device的属性，返回的属性存入*prop中，cudaDeviceProp是一个struct，里面包含了很多个域。表示这个device的属性，同样的，调用也会返回一个是否成功的标识。

`cudaError_t cudaMalloc (void **devPtr, size_t size)`

在device的global memory中分配size个byte大小的内存，是Linear memory，分配好的内存可以通过*devPtr访问到，这个地方感觉是可以使用一次指针的，但是使用了二次指针，具体的原因就不清楚了。

`cudaError_t cudaFree (void *devPtr)`

和上面的函数对应的，用于释放cudaMalloc申请到的内存

`cudaError_t cudaHostAlloc (void **pHost, size_t size, unsigned int flags)`

让OS给host分配一个size大小page-locked memory，具体的信息在上面已经给出来了。注意后面的flags用来指名这个内存具有的一些性质
？？

`cudaError_t cudaMallocHost (void **ptr, size_t size)`

和上面的函数类似，实际上，当上面的函数flag没有给（也就是default的时候），调用的就是这个函数。

`cudaError_t cudaFreeHost (void *ptr)`

对于上面的两个函数申请到的page locked memory，使用这个函数来释放


`cudaError_t cudaMemcpy (void *dst, const void *src, size_t count, cudaMemcpyKind kind)`

在使用标准C的时候，我们会使用memcpy函数来将内存中一部分的内容copy到另外一个地方去。Copy的大小是size个byte，kind表示哪儿到哪儿去，比如`hosttohost`,`hosttodevice`,`devicetodevice`,`devicetohost`。

`template < class T > cudaError_t cudaFuncSetCacheConfig (T *func, cudaFuncCache cacheConfig)`

这个函数用来设置在一个device function（在device上面运行的function）的preferred Cache，也就是使用L1 CACHE还是shared memory。要注意第一个参数虽然是template，但是我们一定要传一个函数指针才可以。还有注意这个函数必须是使用`__global__`声明的，也就是必须是一个kernel

比如下面的函数调用

```
checkCudaErrors(cudaFuncSetCacheConfig(*func1,cudaFuncCachePreferShared));

cudaError_t cudaFuncGetAttributes (cudaFuncAttributes *attr, const void *func)
```
得到一个function的attribute，这个function也必须是一个__global__声明的，也就是一个kernel。cudaFuncAttributes这个struct中的各个域表示这个kernel的一些属性


`cudaError_t cudaDeviceReset (void)`

对于当前的process，将（当前指定的）device Reset了，也就是下一次的调用将重新初始化一个device context。

需要注意的是，这个函数是立即执行的，而一个process可能有多个thread，如果当前的thread Reset了这个device，那么这个process中的其他thread还在操作的话，那么可以肯定这是会出问题的。

还有就是硬件上可能只有过一个gpu,但是每个进程都可以有一个自己的device context。就是说这个GPU被共用的。


`cudaError_t cudaDeviceSynchronize (void)`

这个函数会block直到分配到device上运行的任务都完成了。

注意这个函数是在host中运行的，一个应用场景就是我们的算法由几步组成，其中的每一步都使用一个kernel来完成，那么在一个kernel调用之后，就使用cudaDeviceSynchronize,等待所有的thread完成，然后再调用下一个kernel。

注意这个函数和`__syncthreads`的区别，`__syncthreads`是在kernel里面使用的，其表示这个kernel在各个thread中运行的时候，在某个点需要所有的thread都达到了才运行到后面去。

在一个应用中，要提出的时候，我们通过都会先调用cudaDeviceSynchronize，等到所有任务都完成，然后调用cudaDeviceReset，这样这个应用（进程）在GPU上面申请的所有资源都会释放。
