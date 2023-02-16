# cmake 教程

### CMake一个HelloWorld的语法介绍

```markdown
project(HELLO)
set(SRC_LIST main.cpp)
message(status "this is binary dir " ${HELLO_BINARY_DIR})
message(status "this is source dir" ${HELLO_SOURCE_DIR})
add_executable(hello ${SRC_LIST})
```

### PROJECT关键字

可以用来指定工程的==名字和支持的语言==，默认支持所有语言

PROJECT(HELLO)

PROJECT(HELLO CXX C)

该指定隐式定义了两个CMAKE的变量

<projectname>\_BINARY\_DIR和<projectname>\_SOURCE\_DIR

MESSAGE关键字就可以直接使用这两个变量，当前都指向当前的工作目录，后面会讲外部编译

问题：如果改了工程名，这两个变量名也会改变

解决：又定义两个预定义变量：PROJECT_BINARY_DIR和PROJECT_SOURCE_DIR，这两个变量内容和原来是一致的



### SET关键字

用来显示的指定变量的

set(SRC_LIST main.cpp) SRC_LIST变量就包含了main.cpp

也可以SET(SRC_LIST main.cpp t1.cpp t2.cpp)

### MESSAGE关键字

向终端输出用户自定义的信息

主要包含三种信息：

- SEND_ERROR，产生错误，生成过程被跳过
- SATUS，输出前缀为—的信息
- FATAL_ERROR，立即终止所有CMake的过程

### ADD_EXECUTABLE关键字

生成可执行文件

ADD_EXECUTABLE(hello ${SRC_LIST})生成可执行文件hello，源文件读取变量SRC_LIST中的内容

也可直接写ADD_EXECUTABLE(hello main.cpp)

> 注意：工程名和生成的可执行文件是没有任何关系的

### 语法的基本原则

- 变量使用${}取值，但是在IF控制语句是直接使用变量名
- 指令的参数都是用圆括号括起来的，参数之间使用==空格或者逗号==分隔开
- 指令是大小写无关的，参数和变量是大小写相关的

### 语法注意

- SET(SRC_LIST main.cpp)可以写成SET(SRC_LIST "main.cpp")，如果源文件名中含有空格，则必须加双引号
- ADD_EXECUTABLE(hello main)后缀可以不写，他会自动去找cpp和c，但是不建议这样写

### 内部构建和外部构建

- 内部构建产生的临时文件多
- 外部构建就是把生成的文件放在build文件夹下，不会对源文件产生任何的影响

### 让Hello World看起来更像一个工程

- 为工程添加一个子目录src，用来放置工程源代码
- 添加一个子目录doc，用来放置这个工程文档的hello.txt
- 在工程目录添加文本文件COPYRIGHT,README
- 在工程目录添加一个 `runhello.sh`脚本，用来调用hello二进制
- 将构建好后的目标文件放入构建目录的bin子目录
- 将doc目录的内容以及COPYRIGHT/README安装到/usr/share/doc/cmake



### 将目标文件放入构建目录的bin子目录

![capture_20221022153432562](D:\Huawei Share\Screenshot\capture_20221022153432562.bmp)

外层CMakeLists.txt

```cmake
project(hello)
add_subdirectory(src bin)
```

src下的CMakeLists.txt

```cmake
add_exectable(hello main.cpp)
```

### ADD_SUBDIRECTORY指令

ADD_SUBDORECTORY{source_dir [binary] [EXCLUDE_FROM_ALL]}

- 这个指令用于向当前工程添加存放源文件的子目录，并可以指定中间二进制和目标二进制存放的位置
- EXCLUDE_FROM_ALL函数是将写的目录从编译中排除，如程序中的example
- add_subdirectory(src bin)

​		将src子目录加入工程并指定编译输出路径为bin目录

如果不进行bin目录的指定，那么编译结果（包括中间结果）都将放在build/src

- 要写在最外层

## 安装

- 一种是从代码编译后直接make install安装
- 一种是打包时的指定目录安装
  - 简单的可以这样指定目录：make install DESTDIR==/tmp/test
  - 稍微复杂的可以这样指定目录：./configure -prefix=/usr

### 如何安装HelloWorld

使用cmake一个新指令：INSTALL

​	INSTALL的安装可以包括：二进制、动态库、静态库以及文件、目录、脚本等

