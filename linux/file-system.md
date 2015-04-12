# file-system


## 文件系统的挂载与卸载

### /etc/fstab
linux中的文件系统可以在系统启动的时候被自动挂载,也可以在使用的时候动态的挂载.`/etc/fstab`中指定的文件系统就可以在系统启动的时候自动挂载.如果要在运行的时候挂载一个文件系统,那么就会使用`mount`命令.


    # <file system> <mount point>   <type>  <options>       <dump>  <pass>
    # / was on /dev/sda7 during installation
    UUID=f5a71034-df20-4559-97af-5b658b58f997 /               ext4    errors=remount-ro 0       1
    # swap was on /dev/sda8 during installation
    UUID=2c37026a-c36a-41d9-9999-8a9740fb6ec7 none            swap    sw              0       0


上面是我当前的系统的fstab文件的内容.注意其中使用了文件系统的UUID,而不是像`/dev/sda1`之类的设备文件或者是`LABLE=XXX`这种使用文件系统的label的方式.因为文件系统的UUID肯定是不会改变的,二`/dev/sda1`这种设备名称会因为主机上面多加了一块硬盘或者是其他的设备而被改变.

因为我的系统上面装的是双系统,我想在linux启动的时候,win7的d盘就被挂载到了/media/win7D下面.

那么首先,必须保证`/media/win7D`这个目录是存在的,因为mount必须保证挂载点是一个目录,而且这目录一定要存在,当然,这个目录下面有没有文件都没有关系,因为linux挂载文件系统到一个挂载点的时候,如果这个挂载点下面已经有了内容,那些内容会被隐藏,新挂载的文件系统中的内容会出现.

然后在`/etc/fstab`中将下面的内容加到最后.

    /dev/sda5 /media/winD ntfs defaults 0 0

注意一定不要加到了最前面,因为最前面一定是对`/`挂载点的挂载,如果`/`挂载点上面都没有挂载到文件系统,那么其他的挂载点根本就不会出现,后面的挂载当然不能成功了.

在修改`/etc/fstab`文件后,使用

    sudo mount -a
这个命令会将fstab文件中的挂载命令都一遍,如果有问题的话,说写的语法有问题,*一定要修改正确,不然的话,系统是开不机的*.


### /etc/mtab

实际上,系统中不止挂载了fstab文件中指定的文件系统,当前系统中挂载的文件系统可从`/etc/mtab`得到.

    /dev/sda7 / ext4 rw,errors=remount-ro 0 0
    proc /proc proc rw,noexec,nosuid,nodev 0 0
    sysfs /sys sysfs rw,noexec,nosuid,nodev 0 0
    none /sys/fs/cgroup tmpfs rw 0 0
    none /sys/fs/fuse/connections fusectl rw 0 0
    none /sys/kernel/debug debugfs rw 0 0
    none /sys/kernel/security securityfs rw 0 0
    udev /dev devtmpfs rw,mode=0755 0 0
    devpts /dev/pts devpts rw,noexec,nosuid,gid=5,mode=0620 0 0
    tmpfs /run tmpfs rw,noexec,nosuid,size=10%,mode=0755 0 0
    none /run/lock tmpfs rw,noexec,nosuid,nodev,size=5242880 0 0
    none /run/shm tmpfs rw,nosuid,nodev 0 0
    none /run/user tmpfs rw,noexec,nosuid,nodev,size=104857600,mode=0755 0 0
    none /sys/fs/pstore pstore rw 0 0
    cgroup /sys/fs/cgroup/cpuset cgroup rw,relatime,cpuset 0 0
    cgroup /sys/fs/cgroup/cpu cgroup rw,relatime,cpu 0 0
    cgroup /sys/fs/cgroup/cpuacct cgroup rw,relatime,cpuacct 0 0
    cgroup /sys/fs/cgroup/memory cgroup rw,relatime,memory 0 0
    cgroup /sys/fs/cgroup/devices cgroup rw,relatime,devices 0 0
    cgroup /sys/fs/cgroup/freezer cgroup rw,relatime,freezer 0 0
    cgroup /sys/fs/cgroup/net_cls cgroup rw,relatime,net_cls 0 0
    cgroup /sys/fs/cgroup/blkio cgroup rw,relatime,blkio 0 0
    cgroup /sys/fs/cgroup/perf_event cgroup rw,relatime,perf_event 0 0
    cgroup /sys/fs/cgroup/net_prio cgroup rw,relatime,net_prio 0 0
    cgroup /sys/fs/cgroup/hugetlb cgroup rw,relatime,hugetlb 0 0
    systemd /sys/fs/cgroup/systemd cgroup rw,noexec,nosuid,nodev,none,name=systemd 0 0
    /dev/sda5 /media/winD fuseblk rw,nosuid,nodev,allow_other,blksize=4096 0 0

上面是我的系统当前挂载的文件系统,实际上,上面的文件系统中,除了第一个和最后一个对应到了物理的硬盘分区以外,其他的都是存在于内存中的.
比如挂载在`/proc`上的文件系统,其类型为proc,这个文件系统只存在于内存中,里面包含了当前内核的状态信息.
那么文件系统类型为`tmpfs`的文件系统,其也在内存中,在其挂载点的目录下面进行文件的读写,速度会相当快,但是系统重启之后就没有了.

## 硬盘的物理结构和分区

在linux中,默认使用的文件系统就是ext2/3/4,它们其实可以看成是一种文件系统,因为3是兼容2的,4又是兼容3的.在近几年的linux中,都是使用ext4作为起默认的文件系统了.

