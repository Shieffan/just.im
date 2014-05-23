---
title: Ubuntu/Debian下映射Capslock为Ctrl并保留Capslock功能
layout: post
guid: urn:uuid:b0bbb189-2569-4a6b-ae57-7b8cf8913dfa
tags:
  - ubuntu
  - linux
  - Capslock
---

无论是对Vim党还是Emacs党，无论是对tmux党还是awesome党来说，不少人都觉得现代键盘的Ctrl键位实在是太难过了。

所以我打算把Capslock映射为Ctrl,但是同时通过另外的按键组合来实现Capslock的功能，免得输入大段大写的时候一直难过地按Shift.

首先执行

```bash
xmodmap -pke | grep 'Caps_Lock'
```

查看当前Caps\_Lock的keycode,下面是我自己的命令输出：

> keycode  66 = Caps\_Lock NoSymbol Caps\_Lock

这样就知道了Capslock键的keycode为66,于是在~目录下创建`.Xmodmap`文件，文件内容为：

```
keycode 66 = Control_L Caps_Lock
clear lock
add control = Control_L
```

这样，就将CapsLock键映射为了Ctrl键，同时不影响原有的Ctrl，同时通过Shift+CapsLock的按键组合，仍然可以激活CapsLock的功能。我个人比较喜欢这样的键位跟组合。

还有一种比较常用的映射方式，把Capslock映射为Ctrl，把Shift+Shift映射为CapsLock：

```
keycode 66 = Control_L NoSymbol
keycode  50 = Shift_L Caps_Lock Shift_L
clear lock
add control = Control_L
```
这种映射的触发方式为先按住右侧的Shift，再按一下左侧的Shift，然后就触发了Casplock的功能。注意这种映射可能会对某些程序不兼容，表现出来的现象就是单独按左侧Shift并没有触发相应的keycode.

其实我最喜欢的映射应该是把连续两次按Capslock映射为Capslock，可是Linux下的xmodmap好像没有这种映射方式。罢了。

最后通过`xmodmap ~/.Xmodmap`即可实时生效，重启后xmodmap会自动读取~目录下的.Xmodmap文件，所以重启后也会自动生效。当然，仅限与Debian系与某些其它分发版。
