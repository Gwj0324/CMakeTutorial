## 第二步：添加库

这一节，我们将学习如何去创建一个库，并在我们的项目中使用这个库。我们也会学到怎么让库的使用变成可选择的。

### 练习1-指定目录

在很多时候，我们会有多个文件需要编译，并且我们会将项目的 .h头文件 和 .cpp源文件 放在不同的文件夹中，这时候就需要添加头文件的目录。

在cmake中有两个预定义的变量：`<projectname>_BINARY_DIR` 和`<projectname>_SOURCE_DIR` ，即一旦使用了project指明了一个项目名称，则同时隐式定义了这两个预定义的变量。在内部编译情况下，这两个变量的含义相同，而在外部编译下，两者指代的内容会有所不同。`<projectname>_BINARY_DIR`指的是外部构建指定的目录。通用的变量`PROJECT_SOURCE_DIR`变量是指此项目源代码所在的路径，`PROJECT_BINARY_DIR`同理指的是此项目构建目录。

例子：见01-basic/B-hello-headers,这里将头文件放在include文件夹中，.cpp源文件放在了src文件夹下。
```cmake
cmake_minimum_required(VERSION 3.5)
project (hello_headers)
#添加源文件
set(SOURCES
    src/Hello.cpp
    src/main.cpp
)
add_executable(hello_headers ${SOURCES})
#添加头文件目录
target_include_directories(hello_headers
    PRIVATE 
        ${PROJECT_SOURCE_DIR}/include
)
```

### 练习2-构建一个静态库

接下来，我们将构建一个静态库，并使用它。我们将使用add_library()命令。

```cmake
add_library(<name> [STATIC | SHARED]            
            [<source>...])
```

add_library根据源码来生成一个库供他人使用。`name`是个逻辑名称，在项目中必须唯一。完整的库名依赖于具体构建方式（可能为lib`name`.a or `name`.lib）。

STATIC指静态库，SHARED指动态库。

例子：见`01-basic/C-static-library`

```cmake
cmake_minimum_required(VERSION 3.5)
project(hello_library)
#创建一个静态库，静态库的名字叫做hello_library,静态库由src/Hello.cpp编译而成
add_library(hello_library STATIC 
    src/Hello.cpp
)
#静态库hello_library及引用该动态库的程序所需要的头文件路径
target_include_directories(hello_library
    PUBLIC 
        ${PROJECT_SOURCE_DIR}/include
)
#创建一个可执行程序hello_binary
add_executable(hello_binary 
    src/main.cpp
)
#将静态库hello_binary链接到hello_binary程序
target_link_libraries( hello_binary
    PRIVATE 
        hello_library
)
```
这里我在Windows系统使用`mingw`套件编译后，`build`目录下会生成`libhello_library.a`静态库文件。其它平台或套件可能静态库文件名称有所差异。

### 练习3-构建一个动态库

添加动态库和添加静态库类似。

例子：见`01-basic/D-shared-library`

```cmake
cmake_minimum_required(VERSION 3.5)
project(hello_library)
#生成一个动态库
add_library(hello_library SHARED 
    src/Hello.cpp
)
#指定动态库及引用该动态库的程序，需要包含的头文件
target_include_directories(hello_library
    PUBLIC 
        ${PROJECT_SOURCE_DIR}/include
)

add_executable(hello_binary
    src/main.cpp
)
#将hello_library动态库链接到hello_binary程序
target_link_libraries( hello_binary
    PRIVATE 
        hello_library
)

```

编译后，build目录下会生成libhello_library.dll动态库文件。

最后，在这里解释一下CMake中`PRIVATE`,`PUBLIC`,`INTERFACE`的含义和用法。

在 cmake 中，`PRIVATE`、`PUBLIC`、`INTERFACE` 是用来指定目标之间的依赖关系的关键字，它们可以用在 `target_link_libraries`、`target_include_directories`、`target_compile_definitions` 等指令中，用来控制依赖项的传递和范围。以下是一些示例和解释：

**target_link_libraries示例**

