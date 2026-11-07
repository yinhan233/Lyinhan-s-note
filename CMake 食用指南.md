# CMake 食用指南

说实话，这种类似食用指南的，我已经很久没写了，想回味下大一时懵懂的自己，故做此篇

## 初入 CMake

### 创建项目

1. 新建一个 `helloworld.cpp`。
2. 新建一个 `CMakeLists.txt`，注意**必须为此名**。

```cmake
cmake_minimum_required(VERSION 3.31)
project(Helloworld VERSION 0.1 LANGUAGES CXX)
add_executable(Helloworld helloworld.cpp)
```

其中：

| 命令 / 参数                                         | 说明                                                        |
| ----------------------------------------------- | --------------------------------------------------------- |
| `cmake_minimum_required(VERSION 3.31)`          | 要求运行此项目的 CMake 至少为 3.31，并据此设置相应的策略行为                      |
| `project(Helloworld VERSION 0.1 LANGUAGES CXX)` | 项目名为 `Helloworld`，项目版本为 `0.1`，启用 C++ 语言                   |
| `VERSION`、`LANGUAGES`                           | 都可选；不指定语言时，默认启用 `C` 和 `CXX`，纯 C++ 项目可以显式写 `LANGUAGES CXX` |
| `add_executable(Helloworld helloworld.cpp)`     | 定义名为 `Helloworld` 的可执行目标，将 `helloworld.cpp` 加入它的源文件列表     |

这里有三个容易混淆的地方：

* `3.31` 是 **CMake 的最低版本**，`0.1` 是**这个项目的版本**，二者不是同一回事。
* `Helloworld` 是目标名，不必与源文件同名；但 `helloworld.cpp` 必须与实际的源文件名一致。项目名和目标名在这里恰好相同，也可以不同。
* `add_executable()` 是在**描述如何构建目标**，不是执行到这一行就立即完成编译和链接。它的普通形式是在目标名后列出源文件，也可以先定义目标，再通过后面的 `target_sources()` 补充源文件。

### 构建项目

进入项目文件夹后，通常分两步操作：

```bash
cmake -S . -B build
cmake --build build
```

1. 第一条命令进行**配置与生成**：读取 `CMakeLists.txt`，检查编译环境，并生成构建工具需要的文件。
2. 第二条命令进行**实际构建**：调用对应的构建工具，完成项目源码的编译、库的归档或链接等步骤。

其中，`-S .` 指定当前目录为源码目录，`-B build` 指定构建目录。构建目录可以由 CMake 自动创建。

`cmake .` 也可以配置项目，但会将当前目录用作构建目录；在源码目录直接执行时，生成文件会和源码混在一起。因此这里采用 `-S`、`-B` 分开指定的方式。

查看本机支持的生成器：

```bash
cmake -help
```

输出的 `Generators` 部分会列出可用生成器，并用 `*` 标出默认项。

例如，在 Windows 命令提示符下使用 MinGW：

```bash
cmake -S . -B build-mingw -G "MinGW Makefiles"
cmake --build build-mingw
```

其中：

| 参数   | 说明                 |
| ---- | ------------------ |
| `-S` | 指定源码目录             |
| `-B` | 指定构建目录             |
| `-G` | 指定生成器，也就是要生成哪种构建系统 |

`MinGW Makefiles` 使用 `mingw32-make`，面向 Windows 命令提示符环境，不能直接照搬为 Linux 下的通用命令。Linux 下常见的生成器有 `Unix Makefiles` 和 `Ninja`；具体可用项以本机帮助输出为准，默认选择也可能受 `CMAKE_GENERATOR` 环境变量影响。

**生成器不是编译器，也不是“构建版本”。** 例如，`Unix Makefiles` 生成供 `make` 使用的文件，而具体的 C++ 编译工作由编译器完成。

构建时，可以在项目根目录指定构建目录：

```bash
cmake --build ./build
```

如果终端已经位于构建目录，则执行：

```bash
cmake --build .
```

这里的 `.` 指**当前构建目录**，不是固定指源码目录；也不存在由 CMake 提供的通用裸 `build` 命令。

## 更进一步

### 使用 `target_sources` 指定源文件

在前文中，我们使用 `add_executable(Helloworld helloworld.cpp)` 定义可执行目标。源文件既可以直接列在 `add_executable()` 中，也可以在定义目标后，通过 `target_sources()` 分开添加：

```cmake
add_executable(demo)
target_sources(demo
    PRIVATE
        main.cpp
        complex.cpp
        complex.h
)
```

这个例子中：

