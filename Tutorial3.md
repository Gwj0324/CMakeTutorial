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