首先,对于硬盘,在物理上,其最小的读写单位是sector,一个sector的大小是512B.硬盘是一个3维的构造,将硬盘看成是一个圆柱体,第一个维度为磁柱,也就是从上往下看,那样一圈一圈的.第二维为磁头,就是在每个磁柱上有很多层,每层都有一个磁头,第三维为扇区,就是在特定磁柱的特定磁头上,对这一圈上面的扇区进行了编号.

    Disk /dev/sda: 2000.4 GB, 2000398934016 bytes
    255 heads, 63 sectors/track, 243201 cylinders, total 3907029168 sectors

上面是我的硬盘的信息,我的是一个2TB的硬盘,一共有243201个磁柱,255个磁头,63个磁道,它们三个的乘积就是总的sector的数目.

系统在对硬盘进行IO的时候,都是以sector的整数倍为单位进行的.

    I/O size (minimum/optimal): 4096 bytes / 4096 bytes

在这块硬盘上,就是以4K为单位进行IO的.

上面是硬盘的硬件结构.

       Device Boot      Start         End      Blocks   Id  System
    /dev/sda1   *        2048      206847      102400    7  HPFS/NTFS/exFAT
    /dev/sda2          206848   419432447   209612800    7  HPFS/NTFS/exFAT
    /dev/sda3       419432448  1648232447   614400000    7  HPFS/NTFS/exFAT
    /dev/sda4      1648234494  3907028991  1129397249    5  Extended
    Partition 4 does not start on physical sector boundary.
    /dev/sda5      1648234496  2678226943   514996224    7  HPFS/NTFS/exFAT
    /dev/sda6      2678228344  3694789349   508280503    7  HPFS/NTFS/exFAT
    /dev/sda7      3694790656  3890102271    97655808   83  Linux
    /dev/sda8      3890104320  3907028991     8462336   82  Linux swap / Solaris

上面是我的硬盘的分区信息.硬盘的第0个sector被成为MBR,其中448B存储的是从这块硬盘启动操作系统的一小段代码,剩下的64B是用来存储这块硬盘的分区信息.每个分区信息需要16B,所以可以存储4个分区信息.但是我们的硬盘上面不止4个分区,这是因为这4个分区中可以有一个是扩展分区,扩展分区相当于是一个指针,其指向的那个sector中可以包含其他的分区信息,所以理论上一个硬盘上的分区数目可以是无限的,但是一般都是有限制的.

分区必须以磁柱为单位进行.

硬盘进行了分区之后都需要对分区进行格式化才能使用,在对分区进行格式话的时候,都需要指定这个分区使用的文件系统,不同的文件系统对于如何在这个分区上面进行文件操作是很不相同的.

### ext2文件系统的组成

为了方便存储,文件系统都会将这个分区分成很多个block,每个block的大小一般都会和硬盘的I/O大小一样,都是4096B. block会被编号,其编号是从0开始的. 需要注意的是,在第0个block之前,会有一个512B的启动分区,这个启动分区和硬盘的MBR的启动功能相同,这样这个硬盘就可以使用不同的启动引导程序了.

在ext2分区中,一个inode表示一个文件,其中会包含这个文件的各种属性,比如权限,大小,修改时间之类的,其还会指明这文件的数据在哪些block中.

因为一个分区可能很大,也就是其包含的block数目可能很多,如果将所有的inode放在一起的话,那么这个文件系统的前面很大一部分都是存放的inode table,管理起来很不方便. 所以在ext2中,将所有的block分成了很多个block group,我的分区中,每个block group的大小是32768个block.

0-32767这个block group中的第一个block为super block,起记录了整个文件系统的信息,比如总的block,inode数目,每个block,inode的大小等等.

接着的几个block是这group的描述信息,比如这个group的开始和结束的block之类的.

后面是block bitmap,这个bitmap是方便一个新的文件要使用block的时候文件系统查看的,每个block会使用一个bit来标识,如果这个bit为0,表示这block还没有使用,如果为1,表示这个block已经被使用了,这样就会很快的知道可以使用那些block来存放这个新建的文件.

后面是inode bitmap,和block bitmap类似,当新建一个文件的时候,需要一个inode,通过查看inode bitmap,可以很快的知道可以使用那个inode.

> 通过查看一个group的inode bitmap,可以发现使用了的基本都在前面,因为一个文件之后使用一个inode,文件的增加与删除都不会产生inode上的碎片. 但是查看其block bitmap,特别是那些使用了很久的硬盘,会发现使用了和没有使用的block在group中很杂乱,这是因为有些文件很大,其会使用掉很多的block,在其被删除之后,然后又有一些小的文件建立之后,就会出现这种过情况了.

后面是inode table,在我的系统中,每个inode占用了256B,所以一个block中可以存放16个inode,在我的系统中,一共使用了512个block来存放inode table,所以一共有8192个inode. 也就是说,这group中一共可以存放8192个常规文件和目录文件.

后面是data block,也就是文件和目录的内容存放的地方,在我的系统中,一个group中一共有31200个block是属于data block的.

---

第二个block group从32768-65535,也是32768个block,其布局和第一个block group一样.

要注意的是,第二个(以及后面的所有block group)的第一个super block,都是对第一个block group的备份. 因为一个文件系统只需要一个super block来记录其基本的信息,但是如果只有一个super block,如果第一个block group的superblock出现问题了,那么这个文件系统就完全失效了,所以后面的备份还是很有用的.

后面的block group就类似了.