* `demo` 是已经定义的目标名；要先创建目标，再给它添加源文件。
* `PRIVATE` 表示这些源文件用于构建 `demo` 自己，不作为它的接口源文件传播给其他目标。
* `main.cpp`、`complex.cpp` 是需要编译的源文件；`complex.h` 可以一并列出，方便管理和 IDE 展示，但普通头文件通常不会因此被单独编译成一个 `.o` 文件。

目标除了源文件，还有头文件搜索目录、编译选项、链接库等属性。下面这些是**目标属性名**，不要和设置它们的命令混为一谈，也不是都写进 `target_sources()`：

| 目标属性                     | 含义 / 常用设置方式                                 |
| ------------------------ | ------------------------------------------- |
| `SOURCES`                | 源文件列表，对应 `target_sources()`                 |
| `INCLUDE_DIRECTORIES`    | 头文件搜索目录，对应 `target_include_directories()`   |
| `COMPILE_DEFINITIONS`    | 编译时的宏定义，对应 `target_compile_definitions()`   |
| `COMPILE_OPTIONS`        | 编译选项，对应 `target_compile_options()`          |
| `LINK_LIBRARIES`         | 链接依赖，对应 `target_link_libraries()`           |
| `LINK_OPTIONS`           | 链接选项，对应 `target_link_options()`             |
| `STATIC_LIBRARY_OPTIONS` | 静态库归档选项，可通过 `set_property()` 设置；不要与普通链接选项混用 |

其中，源文件属性是 `SOURCES`，不是 `SOURCE`。`LINK_FLAGS`、`STATIC_LIBRARY_FLAGS` 是较旧的字符串形式属性；新增配置优先考虑对应的 `LINK_OPTIONS`、`STATIC_LIBRARY_OPTIONS`。

### 在 g++ 下编译与链接

先把 `complex.cpp` 编译为目标文件：

```bash
g++ -c complex.cpp -o complex.o
```

`-c` 表示只编译、不进行最终链接；`-o complex.o` 指定输出文件名。这里的 `.o` 是目标文件，还不是可以直接运行的程序。

再编译 `main.cpp`，并把它与 `complex.o` 链接为可执行文件：

```bash
g++ main.cpp complex.o -o demo
```

归档，生成静态库文件 `libcomplex.a`：

```bash
ar rcs libcomplex.a complex.o
```

静态库通常是若干目标文件的归档。这里把 `complex.o` 装进 `libcomplex.a`，不等于已经完成最终可执行文件的链接。

`ar rcs` 中的三个选项分别是：

* `r`：将成员加入归档，已有同名成员时替换。
* `c`：归档不存在时创建它，并抑制创建提示。
* `s`：创建或更新归档的符号索引，便于后续链接。

查看归档中的成员：

```bash
ar -t libcomplex.a
```

这里会列出 `complex.o` 等成员名，不是查看链接依赖或完整符号内容。接下来可以通过库名链接：

```bash
g++ main.cpp -L. -lcomplex -o demo
```

* `-L.` 表示把当前目录加入库的搜索路径。
* `-lcomplex` 让链接器按平台约定查找库。在 Linux 下，常见候选是 `libcomplex.so`、`libcomplex.a`，不仅是补上前缀 `lib`，还涉及后缀。
* 在 GNU 链接器默认的动态链接模式下，同一搜索目录中同时存在共享库和静态库时，通常优先使用共享库；这不是脱离搜索路径和链接选项的无条件规则。
* 静态库通常要放在引用它的源文件或目标文件后面，所以这里将 `-lcomplex` 写在 `main.cpp` 后。

如果要明确使用这个归档，也可以直接给出文件路径：

```bash
g++ main.cpp ./libcomplex.a -o demo
```

### 在 CMake 下实现

在 CMake 下实现如下：

```cmake
cmake_minimum_required(VERSION 3.30)

project(ComplexCalculator VERSION 0.1 DESCRIPTION "demo" LANGUAGES CXX)

# 添加可执行程序
add_executable(demo)
target_sources(demo
    PRIVATE
        main.cpp
)

# 添加静态库
add_library(complex STATIC)
target_sources(complex PRIVATE complex.cpp complex.h)

# 链接库
target_link_libraries(demo PRIVATE complex)
```

这里的几个命令，分别对应前面手动操作的不同部分：

* `add_library(complex STATIC)` 配合 `target_sources()`，描述把库的源文件编译为目标文件，再归档为静态库。
* `target_link_libraries(demo PRIVATE complex)` 为 `demo` 添加对 `complex` 的链接依赖；这里使用的是目标名，不是直接写磁盘上的 `libcomplex.a`。
* CMake 根据目标关系处理这里的库位置与构建依赖，不需要为这个本项目内的库手写 `-L`、`-l`。

