---
title: Linux获取当前用户的id或username
layout: post
guid: urn:uuid:b02d3895-cd2f-470f-a990-28e2efeb1bc5
tags:
  - linux
  - linux command
  - whoami
  - uid
---

Linux要获取当前用户的信息很简单，使用`id`命令即可：

```bash
~$ id
uid=1000(xifan) gid=1000(xifan) groups=1000(xifan),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lpadmin),124(sambashare)
```

可以看出id命令的输出比较详细，会包含当前用户的uid，username，gid，group name，以及附属组及组名。同时，id命令也提供了丰富的选项用于只输出其中的某项信息。

> -u 输出当前用户的uid  
> -g 输出当前用户的Primary group的gid  
> -n 以name形式输出uid或gid  

```bash
~$ id -u  #输出当前用户的uid
1000
~$ id -u -n  #以name形式输出当前用户的uid，即输出username
xifan
~$ id -g   #输出当前用户的Primary group的gid
1000
~$ id -g -n   #输出当前用户的Primary group的group name
xifan
```

除了通过id命令，我们还可以通过环境变量获取当前用户信息，一般情况下，Linux系统会提供`USER`,`UID`,`EUID`,`GID`,`EGID`等环境变量来表示用户信息。在我的系统下：

```bash
~$ echo $USER
xifan
~$ echo $UID
1000
~$ echo $GID
1000
```

获取当前用户username还有一个最简单的方法：

```bash
~$ whoami
xifan
```

Linux下关于获取用户信息的程式还有`w`,`who`,`finger`等，其中`finger`程式一般不自带，需要单独安装。而一般linux发行版都会带有`who`程式，`who`程式的输出比较简单，可以在shell脚本里放心使用。
