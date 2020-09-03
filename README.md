QuaZip is the C++ wrapper for Gilles Vollant's ZIP/UNZIP package
(AKA Minizip) using Trolltech's Qt library.

If you need to write files to a ZIP archive or read files from one
using QIODevice API, QuaZip is exactly the kind of tool you need.

See [the documentation](https://stachenov.github.io/quazip/) for details.

Want to report a bug or ask for a feature? Open an [issue](https://github.com/stachenov/quazip/issues).

Want to fix a bug or implement a new feature? See [CONTRIBUTING.md](CONTRIBUTING.md).

Copyright notice:

Copyright (C) 2005-2018 Sergey A. Tachenov

Distributed under LGPL, full details in the COPYING file.

Original ZIP package is copyrighted by Gilles Vollant, see
quazip/(un)zip.h files for details, but basically it's the zlib license.

https://www.cnblogs.com/charleechan/p/12144950.html

Qt使用quazip库
参考链接： https://blog.csdn.net/zhangxuechao_/article/details/83014473

一般地，源文件被预处理>>编译>>汇编>>链接才能生成可执行程序。

编译汇编的基本流程
预处理: 将源文件(.c文件)中的#define和#include等宏定义和包含的头文件等替换进去，成为新的源文件(.i文件)；每个.c文件对应一个.i文件。
编译: 将(.i文件)编译成为汇编代码(.asm文件,.s文件),每个.i文件对应一个.s文件.
汇编: 将汇编语言的文件(.s,.asm)翻译成为二进制的文件(内容是一堆机器码, .obj/.o文件)，每个.s文件对应一个.o文件。
连接: 将二进制文件(.o文件)连接在一起，形成一个大的应用程序文件(.o/.exe/.elf等)。
静态编译与动态编译
静态编译：编译器在把源文件(.cpp文件)编译可执行文件(.exe文件)时，主程序中调用的函数接口通过查询包含的头文件(.h)，把源文件中要调用的函数代码，翻译为库文件(.lib)，然后链接到可执行文件中去，使可执行文件在运行时不需要依赖于动态链接库(.dll文件)。

动态编译：编译器在把源文件(.cpp文件)编译可执行文件(.exe文件)时，主程序中调用的函数查询包含的头文件(.h)，把源文件中要调的函数代码，翻译为动态链接库文件(.dll)，然后链接到可执行文件(.exe)中，运行时，可执行文件查询导出库(.lib)文件，在动态链接库(.dll)中查找函数命令。其优点一方面是缩小了执行文件本身的体积，另一方面是加快了编译速度，节省了系统资源。缺点一是哪怕是简单的程序，只用了链接库中一两条命令，也要附带相对庞大的链接库；二是若其他计算机上没有安装对应的运行库，动态编译的可执行文件就不能运行。

综上可知, 不管如何,你要使用一个源码生成库，就必须有以下几个文件：

头文件(.h): 用来给应用程序代码中(主调方)看函数接口定义的;
库文件(.lib/.a): 用来给可执行文件查询.lib的，里面存放着相应函数的地址，或函数的内容。
库文件(.dll/.so): 二进制文件，用来给可执行文件调用的相应的服务函数，它内部实现了被调方的功能。
Qt 使用quazip库进行压缩和解压
1 编译生成windows zlib库
下载zlib库：http://www.zlib.net/;
在开始菜单中输入mingling，点击VS2015 x86本机工具命令提示符,切换到目录zlib-1.2.11.tar > zlib-1.2.11 > contrib > masmx86下，运行bld_ml32.bat;
最终生成的inffas32.obj和match686.obj是我们需要的.
回到zlib-1.2.11根目录,执行以下命令:nmake -f win32/Makefile.msc LOC="-DASMV -DASMINF" OBJA="contrib/masmx86/inffas32.obj contrib/masmx86/match686.obj".
生成了zlib1.dll动态库,这是我们后面要用的。
编译生成quazip工程
去https://github.com/stachenov/quazip下载quazip文件夹，qztest可以不用下载。
在quazip目录下(该目录还同时有debian文件夹)下新建文件夹include和文件夹lib。
将zlib中的头文件zlib.h和zconf.h放到include中, 把zlib1.dll放到lib中。
修改quazip.pro工程文件, 添加头文件路径和动态库路径
INCLUDEPATH += $$PWD/include
LIBS += -L$$PWD/lib -lzlib1
编译,会自动生成libquazipd.dll(Debug模式下)和libquazip.dll(Release模式下), 以及libquazipd.a或libquazip.a。
在主调工程中使用
在工程根目录(该目录下还包括了.pro文件)下新建文件夹include和文件夹lib。

将quazip中所有头文件，如ioapi.h,quazip.h,unzip.h,JlCompress.h等放在include中，将quazip.dll和quazipd.dll和zlib1.dll放在lib文件夹中。

在工程文件.pro中，添加头文件和库文件路径。

INCLUDEPATH += $$PWD/include
CONFIG(debug, debug|release) {
    LIBS += -L$$PWD/lib -lquazipd
} else {
    LIBS += -L$$PWD/lib -lquazip
}
在主调程序中书写以下代码以使用该库:

#include "mainwindow.h"
#include <QApplication>
#include <JlCompress.h>

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    MainWindow w;
    w.show();

    JlCompress::compressDir("D:/testzip/a.zip", "D:/testzipdir1");
    JlCompress::extractDir("D:/testzip/a.zip", "D:/testzipdir2");
    
    return a.exec();
}
