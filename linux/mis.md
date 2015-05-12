# mis

## manual的安排

    MANUAL SECTIONS
        The standard sections of the manual include:

        1      User Commands
        2      System Calls
        3      C Library Functions
        4      Devices and Special Files
        5      File Formats and Conventions
        6      Games et. Al.
        7      Miscellanea
        8      System Administration tools and Deamons

        Distributions customize the manual section to their specifics,
        which often include additional sections.

需要注意的是,2的是系统调用.也就是在内核实现的时候,在x86中,使用int 80h中断,并使用不同的功能号区分的那些函数. 它们在内核代码中使用sys_xxx表示的.

3是C的库函数,也就是Linux发行版一般都会提供的库函数.这些函数是包括了标准C中的函数(libc) 还有比如pthread中的函数.


5表示某些程序,特别是daemon需要使用的配置文件的格式,其一般都会在这个文件的man 8后面的see also中给出.

8是管理系统需要使用的一些程序和工具.一般是一些daemon之类的.
