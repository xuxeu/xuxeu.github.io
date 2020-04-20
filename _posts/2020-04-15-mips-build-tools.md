---
layout: post
title: 基于龙芯64位的嵌入式交叉编译工具的构建
date: 2020-04-12
Author: xuxeu
categories: 
tags: [编程杂谈]
comments: true
typora-root-url: ..
---

[TOC]
##摘要
以构建mips嵌入式交叉编译工具mipself-*为例，详细说明了如何在Linux系统下使用newlib库创建灵活配置的嵌入式交叉编译工具的通用方法。该方法具有速度快方便移植的特点，可以用于创建arm，ppc以及mips64交叉编译工具！
##引言
随着嵌入式电子产品的大量应用以及Linux操作系统的广泛使用，基于Linux操作系统的嵌入式开发环境已经越来越多的企业、学校以及科研机构中使用。交叉编译器能够在一个平台上生成另一个平台上的可执行代码，它是嵌入式开发环境中必不可少的工具。
目前，大多数嵌入式交叉编译工具都基于glibc和Linux内核构建的ARCH-linux-*(ARCH可以是ppc，mips或mips64)形式的交叉编译器。这种交叉编译器主要针对运行Linux操作系统的机器，它依赖于指定的C语言库glibc，且需要有MMU的支持，使得这种编译器编译出来的目标文件占用较大空间。
本文构建的交叉编译工具链采用ARCH-elf-*。其特点是：

1. 灵活性强，ARCH-elf-*是一个独立的编译体系，不依赖于指定的C语言库glibc，可以使用newlib等其他C语言库，不需要操作系统的支持，可移植到很多CPU结构上。
2. 支持基于UCLinux的操作系统，且不需要MMU支持，编译出来的目标文件占空间相对要少。
3. 速度比ARCH-linux-*要快。
##编译前的准备
### cygwin环境的搭建

cygwin是一个在Windows平台下运行的Unix模拟环境，是cygnus solutions公司开发的自由软件。他们写了一个共享库，也就是cygwin1.dll，把win32api中没有的Unix风格的调用封装在里面。

也就是说，他们基于win32api写了一个Unix系统库的模拟层。这样，只要把这些工具的源代码和这个共享库连接在一起，就可以使用Unix主机上的交叉编译器来生成可以在Windows平台上运行的工具集。

以这些移植到Windows平台上的开发工具为基础，cygnus又逐步把其他的工具软件移植到Windows上来。我们也可以查看cygwin官网，上面写道：

	cygwin 是：
	提供在Windows上实现Linux类似功能的gnu和开源工具的庞大工具集。
	一个DLL(cygwin1.dll)提供POSIX API功能的东西。
