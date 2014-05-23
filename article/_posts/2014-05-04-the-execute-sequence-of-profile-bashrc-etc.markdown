---
title: Linux下与bash有关的profile及bashrc等文件的执行顺序
layout: post
guid: urn:uuid:895e3489-07f1-4201-92b5-f6d70b59c481
tags:
  - linux
  - bash
  - profile
  - bashrc
  - ubuntu
---

Linux下有关bash的配置文件有一大堆，列举一下，大致有以下这些:

+ /etc/profile
+ /etc/profile.d/\*
+ ~/.bash\_profile
+ ~/.bash\_login
+ ~/.bashrc
+ ~/.profile
+ ~/.bash\_logout

这些文件究竟何时被执行，其执行顺序又是如何呢？

首先，先介绍以下几个概念:

+ **login shell**: 简单地说，通过完整地登入流程而进入的shell，我们称之为login shell。在你登入一个login shell时，会让你输入用户名、密码等信息，通过一套完整的登入机制，最终进入到shell。一般来说，以下场景下打开的shell为login shell:
  - 当用户登入系统时，会打开这个用户的默认shell，这个shell为login shell
  - 当我们通过ssh远程登陆并打开一个shell时，这个shell也是login shell
  - 在终端下使用--login选项执行bash，打开的为login shell
  - 使用`su -`切换用户时，获得此用户的login shell。如果不使用`-`，则获得non-login shell
+ **non-login shell**: 非login shell即为non-login shell，打开一个non-login shell时，一般没有完整的用户认证过程。譬如登入系统后打开Terminal或者Konsole所直接进入的shell，就是non-login shell

按照Shell是否是交互的，又可以分为**interactive shell**和**non-interactive shell**，顾名思义，interactive shell是交互式的，用户输入命令并Enter，shell解释执行这行命令，然后继续等待用户输入新的命令。而non-interactive shell就是读取脚本并且执行脚本的shell.

说完与shell相关的概念，回到正题，来说说以上与bash有关的文件的执行顺序。

##对login shell而言

以上文件的执行顺序如下：

1. 执行/etc/profile文件
2. /etc/profile脚本内一般会`source /etc/profile.d/*.sh`，这样，/etc/profile.d/目录下的shell脚本会被执行
3. 如果~/.bash\_profile文件存在，则执行此文件，执行完毕跳到第6步
4. 如果~/.bash\_login文件存在，则执行此文件，执行完毕跳到第6步
5. 如果~/.profile文件存在，则执行此文件，执行完毕跳到第6步
6. 初始化shell环境完成，进入shell

可见，对于login shell来说，~/.bash\_profile， ~/.bash\_login， ~/.profile文件并不是依序依次执行，而是按照优先顺序，选取存在的优先级最高的一个文件执行，执行完毕后不再执行其它低优先级的文件。

下面我们来验证这个结论。我们依次在/etc/profile， ~/.bash\_profile， ~/.bash\_login， ~/.profile文件里export进一个环境变量：

```bash
~# echo 'export TEST="/etc/profile"' >> /etc/profile
~$ echo 'export TEST="~/.bash_profile"' >> ~/.bash_profile
~$ echo 'export TEST="~/.bash_login"' >> ~/.bash_login
~$ echo 'export TEST="~/.profile"' >> ~/.profile
```

###验证/etc/profile被执行

将~/.bash\_profile， ~/.bash\_login， ~/.profile文件重命名为其它文件，然后登出系统再登入shell，或者使用`bash --login`重登入shell，保证打开的为login shell

```bash
~$ mv .bash_profile .bash_profile.bak
~$ mv .bash_login .bash_login.bak
~$ mv .profile .profile.bak
~$ bash --login
```

接着输出TEST变量观察：

```
~$ echo $TEST
/etc/profile
~$ exit
```

说明/etc/profile文件得以执行。

###验证~/.bash\_profile被执行

将~/.bash\_profile.bak文件重命名回~/.bash\_profile，重新登入一个login shell，输出TEST变量观察：

```
~$ mv .bash_profile.bak .bash_profile
~$ bash --login
~$ echo $TEST
~/.bash_profile 
~$ exit
```

输出的TEST变量为'~/.bash\_profile'，说明.bash\_profile文件得以执行并且执行在/etc/profile文件之后，故覆盖了/etc/profile里的TEST环境变量。

###验证~/.bash\_login被执行

将~/.bash\_profile重命名为~/.bash\_profile.bak，将~/.bash\_login.bak重命名回~/.bash\_login，重新登入一个login shell，输出TEST变量进行观察：

```
~$ mv .bash_profile .bash_profile.bak
~$ mv .bash_login.bak .bash_login
~$ bash --login
~$ echo $TEST
~/.bash_login
~$ exit
```

说明~/.bash\_login文件被执行

###验证~/.profile文件被执行