`STATIC` 在这里显式指定静态库。如果只写 `add_library(complex)`，库类型会受 `BUILD_SHARED_LIBS` 控制，并不保证总是静态库。

### 构建参数

在实际构建时，可以使用参数：

* `--verbose`：请求构建工具显示详细命令，便于观察编译、归档和链接过程。
* `--clean-first`：先构建清理目标，再执行构建；不是删除整个构建目录，也不会因此清空 CMake 缓存。

例如：

```bash
cmake --build build --verbose
cmake --build build --clean-first
```

## 变量

### 普通变量

在 CMake 中，普通变量通过 `set()` 设置，形式如下：

```text
set(<variable> <value>... [PARENT_SCOPE])
```

这里的 `<...>` 是占位符，`...` 表示可以有多个值，`[...]` 表示可选项，不是让你原样输入尖括号或方括号。`PARENT_SCOPE` 是关键字，作用在后面的作用域部分说明。

注意事项：

* 建议使用数字、字母、下划线、连接符。
* **区分大小写**。
* 不要用 `CMAKE_` 等保留前缀定义自己的变量，这些名称留给 CMake 使用。

> 在 CMake 中，所有变量的值都存储为字符串。

eg：`set(myVar hello)` 与 `set(myVar "hello")` 效果相同。

### 列表与字符串

多个值会用分号连接：

```cmake
set(myVar hello world)
message("${myVar}")
```

输出：

```text
hello;world
```

此时 `myVar` 保存的仍是字符串，只是在需要列表的上下文中，`hello;world` 会被当作两个元素。`set(myVar hello world)` 和 `set(myVar "hello world")` 并不相同：前者保存 `hello;world`，后者保存带空格的 `hello world`。

#### 引号影响的是当前这次传参

```cmake
set(myVar hello world)
message("${myVar}")
message(${myVar})
```

输出：

```text
hello;world
helloworld
```

第一条 `message()` 接收一个参数 `hello;world`；第二条相当于 `message(hello world)`，接收两个参数，而 `message()` 会直接拼接这些消息参数。

**双引号可以防止本次展开被分号拆成多个参数，但不会给变量附加一个永久的“非列表类型”。** 例如，`set(myVar "hello;world")` 保存的值里仍有分号，后面不加引号展开时仍可能被拆开。

这也不是 shell 那种简单的“展开后按空格拆分”：CMake 不会仅因为变量值中有空格，就把这次变量展开再次按空格拆开。直接书写一个含空格的参数时，则应使用 `"hello world"` 这样的双引号形式。

#### 转义分号

`;` 可以分隔列表元素，`\;` 可以在相应的列表展开中表示元素内部的分号。例如：

```cmake
message(hello;world)
message(hello\;world)
```

输出：

```text
helloworld
hello;world
```

前者把内容拆成了两个参数，后者保留了分号。涉及变量的完整字符串传参时，优先写 `"${myVar}"`，不要只靠叠加反斜杠来猜展开结果。

### 引用变量

当需要读取普通变量时，使用 `${变量名}`。沿用前面的列表变量：

```cmake
set(myVar hello world)
message("myVar=${myVar}")
```

输出：

```text
myVar=hello;world
```

变量引用也可以嵌套：

```cmake
set(myVarr dog)
set(${myVarr} cat)
message("myVarr=${myVarr}")
message("${myVarr}=${${myVarr}}")
```

输出：

```text
myVarr=dog
dog=cat
```

逐句看：

1. `set(myVarr dog)` 定义变量 `myVarr`，它的值是 `dog`。
2. `set(${myVarr} cat)` 先把 `${myVarr}` 展开为 `dog`，实际执行的是 `set(dog cat)`；它设置的是另一个名为 `dog` 的变量，不会把 `myVarr` 改成 `cat`。
3. `${${myVarr}}` 从内向外展开：先得到 `${dog}`，再读取变量 `dog`，得到 `cat`。

如果写成 `\${myVarr}`，则 `$` 被转义，会保留字面量 `${myVarr}`，而不是读取变量。转义适合需要原样显示语法的时候，不适合这里演示实际展开。

### 多行字符串

双引号可以跨多行，并且仍会进行变量替换：

```cmake
set(myVar world)
set(multiline "
Hello,
${myVar}
")
```

