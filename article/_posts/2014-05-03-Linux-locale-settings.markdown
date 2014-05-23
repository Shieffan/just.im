---
title: Linux的locale环境变量
layout: post
guid: urn:uuid:f79b7ef7-302a-4bbd-b272-dd3b8d0edda6
tags:
  - linux
  - locale 
---

Linux下的软件对国际化拥有很好的支持。我们一般用两个词i18n以及l10n来描述软件的国际化与本地化支持:

+ i18n: 即**I**nternationalizatio**n**,国际化，因为这个单词太长，所以简写为i18n，表示i之后+18个字母+n;
+ l10n: 即**L**ocalizatio**n**,本地化，简写方式同上。

国际化与本地化有着微妙的关系，我们说一个软件有很好的国际化支持，意思是说这个软件可以在全球各个地区，各个语系间使用，有着适用全球的“潜力”。而我们说一个软件可以被本地化，则是说这个软件可以很好地根据本地语言习惯做出调整。i18n与i10n看似两个概念，其实是一回事儿。在微软及IBM等企业中，则会使用全球化来表示此两者的合称。在英文中用g11n表示全球化。详见[维基百科](http://zh.wikipedia.org/wiki/%E5%9B%BD%E9%99%85%E5%8C%96%E4%B8%8E%E6%9C%AC%E5%9C%B0%E5%8C%96)。

在linux里，支持本地化的软件会根据用户的一系列locale环境变量来决定如何对时间、货币、语言等进行格式化输出。要检查自己的locale设定，可以在terminal里输入`locale`命令并回车，在我的终端里输出如下：

```bash
xifan@dodo ~ $ locale
LANG=en_US.UTF-8
LANGUAGE=en_US
LC_CTYPE=en_US.UTF-8
LC_NUMERIC=zh_CN.UTF-8
LC_TIME=zh_CN.UTF-8
LC_COLLATE="en_US.UTF-8"
LC_MONETARY=zh_CN.UTF-8
LC_MESSAGES="en_US.UTF-8"
LC_PAPER=zh_CN.UTF-8
LC_NAME=zh_CN.UTF-8
LC_ADDRESS=zh_CN.UTF-8
LC_TELEPHONE=zh_CN.UTF-8
LC_MEASUREMENT=zh_CN.UTF-8
LC_IDENTIFICATION=zh_CN.UTF-8
LC_ALL=
```

根据上面的输出我们大概可以把这些locale环境变量分为四组：`LANG`,`LANGUAGE`,`LC_*`,`LC_ALL`,应用程序主要使用`LC_*`里的locale环境变量。拥有良好i18n支持的程序通过`setlocale`(定义在**locale.h**文件)函数来设置程序运行时的locale变量。setlocal参数接受两个参数，第一个参数指明是设置哪一个`LC_*`变量，第二个参数指明要给此LC变量设置的值。通常第二个参数为空字符串，这样将会从环境变量里读取此LC变量的值。当第二个参数为空字符串时，setlocale函数根据以下规则顺序来从环境变量里获取相对应LC变量的值：

> 1. 如果存在非空的环境变量`LC_ALL`,则使用其值作为此LC变量的值
> 2. 如果与此LC变量同名的LC环境变量存在且其值不为空，则取此LC环境变量的值作为此LC变量的值
> 3. 如果存在非空的`LANG`环境变量，则取`LANG`环境变量的值作为此LC变量的值

可能以上的说法比较拗口，那再用直白的说法描述一下：

> + 如果环境变量`LC_ALL`存在且不为空，则`LC_ALL`环境变量的值将覆盖所有`LC_*`变量的值。
> + 如果环境变量`LC_ALL`不存在或者其值为空，则`LC_*`变量的值为其自身的值，如果其值为空，则取`LANG`环境变量的值为此LC变量的值。
> + 一般不会有程序直接使用`LANG`这个环境变量，`LANG`环境变量一般作为所有`LC_*`变量的fallback支持。

还剩下一个`LANGUAGE`没有提到，我翻译引用[一段话](https://lists.debian.org/debian-i18n/2005/11/msg00019.html)来解释一下这个环境变量：

> 在使用gettext函数族时，如果设置了`LANGUAGE`环境变量，它将比`LANG`,`LC_MESSAGES`,甚至`LC_ALL`等变量拥有更高的优先级，但是字符的**encoding**信息依然从`LC_CTYPE`获取。  
> 举例来说，假设你的locale设置为**en_US.ISO-8859-1**但是你设置了如下的环境变量:   
> LANG=en\_US  
> LANGUAGE=zh\_CN:en\_US  
> 这样，你将不能看到正常显示为中文，因为在LC_CTYPE中，**encoding**被设置为**ISO-8859-1**,但是你却希望显示中文。

在使用GNU gettext函数族进行翻译时，`LANGUAGE`变量将具有最高的优先级别，除此之外的任何场合，`LANGUAGE`环境变量将不会被使用。`LANGUAGE`里可以指定多种语言，只需要将其用`:`分开即可，例如上面的`LANGUAGE=zh_CN:en_US`。gettxt函数在获取翻译时，会依照`LANGAGE`里设定的语言优先顺序来获取翻译内容。
