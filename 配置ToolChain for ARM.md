#配置工具链ToolChain for ARM

### ToolChain简介
ToolChain包括许多部件: 主要之一是GCC,  GCC是有Binutils工具支持, Binutils是二进制代码维护工具.

### 定制ToolChain
大致步骤:
- 决定目标的名称
- 决定目标存放位置
- 编译,安装Binutils
- 编译,安装GLIBC
- 让交叉编译工具支持更多编译语言
- 测试定制的交叉编译工具

#### 决定目标的名称
对于ARM开发来说, 需要一个在本机编译, 但编译生成ARM运行代码的一套交叉编译工具集.
选择一个与本机工具集相异的特定含义的目标名称是有必要的.

选择什么名称对定制ARM交叉编译工具集不影响.
常用的名字:
插入支持格式
arm-linux, arm-aout, arm-coff, arm-elf, arm-thumb
或者插入版本信息
armv2: ARMV2核, 支持26bit模式.
armv3l, armv3b,  小字节(l)或者大字节(b)模式
armv5l, armv5b.

#### 决定目标的存放位置
定制ARM交叉编译工具集, 不能覆盖本机的编译工具, 所以需要将ARM交叉编译工具集存放在另外位置. 可以使用 /usr/local/arm, (本机编译工具集通常在/usr, /usr/local)

默认ARM交叉编译工具集名称 arm-pc-linux,
本地PC编译工具名称通常是: i686-pc-linux-gnu.

ARM核心文件位置: ~/armlinux, 这里仅是联接, 指向ARM核心源代码
本机核心代码存放位置通常在/usr/src/linux.

### 编译, 安装Binutils
Binutils主要是二进制代码的处理维护工具.

#### Binutils工具部件简介
add2line: 将地址转换成文件名或行号对, 便于调试
ar: 从体系文件中创建, 修改, 扩展程序代码
as: 生成汇编程序代码
c++filt: 建立低级语言和用户级语言的名称符合联接, 并保持他们的相互关系
gasp: 汇编宏处理器
ld: 目标代码联接, 联接个目标代码块, 它是生成可执行代码的最终步骤
nm: 从目标代码文件中枚举所有调试符号名
objcopy: 使用GNU BSD库, 把目标代码从一文件格式拷贝成另一种格式.
objdump: 显示目标文件信息.
readelf: 显示elf文件信息.
ranlib: 生成索引以加快对归档文件的访问.
size: 列出目标模块或文件的代码尺寸.
strings: 打印可打印的目标代码字符.
strip: 放弃所有符合联接.

#### Binutils的配置和编译安装
```
tar -jxvf binutils-2.16.1.tar.bz2 -C ~
cd ~/binutils-2.16.1
make clean
make distclean

./configure --target=arm-pc-linux --prefix=/usr/local/arm

make
make install
```