这里 `${myVar}` 会替换为 `world`，字符串中的换行会保留，包括这个写法中开头和结尾的换行。

也可使用两对方括号 `[[...]]`：

```cmake
set(myVar world)
set(raw [[Hello,
${myVar}]])
message("${raw}")
```

输出：

```text
Hello,
${myVar}
```

方括号参数中的内容不进行变量替换或反斜杠转义，所以这里保留了 `${myVar}`。如果内容本身包含 `]]`，可以使用 `[=[...]=]`，左右两侧的 `=` 数量需要一致。

`unset(myVar)` 可以取消定义普通变量，注意变量名的大小写要一致。它与 `set(myVar "")` 不完全相同，涉及同名缓存时的区别见后面的例子。

### 环境变量

```cmake
set(ENV{Varname} value)
```

eg：

```cmake
set(ENV{PATH} "$ENV{PATH}:/opt/myDir")
```

> 这个 PATH 拼接示例适用于 Linux 等 Unix 环境。修改只影响当前 CMake 进程及其之后启动的子进程，不会反向修改启动它的 shell 或系统环境，也不会自动影响以后单独运行的构建、测试进程。

### 缓存变量

缓存变量存放在构建目录的 `CMakeCache.txt` 中，能够在多次配置之间保留。**每个构建目录有自己的一份缓存**，不是整个系统共用一份。

形式如下：

```text
set(<variable> <value>... CACHE <type> "<helpstring>" [FORCE])
```

`CACHE`、`FORCE` 是关键字；`type` 主要用于 GUI 等工具如何展示和处理缓存项，不会让 CMake 普通变量变成 C++ 那样的强类型变量。

| `type`     | 用途                                |
| ---------- | --------------------------------- |
| `BOOL`     | 开关，GUI 中通常显示为复选框                  |
| `FILEPATH` | 文件路径                              |
| `PATH`     | 目录路径                              |
| `STRING`   | 字符串                               |
| `INTERNAL` | 内部缓存项，GUI 中不展示；这个类型还隐含 `FORCE` 行为 |

其中，在条件判断中，下列取值按真假处理，英文布尔常量不区分大小写：

| 取值                                                                       | 含义 |
| ------------------------------------------------------------------------ | -- |
| `false`、`no`、`off`、`n`、`0`、空字符串、`IGNORE`、`NOTFOUND`、以 `-NOTFOUND` 结尾的字符串 | 假  |
| `true`、`yes`、`on`、`y`、`1` 或其他非零数，如 `2`、`.1`                              | 真  |

`.1` 确实是非零数，可以表示真；入门开关通常写 `ON` / `OFF` 或 `1` / `0` 更直观。

#### 默认不覆盖已有缓存

`helpstring` 用于说明缓存项的用途。普通的 `set(... CACHE ...)` 默认只在缓存项不存在时设置它，不会每次配置都覆盖已有值。

假设 `CACHE_DEMO` 尚不存在：

```cmake
set(CACHE_DEMO first CACHE STRING "演示缓存覆盖")
set(CACHE_DEMO second CACHE STRING "演示缓存覆盖")
message("CACHE_DEMO=$CACHE{CACHE_DEMO}")

set(CACHE_DEMO second CACHE STRING "演示缓存覆盖" FORCE)
message("CACHE_DEMO=$CACHE{CACHE_DEMO}")
```

首次运行的输出：

```text
CACHE_DEMO=first
CACHE_DEMO=second
```

第二条 `set(... CACHE ...)` 没有覆盖已存在的缓存。带 `FORCE` 的那一次会强制写入新值。

**`FORCE` 是选项，不是一个名为 `FORCE` 的变量。** 每次执行到带 `FORCE` 的这条命令，都会按给定值覆盖缓存项，也可能覆盖用户通过 `-D` 提供的值，不宜给所有缓存项都加上它。

对于 `BOOL` 类型的缓存变量，也可使用 `option()`：

```text
option(<variable> "<helpstring>" [初始值])
```

例如：

```cmake
option(ENABLE_DEMO "是否启用 demo" OFF)
```

省略初始值时默认为 `OFF`；它也不是每次执行都强制覆盖已有设置的开关。

#### 与普通变量的关系

**读取 `${变量名}` 时，先找普通变量，再查同名缓存。** 普通变量只是遮住缓存项，不是把它删除。

假设 `SHADOW_DEMO` 的缓存项尚不存在：