`PRIVATE`：表示依赖项只对当前目标有效，不会传递给其他依赖于当前目标的目标。例如，如果你有一个库 `libA`，它依赖于另一个库 `libB`，并且只在 `libA` 的源文件中使用了 `libB` 的功能，那么你可以这样写：

```cmake
target_link_libraries(libA PRIVATE libB)
```

这样，`libA` 就可以链接到 `libB`，但是如果有一个可执行文件 `app`，它依赖于 `libA`，那么它不会自动链接到 `libB`，也不能使用 `libB` 的功能。如果你想让 `app` 也链接到 `libB`，你需要显式地写出：

```cmake
target_link_libraries(app libA libB)
```

`INTERFACE`：表示依赖项只对其他依赖于当前目标的目标有效，不会影响当前目标本身。例如，如果你有一个库 `libA`，它依赖于另一个库 `libB`，并且只在 `libA` 的头文件中使用了 `libB` 的功能，那么你可以这样写：

```cmake
target_link_libraries(libA INTERFACE libB)
```

这样，`libA` 就不会链接到 `libB`，但是如果有一个可执行文件 `app`，它依赖于 `libA`，那么它会自动链接到 `libB`，并且可以使用 `libB` 的功能。这种情况通常发生在当前目标只是作为一个接口或者包装器，将另一个目标的功能暴露给其他目标。

`PUBLIC`：表示依赖项既对当前目标有效，也会传递给其他依赖于当前目标的目标。例如，如果你有一个库 `libA`，它依赖于另一个库 `libB`，并且在 `libA` 的源文件和头文件中都使用了 `libB` 的功能，那么你可以这样写：

```cmake
target_link_libraries(libA PUBLIC libB)
```

这样，`libA` 就会链接到 `libB`，并且如果有一个可执行文件 `app`，它依赖于 `libA`，那么它也会自动链接到 `libB`，并且可以使用 `libB` 的功能。这种情况通常发生在当前目标既使用了另一个目标的功能，也将其暴露给其他目标。

**target_include_directories示例**

用于指定目标包含的头文件路径。`PRIVATE`、`PUBLIC` 和 `INTERFACE`，分别表示不同的作用域和传递方式。

`PRIVATE`：表示头文件只能被目标本身使用，不能被其他依赖目标使用。例如，如果你有一个库 `libhello-world.so`，它只在自己的源文件中包含了 `hello.h`，而不在对外的头文件 `hello_world.h` 中包含，那么你可以写：

```cmake
target_include_directories(hello-world PRIVATE hello) 
```

这样，其他依赖 `libhello-world.so` 的目标（比如一个可执行文件）就不会自动包含 `hello.h`。

`PUBLIC`：表示头文件既能被目标本身使用，也能被其他依赖目标使用。例如，如果你有一个库 `libhello-world.so`，它在自己的源文件和对外的头文件中都包含了 `hello.h`，那么你可以写：

```cmake
target_include_directories(hello-world PUBLIC hello) 
```

这样，其他依赖 `libhello-world.so` 的目标就会自动包含 `hello.h`。

`INTERFACE`：表示头文件不能被目标本身使用，只能被其他依赖目标使用。例如，如果你有一个库 `libhello-world.so`，它只在对外的头文件中包含了 `hello.h`，而不在自己的源文件中包含，那么你可以写：

```cmake
target_include_directories(hello-world INTERFACE hello) 
```

这样，其他依赖 `libhello-world.so` 的目标就会自动包含 `hello.h`，但是 `libhello-world.so` 本身不会。

一些示例：

如果你有一个静态库或者动态库，它需要暴露一些头文件给其他目标使用，那么你可以使用 `PUBLIC` 或者 `INTERFACE` 关键字。

如果你有一个可执行文件或者测试程序，它需要包含一些内部的头文件，而不需要暴露给其他目标使用，那么你可以使用 `PRIVATE` 关键字。

如果你有一个接口库（INTERFACE library），它只定义了一些接口或者抽象类，并没有实现任何功能，那么你可以使用 `INTERFACE` 关键字。