我们可以在cygwin官网[https://www.cygwin.com/](https://www.cygwin.com/)下载它的安装程序 [setup-x86.exe](https://www.cygwin.com/setup-x86.exe)。

cygwin的安装教程很多，需要注意的一点是，当我们选择一个download set的时候，我们需要选择一个国内的镜像地址，否则下载将会很慢。我们可以选择：http://www.cygwin.cn。如果找不到这个地址，也可以使用阿里云镜像http://mirrors.aliyun.com/cygwin/选择完成后，点击“下一步”即可。

![Cygwin5](/images/2020-04-15-mips-build-tools/Cygwin5.png)

最后，我们选择需要下载安装的组件包，为了使我们安装的Cygwin能够编译程序，我们需要安装gcc编译器，默认情况下，gcc并不会被安装，我们需要选中它来安装。为了安装gcc，我们用鼠标点开组件列表中的“Devel”分支，在该分支下，有很多组件，我们必须的是：

- binutils 
- gcc 
- make 
- gdb

安装完成以后，打开cygwin，如果可以在里面执行命令make，gcc等，则说明cygwin环境已经搭建成功。



###源代码准备

宿主机是用来开发和调试嵌入式系统应用程序的计算机系统。宿主机硬件可以用PC机，配置linux操作系统。
Linux下的交叉编译环境主要包括以下几个重要部分：
1. 针对目标系统的二进制工具binutils
2. 针对目标系统的动态编译器gcc
3. 针对目标系统的标准C库
   因此，在生成目标机工具链的过程中，需要准备如下源代码：
4. 用于生成目标系统的二进制工具集binutils
5. 用于生成目标系统的编译器gcc源代码，mpfr，gmp，mpc
6. newlib源代码
7. make工具以及linux内核头文件
###准备build目录和安装目录
首先，将下载的源代码放于/home/me/gnuu下并解压源代码文件，解压后会生成binutils，gcc，newlib。然后建立相关目录。

![1](.\1.png)

##构建过程
###生成目标二进制工具
目标系统的二进制工具binutils是通过对源码binutils--2.32进行编译和安装而生成。生成的目标系统二进制工具，如ld、objdump等将安装到/home/me/gnuu中。

####修改binutils源码
Bfd目录下的config.bfd
![2](/images/2020-04-15-mips-build-tools/2.png)
Gas目录下的configure
![3](/images/2020-04-15-mips-build-tools/3.png)
Gas目录下的configure.ac
![4](/images/2020-04-15-mips-build-tools/4.png)
Ld目录下的configure.tgt
![5](/images/2020-04-15-mips-build-tools/5.png)
#### 什么是BFD

BFD是binary file descriptor library的简称，它的目标是希望通过一种统一的接口来处理不同的目标文件格式。

现在GCC（更具体地讲是GNU汇编器GAS，GNU Assembler）、链接器ld、调试器GDB及binutils的其他工具都通过BFD库来处理目标文件，而不是直接操作目标文件。这样做最大的好处是将编译器和链接器本身同具体的目标文件格式隔离开来。

一旦我们需要支持一种新的目标文件格式，只需要在BFD库里面添加一种格式就可以了，而不需要修改编译器和链接器。到目前为止，BFD库支持大约25种处理器平台，将近50种目标文件格式。

我们利用objcopy(操作)，objdump（读取、分析），readelf(读取)，就是全仰仗BFD的功能。

#### 什么是ABI

我们很熟悉API，如果我们想使用某些库或操作系统的功能，将使用API。API由数据类型/结构，常量，函数等组成，您可以在代码中使用它们来访问该外部组件的功能。比如POSIX接口，就有一套它遵循的API标准。当我们要使用一个POSIX接口，我们可以从其头文件知道其接口名，以及参数类型和返回值类型，这些都是确定好的，我们使用时只能遵循它，而不能改变它。

ABI非常相似，可以将其视为API的编译版本（或作为机器语言级别的API）。编写源代码时，可以通过API访问库。编译代码后，您的应用程序将通过ABI访问库中的二进制数据。ABI定义了编译的应用程序将用于访问外部库的结构和方法（就像API一样），仅在较低级别上。

有时，ABI的变化是不可避免的。发生这种情况时，任何使用该库的程序都将无法运行，除非重新编译它们以使用新版本的库。如果ABI发生变化但API没有变化，则新旧库版本有时称为“源兼容”。

ABI是一系列约定的集合，例如GNU/Linux约定函数调用的头六个整型参数放在寄存器RDI, RSI, RDX, RCX, R8和R9上；其他额外的参数推入栈，返回值保存在RAX中。可以说调用惯例(calling convention)就是ABI。因此，ABI是和具体CPU架构和OS相关的。

**举个例子：**

- o32与n64即纯粹的32位与64位模式，二者除指针与变量类型的长度差异外，n64还用寄存器来传递更多的参数，性能有所提高。
- n32则是32位数据结构和64位指令的结合体。MIPS N32 ABI在保留 MIPS N64 ABI 的几乎所有特性的情况下（主要是寄存器和堆栈中的函数参数传递约定），重点在于仅将long long与double类型编译为64位，其余指针与变量类型设定与o32相同（例如，指针和long int是32位的），因此更接近MIPS N64 ABI。

####在binbuild目录下执行

```cmake
sudo ../binutils-2.32/configure --prefix=/home/me/gnuu/bininstall --target=mips-loongson-elf  --disable-nls --disable-werror --enable-shared --enable-threads=posix --enable-interwork --enable-multilib --enable-languages=c,c++,fortran
make
make install
```
通过源码编译编译器，我们就会使用到**Configure脚本配置工具**，它是autoconf的工具的基本应用。

'configure'脚本有大量的命令行选项。对不同的软件包来说，这些选项可能会有变化，但是许多基本的选项是不会改变的。带上'--help'选项执行'configure'脚本可以看到可用的所有选项。尽管许多选项是很少用到的，但是当你为了特殊的需求而configure一个包时，知道他们的存在是很有益处的。

```
--build=BUILD

指定软件包安装的系统平台。如果没有指定，默认值将是'--host'选项的值。

--host=HOST

指定软件运行的系统平台。如果没有指定。将会运行`config.guess'来检测。

--target=GARGET

指定软件面向(target to)的系统平台。这主要在程序语言工具如编译器和汇编器上下文中起作用。如果没有指定，默认将使用'--host'选项的值。

--prefix=PEWFIX

'--prefix'是最常用的选项。制作出的'Makefile'会查看随此选项传递的参数，当一个包在安装时可以彻底的重新安置他的结构独立部分。
```



target选项用来指定目标板的类型。由于本次编译采用了newlib，且生成的编译器可以独立运行，不需要linux系统支持，因此，目标板的类型设置为mips-elf。如果需要构建其它类型的目标板，只需要用其它目标类型替换mips即可，如arm、ppc等。Prefix选项用来指定安装目录。完成安装后在/home/me/gnuu/bininstall/bin生成如下二进制工具：
![6](/images/2020-04-15-mips-build-tools/6.png)

###生成静态库编译器
静态库编译器也称自举编译器bootstrap compiler。因为这时并没有mips的C语言库可使用，所以只能先生成一个简单的mips-elf-gcc编译器，该编译器只支持C语言的编译。用该静态编译器对C语言库进行编译后，再重新对编译器进行编译，从而生成完整的交叉编译工具。
####编译依赖库
编译gcc期间可能会存在软件的依赖，因此需提前下载好gmp，mpfr，mpc库，下载方法非常简单。在/home/me/gnuu/gcc-7.4.0/contrib中，存在download_prerequisites这个脚本文件，执行它，会自动安装相关依赖库。

![依赖](/images/2020-04-15-mips-build-tools/依赖.bmp)

####在gccbuild目录下执行
```
sudo ../gcc-7.4.0/configure --target=mips-loongson-elf --enable-threads --disable-libmudflap --disable-libssp --disable-libstdcxx-pch --disable-hosted-libstdcxx --enable-version-specific-runtime-libs --disable-sjlj-exceptions --disable-symvers --enable-__cxa_atexit --disable-fixed-point --disable-decimal-float --enable-interwork --enable-multilib --with-gnu-as --with-gnu-ld  --with-newlib --enable-languages=c,c++ --enable-shared --disable-lto --disable-hosted-libstdcxx --prefix=/home/me/gnuu/gccinstall --with-gxx-include-dir=''\''/'\''include/c++/7.4' --disable-libgomp --disable-libitm --with-build-time-tools=/home/me/gnuu/bininstall/mips-loongson-elf/bin
make all-gcc                 //编译静态gcc
make install-gcc              //安装静态gcc
```
由于这里还没有对库文件进行编译，且对库文件编译时需要静态编译工具mips-elf-gcc；因此需要对gcc进行编译时可以不用with-header的选项。但是，为了下步进行动态编译时不在重新配置gcc，这里将动态配置时的选项增加进来。其中，with-header指定的头文件是newlib标准库中的头文件，而不再使用基于linux的C标准库文件。Enable-language选项用于指定生成的交叉编译工具对C和C++编程语言的支持。完成安装后在/home/me/gnuu/gccinstall/bin中：
![7](/images/2020-04-15-mips-build-tools/7.png)
###利用静态编译工具编译嵌入式库文件
本文主要采用newlib库取代glibc库，从而生成一个不依赖于linux操作系统的灵活、方便的编译器，对newlib的编译主要是使用静态编译器对newlib进行编译，从而生成支持交叉编译器的动态链接库：
```
sudo ../newlib-3.1.0/configure --prefix=/home/me/gnuu/newinstall --target=mips-loongson-elf  --disable-nls --disable-werror --enable-shared --enable-threads=posix --enable-interwork --enable-multilib --enable-languages=c,c++,fortran
make
make install
```
###生成交叉编译工具链
在生成静态gcc时，已经添加了所有用到的选项，如使用newlib中的头文件。生成对C和C++支持编译器等。因此，只需要对gcc直接进行编译即可，无需重新配置各编译选项。该步骤完成后，会在install目录下生成完整的嵌入式交叉编译工具。
##验证工作
生成好改交叉编译链以后，我们使用它来验证其正确性：
```
./mips-loongson-elf-gcc -EL -mabi=64 -c test.c -o test.o
./mips-loongson-elf-gcc -EL -mabi=64 -c sub.c -o sub.o
./mips-loongson-elf-ld -r -T link.OUT test.o sub.o -o test.out
./mips-loongson-elf-objdump -t -x -h test.out > test.txt
```
通过objdump反汇编来看，是符合要求的：
![8](/images/2020-04-15-mips-build-tools/8.png)

## 注意事项

### ar命令

因为，此次编译的工具链，支持的abi有o32、n32，以及64，因此在编译链接以及ar命令的时候一定要通过参数-mabi=xx，来指定其abi接口类型。

比如，当ar的时候没有指定abi的时候，报的错：

*error adding symbols: archive has no index; run ranlib to add one*

究其原因，其实是ar的时候，需要指定目标板具体的体系结构，--target=elf32-nlittlemips
否则，ar会以默认的target来打包程序，而链接的时候因为制定了abi为n32，则导致找不到静态库的符号索引表。

### 链接脚本

使用新的gcc时，需要修改链接脚本，在其中添加：
OUTPUT_FORMAT("elf32-nlittlemips")
OUTPUT_ARCH(mips)
ENTRY(__start)
用以指定输出的elf格式和体系结构，以及起始函数。

##结束语

通过ARCH-elf-*搭建跨平台mips开发工具链的实例，说明了如何使用newlib库构建灵活配置的嵌入式开发平台，从而使嵌入式开发人员可以直接在PC机上开发基于嵌入式结构的应用程序。
当我们需要开发基于VxWorks库的嵌入式开发平台，可以以至进行参考。无非就是target变了，binutils里面的configure等文件做好相应更改就可以了。

**出差必备:**

买火车票、高铁票、机票，订酒店都打9折的出行工具TRIP,[点击注册](https://h5.itrip.world/#/register/6tpd1Z)