验证方法同上，不再赘述

###验证~/.bash\_profile，~/.bash\_login， ~/.profile并非依序全部执行，而是优先择一执行

将重命名后的上述文件改回原名，即~/.bash\_profile，~/.bash\_login， ~/.profile，删除之前在其中export的环境变量，然后在三个文件中分别export入一个不同的环境变量。

```
~$ echo 'export TEST1="TEST1"' >> ~/.bash_profile
~$ echo 'export TEST2="TEST2"' >> ~/.bash_login
~$ echo 'export TEST3="TEST3"' >> ~/.profile
~$ bash --login
~$ echo $TEST1
TEST1
~$ echo $TEST2

~$ echo $TEST3

~$ exit 
```

根据以上输出可以看出，当存在~/.bash\_profile文件时，~/.bash\_login， ~/.profile文件并没有执行，故环境变量TEST2，TEST3不存在。

按照上述测试步骤，可以验证得出这三个文件的优先级别。当一个高优先级别的配置文件存在时，仅会执行这个高优先级的配置文件。

###~/.bashrc哪里去了呢?

对于login shell来说，按照其执行流程，并不会**主动**执行到~/.bashrc文件。


> **注意**: 也许上述一系列的验证结果与你观测到的不符，原因在于一些第三方软件对这些配置文件的修改可能导致这些bash配置文件相互关联。  
> 举例来说，ruby管理器rvm的安装会修改.bash\_profile文件，使.bash\_profile文件source ~/.profile文件，所以在执行.bash\_profile文件时会执行到.profile文件。   
> 另一方面，.profile文件一般会source ~/.bashrc，这样，虽然login shell在初始化过程中并不会主动读取执行~/.bashrc，但是因为~/.profile的执行，连带引起.bashrc文件的执行。


##对于non-login shell而言

其会依次执行/etc/bash.bashrc，~/.bashrc文件，除此之外，其不会**主动**执行任何其它配置文件。

要验证这个结论很简单，你可以在其他几个文件(~/.bash\_profile等)里export一个环境变量并打开一个non-login shell(譬如在gnome下打开terminal或者在kde下打开konsole)，你会发现在其它几个文件里export的环境变量在此non-login shell里并不存在。

```bash
~# echo 'export TEST="/etc/profile"' >> /etc/profile
~$ echo 'export TEST="~/.bash_profile"' >> ~/.bash_profile
~$ echo 'export TEST="~/.bash_login"' >> ~/.bash_login
~$ echo 'export TEST="~/.profile"' >> ~/.profile
~$ echo 'export TEST_BASHRC="~/.bashrc"' >> ~/.bashrc
~$ bash   #打开一个新的non-login shell
~$ echo $TEST

~$ echo $TEST_BASHRC
~/.bashrc
~$ exit
```

可以看出，环境变量TEST不存在，说明当打开一个non-login shell时，并不会执行到/etc/profile，~/.bash\_profile等。而TEST\_BASHRC存在，说明~/.bashrc文件得到了执行。

##还有些什么？

`/etc/environment`: 这个文件不像上述文件，它不是个脚本文件，而是用键-值对来设置环境变量。一般来说，只有在系统启动时会读取这个文件，并在整个系统级别上设置环境变量。注意，这个过程发生在用户登入Shell前，故用户在/etc/profile，~/.bash\_profile等文件的设定可能会覆盖/etc/environment里的环境变量设定。

`~/.bash_logout`: 当用户登出一个login shell时，会执行~/.bash\_logout。

###Some tips

1. 我们可能经常需要配置一些环境变量，到底该写在哪个文件里呢？
  
   如果你希望写入的环境变量对系统内的所有用户可用，那么：   

   + 如果存在/etc/environment文件，那么考虑在这里设置系统级别的环境变量。
   + 一般不要直接在/etc/profile文件里export环境变量，因为系统升级可能会覆盖这个文件。
   + 可以将一些自定义方法或者环境变量写在脚本里，然后放入/etc/profile.d/文件夹内。这个文件夹内的脚本会被/etc/profile source并执行。

2. 我只在~/.bash\_profile(或~/.profile)里导入过环境变量，为什么在打开的terminal里也能访问到这个环境变量呢？终端模拟器里打开的不是non-login shell么？

   一般情况来说打开终端模拟器后默认打开的Shell为non-login shell，然而包括terminal以及terminal打开的shell，都是最开始登入系统时的login shell的子进程，子进程会继承父进程的环境变量，所以即使在.bashrc里没有export某些环境变量，登入系统后打开的non-login shell依然有可能访问到~/.bash\_profile，~/.profile里export的环境变量。

3. 我使用zsh而不是bash，请问zsh相关的配置文件是如何执行的呢？

   可以参考本文的分析来自行测试并得出结论。zsh的配置文件的执行顺序与本文分析的bash基本一致。
