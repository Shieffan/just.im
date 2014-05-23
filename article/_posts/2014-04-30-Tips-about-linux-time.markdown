---
title: Linux的系统时间是如何确定的
layout: post
guid: urn:uuid:87215fec-ac93-4d4c-a17e-5b265824c3e9
tags:
  - linux
  - UTC
  - timezone
  - time
---

大学的时候喜欢体验各种不同的Linux发行版，经常会遇到装了系统之后，系统的时间错乱，尤其是与windows共存时，两个系统间的时间总是有时差。经过一段时间的摸索，就会留意在安装的时候把__使用UTC__的勾去掉，这样就一切正常了。今天仔细研究了下这个问题，总结一下Linux对系统时间及时区的处理。

首先，是三个名词：

+ UTC,称为世界标准时间或世界协调时间。其在时间值上与GMT(格林威治)时间是一样的。
+ localtime 指当地时间
+ Real Time Clock, 或者叫Hardware Clock,硬件时钟，指计算机在BIOS的cmos里所保存的时间，因为历史等原因，CMOS里仅仅保存时间值，譬如`2014-05-01 14:44:44`这种具体的时间值，并不保存时区等信息。
+ System Clock，系统时钟，操作系统在启动之后，会在系统内核里以自己的格式保存时间信息，对于Linux系统，会保存从UTC时间1970年1月1日起经过的秒数，以此来作为系统的标准时间以给上层软件提供时间支持。

下面结合操作系统的启动过程，来解释一下这几个不同层面的时间之间的关联。

首先，主板厂商在会在主板出厂时设置好时钟人。主板厂商可能会按照销售地的时区来设置时钟，譬如主板在欧洲生产但是销往中国，厂商极有可能将硬件时钟设置为CST(China Standard Time)时区的当地时间。然而这个只是我们的期望，厂商完全可以自主决定如何设置主板时间。再者，硬件时间很有可能是混乱或者完全错误的，BIOS的刷新，CMOS的放电，可能都会重置BIOS所保存的硬件时钟信息。

在安装操作系统时，一般都会让我们会设置地区、时区等信息，大多数的操作系统也会让你选择是否使用UTC(Windows没有，一会儿另说).

Linux系统启动时，会读取BIOS里的RTC时钟信息，然后根据是否使用UTC，做出不同的处理：

+ 假设使用UTC，Linux系统会认为从BIOS里读取到的RTC时间信息为UTC时间，然后将此时间值减去UTC时间1970-01-01 00:00:00，将计算出的时间差以秒的形式记录在系统内核里，操作系统把此时间作为系统启动时的系统时间，随后的系统运行过程中，将不再读取BIOS里的RTC时钟，而是通过时钟中断计数来维护系统时钟。当系统时钟确定以后，操作系统会根据用户的时区设置，计算出当地时间，最后把这个时间信息呈现给用户。
+ 假设不使用UTC，Linux系统会把从BIOS里读取到的RTC时间认为是localtime，即用户所设置的时区的当地时间，之后，根据用户设置的时区信息，系统将这个RTC时间加上或者减去一定的小时，从而计算出UTC时间，之后计算出自UTC 1970年起的时间戳，从而得出系统时间。这种情况下，最终呈现给用户的localtime，其实就是从BIOS里面读出来的硬件时钟信息。

对于Windows系统来说,系统在启动时，同样要读取RTC信息，windows会认为这个时钟信息就是用户所设置时区的localtime，最终windows就是把RTC时间作为localtime反馈给用户。

所以问题就来了，当Linux使用UTC时，系统会将硬件时钟认为是UTC时间，而windows则将硬件时钟为localtime，这样，必然导致时间的不一致(对于中国东八区，windows跟Linux就有了八个小时的时差)。这个问题怎么解决呢，最简单的就是安装Linux时不要勾选__使用UTC__。当然，安装之后也可以调整设置是否使用UTC时间，一些发行版可以修改`/etc/sysconfig/clock`文件，将UTC设为false，在ubntu中,是修改`/etc/default/rcS`文件。

---

###Some tips

1.如何在Linux系统中查看当前的详细时间信息：

```bash
sudo timedatectl
> time: 三 2014-04-30 16:13:10 CST
> Universal time: 三 2014-04-30 08:13:10 UTC
> RTC time: 三 2014-04-30 16:13:10
> Timezone: Asia/Chongqing (CST, +0800)
> NTP enabled: yes
> NTP synchronized: no
> RTC in local TZ: yes
> DST active: n/a
```

2.查看当地时间:`date`

3.查看硬件时钟:`hwclock --localtime`或者`sudo timedatectl | grep 'RTC time'`

3.查看系统维护的UTC时间:`date -u`
