# attr

## init,destroy

    int pthread_attr_init(pthread_attr_t *attr);
    int pthread_attr_destroy(pthread_attr_t *attr);

pthread_attr_init会初始化attr指向的pthread_attr_t. 这个函数在OO中就是完成构造函数的功能. 初始化的时候,attr中的各个域会被初始化为默认值. 在这个调用之后,可以使用其他的一堆pthread_attr_setxxx函数来改变attr中各个域的值. 经过初始化之后,attr可以被用在`pthread_create`中(attr也就是用来创建一个线程的).

对一个已经被初始化的attr进行pthread_attr_init操作会导致不定的结果.(不要这样做).

当一个attr不需要的时候,需要用pthread_attr_destroy来清除,也就是OO中的析构函数. 删除一个attr不会对已经使用这个attr新建的线程产生影响.

当一个attr被清除了之后,可以重新使用pthread_attr_init来进行初始化.

---
注意,pthread_attr_t应该被看成是一个不透明的数据结构,如果直接对其中的域进行访问,会产生不定的结果.(C中所有域的访问都是public的,所以需要我们自己不去访问pthread_attr_t中的各个域. 如果在C++中,那么加入private,就不会有这个事情了.)


## affinity

    int pthread_attr_setaffinity_np(pthread_attr_t *attr,
                      size_t cpusetsize, const cpu_set_t *cpuset);
    int pthread_attr_getaffinity_np(const pthread_attr_t *attr,
                      size_t cpusetsize, cpu_set_t *cpuset);

affinity是亲和力的意思. 设置和得到attr指向的pthread_attr_t中的affinity.

cpusetsize是第三个参数占用的字节数,一般都是`sizeof(cpu_set_t)`(这个是C中一个比较冗余的地方,没有办法,只能这样使用).

第三个参数是要设这或者得到的cpu_set_t.

## detachstate

    int pthread_attr_setdetachstate(pthread_attr_t *attr, int detachstate);
    int pthread_attr_getdetachstate(const pthread_attr_t *attr, int *detachstate);

设置和得到attr中的detachstate属性. 其可能的值为

* PTHREAD_CREATE_DETACHED, 使用attr创建的线程会处于detached状态.
* PTHREAD_CREATE_JOINABLE, 使用attr创建的线程会处于joinable的状态.

默认值为PTHREAD_CREATE_DETACHED.

## guardsize

    int pthread_attr_setguardsize(pthread_attr_t *attr, size_t guardsize);
    int pthread_attr_getguardsize(const pthread_attr_t *attr, size_t *guardsize);
设置和得到attr中的guardsize.

如果guardsize是大于0的. 那么使用attr来创建的线程在其stack的后面至少会有一个page大小的guard.

## inheritsched

int pthread_attr_setinheritsched(pthread_attr_t *attr,
                                int inheritsched);
int pthread_attr_getinheritsched(const pthread_attr_t *attr,
                                int *inheritsched);