使用CMAKE一个新的变量：CMAKE_INSTALL_PREFIX

![capture_20221022155756666](D:\Huawei Share\Screenshot\capture_20221022155756666.bmp)

#### 安装文件COPYRIGHT和README

INSTALL(FILES COPYRIGHT README DESTINATION share/doc/cmake)

FILES:文件

DESTINATION:

1. 写绝对路径
2. 可以写相对路径，相对路径其实是：${CMAKE_INSTALL_PREFIX}/<DESTINATION定义的路径>

​		CMAKE_INSTALL_PREFIX 默认是在/usr/local/

​		cmake -DCMAKE_INSTALL_PREFIX=/usr 在cmake的时候指定的DCMAKE_INSTALL_PREFIX变量的路径

#### 安装脚本

PROGRAMS:非可执行程序

INSTALL(PROGRAMS runhello.sh DESTINATION bin)

#### 安装doc中的hello.txt

一、是通过在Doc目录中建立CMakeLists.txt，通过install下的file

二、是直接在工程目录通过

INSTALL(DIRECTORY doc==/== DESTINATION share/doc/cmake)

DESTINATION后面连接的是所在Source目录的相对路径

> 注意：doc目录不以/结尾，表示安装这个目录
>
> 以/结尾，表示安装这个目录下的内容

### 静态库和动态库的构建

任务：

1. 建立一个静态库和动态库，提供HelloFunc函数供其他程序编程使用，HelloFunc向终端输出Hello World字符
2. 安装头文件与共享库

静态库与动态库的区别

- 静态库的扩展名一般为“.a”或".lib"；动态库的扩展名一般为".so" or ".dll"
- 静态库在编译时会直接整合到目标程序中，编译成功的可执行文件可独立运行
- 动态库在编译的时候不会放到连接的目标程序中，即可执行文件无法独立运行

#### ADD_LIBRARY

ADD_LIBRARY(hello SHARED ${LIBHELLO_SRC})

- HELLO:就是库名，最后生成的时lib会加到前面，即为libhello.so
- SHARED,动态库    STATIC，静态库
- ${LIBHELLO_SRC}源文件

> 若要同时构建动态库和静态库，则必须把两个库的名字区别开，否则只会构建出动态库

同时构建动态库和静态库

```cmake
SET(LIBHELLO_SRC hello.cpp)
ADD_LIBRARY(hello_static STATIC ${LIBHELLO_SRC})
#对hello_static重命名为hello
SET_TARGET_PROPERTIES(hello_static PROPERTIES OUTPUT_NAME "hello")
#cmake在构建一个新的target时，会尝试清理掉其他这个名字的库，比如，在构建libhello.so时，就会清理掉libhello.a
SET_TARGET_PROPERTIES(hello_static PROPERTIES CLEAN_DIRECT_OUTPUT 1)

ADD_LIBRARY(hello SHARED ${LIBHELLO_SRC})

SET_TARGET_PROPERTIES(hello PROPERTIES OUTPUT_NAME "hello")
SET_TARGET_PROPERTIES(hello PROPERTIES CLEAN_DIRECT_OUTPUT 1)

```

#### 安装共享库和头文件

本例中我们将hello的共享库安装到<prefix>/lib目录

将hello.h安装到<prefix>/include/hello目录

```cmake
INSTALL(FIELS hello.h DESTINATION include/hello)
#二进制，静态库，动态库都用TARGET
#ARCHIVE 特指静态库，LIBRARY特指动态库，RUNTIME，特指可执行目标二进制
INSTALL(TARGET hello hello_static LIBRARY DESTINATION lib ARCHIVE DESTINATION lib)
```

#### 使用共享库和头文件

解决：make后头文件找不到的问题

PS：include <hello/hello.h>

关键字：INCLUDE_DIRECTORIES 这条指令可以用来向工程添加多个特定的头文件搜索路径，路径之间用空格分隔

在CMakeLists.txt中加入头文件搜索路径

INCLUDE_DIRECTORIES(/usr/include/hello)

解决：找到引用函数问题

关键字：LINK_DIRECTORIES(/usr/include/hello)添加非标准的共享库搜索路径

关键字：TARGET_LINK_LIBRARIES(可执行文件名  共享库)

> 在CMakeLists.txt中插入连接共享库，主要要插在executable后面



