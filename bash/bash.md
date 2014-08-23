关于bash的知识
========

bash是使用linux的人都会接触的东西，其要设置的内容也是比较复杂的。

## 初始化
linux中的shell，分为login shell和non-login shell.根据这个分类，得到这个shell的时候，其环境变量是不同的。

定义：

`login shell`：取得bash 时需要完整的登入流程，就称为login shell。举例来说，同tty1~tty6登入时， 需要输入用户名和密码，此时取得的bash就称为login shell

`non-login shell`:取得bash介面的方法不需要重复登入的动作。
举两个例子（1）以X window登入linux后，再以X 的图形化介面启动终端机，此时那个终端机并不需要再次的输入用户名和密码，那个bash的环境就称为non-login shell 
(2)在原本的bash环境中再次下达bash这个指令，同样没有要求输入用户名和密码，那个第二个bash也是non-login shell。

从上面的定义可以知道，我们通常使用的在图形界面下面打开的terminal，都是不需要输入用户名和密码的，所以其都是non-login shell.

要得到login shell,我们可以C+A+F2，然后登陆，这样我们就得到了一个字符界面的login shell。

### /etc/profile,~/.bash_profile
当得到一个login shell的时候，其会读取两个文件。

第一个是全局的，即`/etc/profile`，`/etc/profile`是一个全局的设定，其主要的功能是读取`/etc/profile.d`下面的所有*.sh文件，所有当我们要改变一个全局的设置的是，就在`/etc/profile.d`下面添加一个`custom.sh`，这样下一次登陆的时候所有用户都可以使用了。

第二个是对当前用户的，为`~/.bash_profile`，在这个文件中，我们可以设置当前用户需要的环境变量和该用户登陆时需要自动启动的程序。在这个文件中，其会读取`~/.bashrc`.

### ~/.bashrc,/etc/bashrc
当打开一个non-login shell的时候
* 其只会继承已有的login shell中的环境变量。
* 同时，其会读取`~/.bashrc`中的内容【所以那些需要在用户登陆的时候自动执行的程序不能放在.bahsrc中，因为每次一个non-login shell打开的时候都会执行，这样就会执行很多次了。同理，用户环境变量的设置也不能放到.bashrc，要放到.bash_profile】

在.bashrc中，其会读取/etc/bashrc,在/etc/bashrc中，首先会设置一些bash的外观，然后其会读取/etc/profile.d/*sh来执行。所以，将系统级别的要更改的设置写在/etc/profile.d/custom.sh，那么login shell和non-login shell都可以用到。而将用户级别的环境变量设置写在.bash_profile中，这样用户登陆的时候就可以用到了。
