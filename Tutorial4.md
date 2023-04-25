## 第四步：子项目

当我们做一个较大规模的项目时，这个项目可能会包含几个子项目。

### 命令

ADD_SUBDIRECTORY命令
语法： ADD_SUBDIRECTORY(source_dir [binary_dir] [EXCLUDE_FROM_ALL])
该命令告诉CMake去子目录中查看可用的CMakeLists.txt文件
指令用于向当前工程添加存放源文件的子目录,并可以指定中间二进制和目标二进制存放的位置。 EXCLUDE_FROM_ALL 参数的含义是将这个目录从编译过程中排除。比如,工程的 example,可能就需要工程构建完成后,再进入 example 目录单独进行构建。


### 例子

例如：见`02-sub-projects/A-basic`

下面的项目包含了三个子项目，一个生成可执行文件，其它两个生成库文件。

```
$ tree
.
├── CMakeLists.txt
├── subbinary
│   ├── CMakeLists.txt
│   └── main.cpp
├── sublibrary1
│   ├── CMakeLists.txt
│   ├── include
│   │   └── sublib1
│   │       └── sublib1.h
│   └── src
│       └── sublib1.cpp
└── sublibrary2
    ├── CMakeLists.txt
    └── include
        └── sublib2
            └── sublib2.h
```

最顶层的目录下的`CMakeLists.txt`:

```cmake
cmake_minimum_required (VERSION 3.5)
project(subprojects)

# 添加子目录
add_subdirectory(sublibrary1)
add_subdirectory(sublibrary2)
add_subdirectory(subbinary)
```

sublibrary1目录下的`CMakeLists.txt`:

```cmake
project (sublibrary1)

# 添加一个库sublibrary1给它取别名为sub::lib1
add_library(${PROJECT_NAME} src/sublib1.cpp)
add_library(sub::lib1 ALIAS ${PROJECT_NAME})

target_include_directories( ${PROJECT_NAME}
    PUBLIC ${PROJECT_SOURCE_DIR}/include
)
```

sublibrary1目录下的`CMakeLists.txt`:

```cmake
project (sublibrary2)

add_library(${PROJECT_NAME} INTERFACE)
add_library(sub::lib2 ALIAS ${PROJECT_NAME})

target_include_directories(${PROJECT_NAME}
    INTERFACE
        ${PROJECT_SOURCE_DIR}/include
)
```

