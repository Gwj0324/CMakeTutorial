## 第三步：安装

这里，将介绍如何将我们的代码打包成一个可执行程序。使得它能运行在其它电脑或设备上。我们将使用`install()`命令。

```cmake
install(TARGETS target
        [[ARCHIVE|LIBRARY|RUNTIME|OBJECTS|FRAMEWORK|BUNDLE|PRIVATE_HEADER|PUBLIC_HEADER|RESOURCE]
            [DESTINATION <dir>])
            
 install(FILES files...
         [[DESTINATION <dir>])
          
  install(DIRECTORY dirs...
          [[DESTINATION <dir>])
```

TARGETS 参数指定要安装的目标名称。 ARCHIVE、LIBRARY、RUNTIME、FRAMEWORK、BUNDLE 等指定了将要安装的不同类型的目标。在安装时，DESTINATION 参数指定安装路径。对于库文件的安装，使用 LIBRARY 来指定库文件的类型.

FILES 参数指定要安装的文件。DIRECTORY指定要安装的目录，如果不加过滤，该目录下所有文件将被拷贝到安装路径下。

例如：见`01-basic/E-installing`

```cmake
cmake_minimum_required(VERSION 3.5)
project(cmake_examples_install)

add_library(cmake_examples_inst SHARED
    src/Hello.cpp
)

target_include_directories(cmake_examples_inst
    PUBLIC 
        ${PROJECT_SOURCE_DIR}/include
)


add_executable(cmake_examples_inst_bin
    src/main.cpp
)
target_link_libraries( cmake_examples_inst_bin
    PRIVATE 
        cmake_examples_inst
)

# 指定可执行文件 安装
install (TARGETS cmake_examples_inst_bin
    DESTINATION bin)

# 指定库文件 安装
install (TARGETS cmake_examples_inst
    LIBRARY DESTINATION lib)

# 指定头文件目录 安装
install(DIRECTORY ${PROJECT_SOURCE_DIR}/include/ 
    DESTINATION include)

# 指定配置文件 安装
install (FILES cmake-examples.conf
    DESTINATION etc)
```

我们将此项目构建好后。切换到外部构建目录，运行以下命令：

```cmake
#这里从当前构建目录提取安装信息，并将安装后程序信息存储在install目录下
cmake --install . --prefix "../install"
```

这里可以指定从哪个构建目录下提取安装信息，当然也可以指定安装后程序信息存储目录。


5.add_subdirectory
add_subdirectory(source_dir [binary_dir] [EXCLUDE_FROM_ALL])
将子目录添加到构建系统中。source_dir指定一个目录，其中存放CMakeLists.txt文件和代码文件。binary_dir指定的目录存放输出文件，如果没有指定则使用source_dir。

9.include_directories
include_directories([AFTER|BEFORE] [SYSTEM] dir1 [dir2 ...])
将指定目录添加到编译器的头文件搜索路径之下，指定目录被解释为当前源码路径的相对路径。[AFTER|BEFORE]定义了追加指定目录的方式在头还是尾。[SYSTEM]告诉编译器在一些平台上指定目录被当作系统头文件目录。

10.include
include(<file|module> [OPTIONAL] [RESULT_VARIABLE <VAR>]
                      [NO_POLICY_SCOPE])
从指定的文件加载、运行CMake代码。如果指定文件，则直接处理。如果指定module，则寻找module.cmake文件，首先在${CMAKE_MODULE_PATH}中寻找，然后在CMake的module目录中查找。



同时cmake还预定义了PROJECT_BINARY_DIR和PROJECT_SOURCE_DIR变量，其中PROJECT_BINARY_DIR就等同于<projectname>_BINARY_DIR,PROJECT_SOURCE_DIR等同于<projectname>_SOURCE_DIR。因此在实际应用中，强烈推荐使用PROJECT_BINARY_DIR和PROJECT_SOURCE_DIR变量，这样即使项目名称发生变化也不会影响CMakeLists.txt文件。

ADD_SUBDIRECTORY命令
语法： ADD_SUBDIRECTORY(source_dir [binary_dir] [EXCLUDE_FROM_ALL])
该命令告诉CMake去子目录中查看可用的CMakeLists.txt文件
指令用于向当前工程添加存放源文件的子目录,并可以指定中间二进制和目标二进制存放的位置。 EXCLUDE_FROM_ALL 参数的含义是将这个目录从编译过程中排除。比如,工程的 example,可能就需要工程构建完成后,再进入 example 目录单独进行构建。

ADD_EXECUTABLE命令
告诉工程生成一个可执行文件。该命令定义了工程最终生成的可执行文件的文件名以及参与编译的头文件和cpp文件。
如果想指定生成的可执行文件的存放路径，可以设置cmake中预定义变量EXECUTABLE_OUTPUT_PATH 的值。例如，将生成的可执行文件放置在cmake编译路径的bin文件夹下可用：SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)

ADD_LIBRARY命令
语法：ADD_LIBRARY(libname [SHARED|STATIC])
告诉工程生成一个库文件

SET命令——用于设置变量，相当于为变量取别名
SET(CMAKE_BUILE_TYPE DEBUG) 设置编译类型debug 或者release。 debug 版会生成相关调试信息，可以使用GDB 进行调试；release不会生成调试信息。当无法进行调试时查看此处是否设置为debug.

EXECUTABLE_OUTPUT_PATH和LIBRARY_OUTPUT_PATH变量
我们可以通过 SET 指令重新定义 EXECUTABLE_OUTPUT_PATH 和 LIBRARY_OUTPUT_PATH 变量来指定最终的目标二进制的位置(指最终生成的CRNode可执行文件或者最终的共享库，而不包含编译生成的中间文件)。
命令如下：
SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
SET(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)
需要注意的是，在哪里 ADD_EXECUTABLE 或 ADD_LIBRARY,如果需要改变目标存放路径,就在哪里加入上述的定义。

变量名	含义
PROJECT_NAME	project命令中写的项目名
CMAKE_VERSION	当前使用CMake的版本
CMAKE_SOURCE_DIR	工程顶层目录，即入口CMakeLists文件所在路径
PROJECT_SOURCE_DIR	同CMAKE_SOURCE_DIR
CMAKE_BINARY_DIR	工程编译发生的目录，即执行cmake命令进行项目配置的目录，一般为build
PROJECT_BINARY_DIR	同CMAKE_BINARY_DIR
CMAKE_CURRENT_SOURCE_DIR	当前处理的CMakeLists.txt所在的路径
CMAKE_CURRRENT_BINARY_DIR	当前处理的CMakeLists.txt中生成目标文件所在编译目录
CMAKE_CURRENT_LIST_FILE	输出调用这个变量的CMakeLists.txt文件的完整路径
CMAKE_CURRENT_LIST_DIR	当前处理的CMakeLists.txt文件所在目录的路径
CMAKE_INSTALL_PREFIX	指定make install命令执行时包安装路径
CMAKE_MODULE_PATH	find_package命令搜索包路径之一，默认为空
编译配置相关变量：

变量名	含义
CMAKE_BUILD_TYPE	编译选项，Release或者Debug，如set(CMAKE_BUILD_TYPE "Release")
CMAKE_CXX_FLAGS	编译标志，设置C++11编译，set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
CMAKE_CXX_STANDARD	也可以设置C++11编译，set(CMAKE_CXX_STANDARD 11)
