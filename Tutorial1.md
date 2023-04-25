## 第一步：起点

### 练习1-构建一个项目

最基本的CMake项目是从单个源代码文件构建的可执行文件。对于像这种简单的项目，只需要一个包含三条命令的`CMakeLists.txt`文件就行了。

- `cmake_minimum_required()`
- `project()`
- `add_executable()`

> 注意：虽然CMake支持大写，小写以及大小写混合命令，但是我们更倾向于使用小写命令。参数区分大小写！

`cmake_minimum_required(VERSION 3.10)`：指定CMake的最小版本为3.10。任何项目最顶层的`CMakeLists.txt`必须先指定该项目所需使用的最低版本，使用`cmake_minimum_required()`命令。这样确保接下来的CMake命令使用兼容版本的CMake运行。

`project(Tutorial VERSION 1.0 LANGUAGES CXX)`：指定项目名称Tutorial及版本号1.0，编程语言C++。这是每个项目必须包含的命令，并且应该在`cmake_minimum_required()`命令之后立即被调用。

`add_executable(Tutorial tutorial.cxx)`:告诉CMake去用指定的源文件创建一个可执行程序。
```cmake
cmake_minimum_required(VERSION 3.10)

project(Tutorial)

add_executable(Tutorial tutorial.cpp)
```
**构建并运行：**

在`Step1`文件夹中有`tutorial.cxx`,`CMakeLists.txt`文件。我们将通过下面的步骤去生成一个计算平方根的可执行程序。

> 注意：请确保你的电脑上安装了合适的CMake版本。

通过命令行，我们切换到git仓库的根目录,创建一个构建目录：

```bash
mkdir Step1_build
```

然后，切换到构建目录并运行`cmake`去配置项目并且生成本地构建系统：

```bash
cd Step1_build
cmake ../Step1
```

接着调用构建系统去编译链接项目程序：

```bash
cmake --build .
```

最后，尝试运行新构建出来的`Tutorial`程序吧。

### 练习2-指定C++标准

CMake有一些特殊的变量，它们要么是在幕后创建的，要么在我们编写`CMakeLists.txt`时对CMake有意义。这些特殊变量很多都是由 `CMAKE_` 开头的，所以你在编写`CMakeLists.txt`创建自己项目的变量时要注意避开这种命名方式。接下来介绍两个用户可以设置属性的特殊变量：`CMAKE_CXX_STANDARD`和`CMAKE_CXX_STANDARD_REQUIRED`。他们被用来指定构建项目所需要的C++标准。

set() 是 CMake 中的一个命令，用于设置变量的值。该命令的基本语法如下：
```cmake
set(<variable> <value>)
```
其中 `variable` 是要设置的变量名称；`value` 是要为变量设置的新值。

例子：指定项目的C++标准要求为11。
我们需要在`add_executable()`之前添加声明。

```cmake
cmake_minimum_required(VERSION 3.10)
project(Tutorial)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED true)

add_executable(Tutorial tutorial.cpp)
```

### 练习3-添加一个版本号并配置头文件

有时候，你在`CMakeLists.txt`定义了一个变量，并且想要这个变量也能在代码中使用，这或许能方便地解决一些问题。例如，将`CMakeLists.txt`中定义的版本号用在代码中。

我们可以使用配置的头文件来实现这一目的。首先需要创建一个输入文件，这个文件里面有一个或多个要替换的变量。这些变量有着特殊的语法，如：`@VAR@`或`${VAR}`。然后，我们使用`configure_file()`命令去复制输入文件到一个给定的输出文件中，并用`CMakeLists.txt`文件中的`VAR`变量的当前值替换这些变量。

例子：声明版本号为1.0（主版本号为1，次版本号为0）

虽然我们可以直接在源代码中编辑版本号，但是使用这种方法有好处。因为它创建一个单一的真实数据源并且避免了复杂性。

```cmake
cmake_minimum_required(VERSION 3.10)
project(Tutorial VERSION 1.0)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED true)

configure_file(TutorialConfig.h.in TutorialConfig.h)

add_executable(Tutorial tutorial.cpp)

target_include_directories(Tutorial PUBLIC ${PROJECT_BINARY_DIR})
```

>注：CMake中的版本号组成部分：\<Major>.\<Minor>.\<Patch>.\<Tweek>

首先，使用`project()`命令设置项目名称和版本号。当此命令被调用，CMake会在幕后定义变量`<PROJECT-NAME>_VERSION_MAJOR`和变量`<PROJECT-NAME>_VERSION_MINOR`。

接着,我们在`TutorialConfig.h.in`文件中配置相关变量。当`CMakeLists.txt`中的`configure_file()`命令被调用，`TutorialConfig.h`文件中，变量`@<PROJECT-NAME>_VERSION_MAJOR@`和变量`@<PROJECT-NAME>_VERSION_MINOR@`将被指定的版本号数字替换掉。

`TutorialConfig.h.in`文件：

```c
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

替换后生成的`TutorialConfig.h`文件：

```c
#define Tutorial_VERSION_MAJOR 1
#define Tutorial_VERSION_MINOR 0
```

使用`configure_file()`命令去复制被具体的CMake变量替换后的输入文件。

configure_file() 命令的基本语法如下：

```cmake
configure_file(<input> <output>)
```
其中`input`是要复制和修改的源文件路径；`output`是目标文件的路径；

因为配置文件将被写入项目二进制目录中，我们必须将这个项目二进制目录添加到搜索包含文件的路径列表中去。`PROJECT_BINARY_DIR`变量代表的就是项目二进制目录，我们可以直接引用这个变量。

>注：在本次教程中，项目二进制目录和项目构建目录是相同的意思。

我们使用`target_include_directories()`来指定包含路径，可执行程序将从这个路径寻找头文件。

`target_include_directories()` 是 CMake 中的一个命令，用于为目标文件添加头文件搜索路径。该命令通常用于设置依赖库（如静态库、动态库）的头文件路径，以便在编译时能够正确地包含所需的头文件。

`target_include_directories()` 命令的基本语法如下：

```cmake
target_include_directories(<target> [SYSTEM] <INTERFACE|PUBLIC|PRIVATE> [items...])
```
其中 `target` 是要添加头文件搜索路径的目标文件名称；`SYSTEM` 选项用于指示头文件路径是系统级别的路径，可以再利用这个命令切换表现不同操作系统的头文件路径；`items...` 则是要添加的头文件路径列表。还可以指定访问级别，以控制哪些目标可以使用这些头文件路径。

然后，在你的主程序中引用`TutorialConfig.h`文件，就可以获取到版本号了。