```cmake
set(SHADOW_DEMO cache CACHE STRING "演示普通变量遮蔽缓存")
set(SHADOW_DEMO normal)
message("normal=${SHADOW_DEMO}")
message("cache=$CACHE{SHADOW_DEMO}")

unset(SHADOW_DEMO)
message("after unset=${SHADOW_DEMO}")

set(SHADOW_DEMO "")
message("after empty=[${SHADOW_DEMO}]")
```

输出：

```text
normal=normal
cache=cache
after unset=cache
after empty=[]
```

* `$CACHE{SHADOW_DEMO}` 直接访问缓存项，不经过同名普通变量。
* `unset(SHADOW_DEMO)` 取消普通变量的定义，此后读取 `${SHADOW_DEMO}` 时可以重新看到缓存值。
* `set(SHADOW_DEMO "")` 定义了一个值为空的普通变量，它仍会遮住缓存，因此读取结果是空字符串。

通常，缓存变量采用全大写，普通变量采用驼峰；这只是命名习惯，不是语法规则，变量种类也不是由大小写决定的。

#### 在命令行中设置与删除

在命令行下，可通过 `-D` 创建或更新缓存变量：

```bash
cmake -S . -B build -D "MY_VAR:STRING=hello world"
```

这里 `MY_VAR` 是缓存项名，`STRING` 是缓存类型，值为含空格的 `hello world`。引号用于让 shell 把这个赋值完整地传给 CMake；没有 `FORCE` 等覆盖逻辑时，脚本中的默认缓存值不会覆盖这个用户设置。

使用 `-U` 移除匹配的缓存项：

```bash
cmake -S . -B build -U "MY*"
```

`*` 是通配符，因此这里仅删除名称以 `MY` 开头的缓存项，不是删除所有缓存变量。引号也能避免 shell 提前展开通配符。

移除后仍会继续执行配置；如果配置脚本再次定义这个缓存项，它还可能按脚本中的默认值重新建立。

### 作用域

| 变量 / 作用域 | 范围                                    |
| -------- | ------------------------------------- |
| 普通变量     | 按目录、函数或变量块作用域查找，内层可以定义同名变量遮蔽外层        |
| 缓存变量     | 属于当前构建目录的缓存，可跨普通变量作用域访问，但仍可能被同名普通变量遮住 |

下面用已有的 `block()` 对照局部修改、写入父作用域和离开块时传播的区别。`block()` 从 CMake 3.25 开始提供。

> CMake 的行注释使用 `#`，不是 C/C++ 中的 `//`。

#### 局部作用域

```cmake
set(myVar cat)
block()
    set(myVar dog)
    message("myVar=${myVar}")
endblock()
message("myVar=${myVar}")
```

块内能够读取外层的变量；但普通的 `set(myVar dog)` 只在块内建立新的局部绑定，遮住外层的同名变量。离开块后，这个局部绑定消失，外层变量没有被修改。

#### `PARENT_SCOPE`

```cmake
set(myVar cat)
block()
    set(myVar dog PARENT_SCOPE) # 写入父作用域，不修改块内的原有值
    message("myVar=${myVar}")
endblock()
message("myVar=${myVar}")
```

`PARENT_SCOPE` 把值写入**最近一级父作用域**，当前作用域中的值保持不变。因此它不是建立引用，也不是让之后对局部变量的任何修改都自动同步到外层。

#### `PROPAGATE`

```cmake
set(myVar cat)
block(PROPAGATE myVar)
    set(myVar dog) # 先修改块内变量，离开块时才向父作用域传播
    message("myVar=${myVar}")
endblock()
message("myVar=${myVar}")
```

如果希望向外传播变量，必须在 `PROPAGATE` 后面列出**希望传播的变量名**。这里只写 `block(PROPAGATE)` 不会自动传播所有变量。

它允许块内先正常修改局部变量，在离开块时把所列变量的最终状态传播到父作用域，也不是引用机制。

三个例子的输出对照：

| 写法                       | 块内的 `message()` | 块外的 `message()` |
| ------------------------ | --------------- | --------------- |
| 普通 `block()`             | `myVar=dog`     | `myVar=cat`     |
| `set(... PARENT_SCOPE)`  | `myVar=cat`     | `myVar=dog`     |
| `block(PROPAGATE myVar)` | `myVar=dog`     | `myVar=dog`     |

相关官方文档：[CMake 命令行](https://cmake.org/cmake/help/v3.31/manual/cmake.1.html)、[CMake 语言](https://cmake.org/cmake/help/v3.31/manual/cmake-language.7.html)、[`set()`](https://cmake.org/cmake/help/v3.31/command/set.html)、[`block()`](https://cmake.org/cmake/help/v3.31/command/block.html)。
