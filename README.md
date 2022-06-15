

 
# LWM_MigrateTest(搬运TheLartians/ModernCppStarter)

建立一个新的C++项目通常需要大量的准备工作和模板代码，对于带有测试、可执行文件和持续集成的现代C++项目更是如此。
这个模板是在许多以前的项目中学习的结果，应该有助于减少建立现代C++项目所需的工作。

## 特性

- [现代化的CMake实践](https://pabloariasal.github.io/2018/02/19/its-time-to-do-cmake-right/)
- 适用于任何规模的单头库和项目
- 库和可执行代码的干净分离
- 集成测试套件
- 通过[GitHub Actions](https://help.github.com/en/actions/)进行持续集成
- 通过[codecov]进行代码覆盖(https://codecov.io)
- 通过[Clang-format](https://clang.llvm.org/docs/ClangFormat.html)和[cmake-format](https://github.com/cheshirekow/cmake_format)的[Format.cmake](https://github.com/TheLartians/Format.cmake)强制执行代码格式化
- 通过[CPM.cmake](https://github.com/TheLartians/CPM.cmake)进行可重复的依赖性管理
- 通过[PackageProject.cmake](https://github.com/TheLartians/PackageProject.cmake)实现具有自动版本信息和头文件生成的可安装目标
- 通过[Doxygen](https://www.doxygen.nl)和[GitHub Pages](https://pages.github.com)自动进行[文档](https://thelartians.github.io/ModernCppStarter)和部署
- 支持[sanitizer工具，以及更多工具](#additional-tools)

## 使用方法

### 根据你的需要调整模板

- 使用这个 repo [作为一个模板](https://help.github.com/en/github/creating-cloning-and-archiving-repositories/creating-a-repository-from-a-template)。
- 在相关的CMakeLists.txt中用你的项目名称替换所有出现的 "Greeter"。
  - 这里的大写字母很重要。`Greeter`是指项目的名称，而`greeter`用于文件名。
  - 记得重命名`include/greeter`目录，以使用你的项目的小写名称，并相应地更新所有相关的`#include`s。
- 用你自己的源文件替换
- 对于只用头的库：见[CMakeLists.txt](CMakeLists.txt)中的注释
- 将[你的项目的codecov token](https://docs.codecov.io/docs/quick-start)添加到你的项目的github secrets的`CODECOV_TOKEN`下。
- 编码愉快!

最终，你可以删除任何不使用的文件，例如独立目录或与你的项目无关的github工作流程。
随意用适合你的项目的许可证来替换。

为了干净地分离库和子项目的代码，外部的`CMakeList.txt`只定义了库本身，而测试和其他子项目则自成一体，放在自己的目录里。
在开发过程中，通常方便[一次性构建所有子项目]（#build-everything-at-once）。

### 构建和运行独立的目标

使用下面的命令来构建和运行可执行的目标。

```bash
cmake -S standalone -B build/standalone
cmake --build build/standalone
./build/standalone/Greeter --help
```

### 构建和运行测试套件

在项目的根目录下使用以下命令来运行测试套件。

```bash
cmake -S test -B build/test
cmake --build build/test
CTEST_OUTPUT_ON_FAILURE=1 cmake --build build/test --target test

# 或者直接调用可执行文件。
./build/test/GreeterTests
```


要收集代码覆盖率信息，请使用`-DENABLE_TEST_COVERAGE=1`选项运行CMake。

### 运行clang-format

在项目的根目录下使用以下命令来检查和修复C++和CMake的源码风格。
这需要在当前系统中安装_clang-format_、_cmake-format_和_pyyaml_。

```bash
cmake -S test -B build/test

# 查看变化
cmake --build build/test --target format

# 应用改动
cmake --build build/test --target fix-format
```

详见 [Format.cmake](https://github.com/TheLartians/Format.cmake)。

### 构建文档

每当[GitHub Release](https://help.github.com/en/github/administering-a-repository/managing-releases-in-a-repository)被创建时，文档就会被自动构建并[发布](https://thelartians.github.io/ModernCppStarter)。
要手动构建文档，请调用以下命令。

```bash
cmake -S documentation -B build/doc
cmake --build build/doc --target GenerateDocs
# 查看文档
打开 build/doc/doxygen/html/index.html
```

要在本地构建文档，你需要在你的系统上安装Doxygen、jinja2和Pygments。

### 一次性构建所有东西

该项目还包括一个`all`目录，允许同时构建所有目标。
这在开发过程中很有用，因为它可以将所有的子项目暴露在你的IDE中，避免了库的重复构建。

```bash
cmake -S all -B build
cmake --build build
```
# 运行测试
./build/test/GreeterTests
# 格式化代码
cmake --build build --target fix-format
# 运行单机
./build/standalone/Greeter --help
# 构建文档
```
cmake --build build --target GenerateDocs
```

### 额外的工具

测试和独立子项目包括[tools.cmake](cmake/tools.cmake)文件，用于通过CMake配置参数按需导入额外工具。
目前支持以下内容。

#### Sanitizers

可以通过在CMake中配置`-DUSE_SANITIZER=<Address | Memory | MemoryWithOrigins | Undefined | Thread | Leak | 'Address;Undefined'>来启用Sanitizers。

#### 静态分析器

静态分析器可以通过设置`-DUSE_STATIC_ANALYZER=<clang-tidy | iwyu | cppcheck>`来启用，或者设置引号中的组合，用分号隔开。
默认情况下，分析器会自动查找配置文件，如`.clang-format`。
其他参数可以通过设置`CLANG_TIDY_ARGS`、`IWYU_ARGS`或`CPPCHECK_ARGS`变量传递给分析器。

#### Ccache

缓存可以通过配置`-DUSE_CCACHE=<ON | OFF>`来启用。

## FAQ

> 我可以将其用于仅有头文件的库吗？

可以，但是你需要将库的类型改为 "INTERFACE "库，如[CMakeLists.txt](CMakeLists.txt)中所记载的。
参见 [here](https://github.com/TheLartians/StaticTypeInfo) 以了解一个基于模板的纯头文件库的例子。

> 我不需要一个独立的目标/文档。我怎样才能去掉它？

只需删除单机/文档目录和根据github工作流文件。

> 我可以在同一时间开发单机独立程序和测试程序吗？/ 我怎样才能告诉我的IDE所有的子项目？

为了保持模板的模块化，所有从库中衍生出来的子项目都被分离到它们自己的CMake模块中。
这种方法使第三方项目重新使用项目库的代码变得微不足道。
为了让IDE看到项目的全部范围，该模板包括`all`目录，它将为所有子项目创建一个单一的构建。
使用这个目录作为主目录以获得最佳的IDE支持。

> 我看到你在使用`GLOB`在CMakeLists.txt中添加源文件。这不是很邪恶吗？

Glob被认为是不好的，因为对源文件结构的任何改变[可能不会被CMake的构建器自动捕获](https://cmake.org/cmake/help/latest/command/file.html#filesystem)，你需要在改变时手动调用CMake。
  我个人更喜欢 "GLOB "解决方案，因为它很简单，但可以自由地将其改为明确列出源文件。

> 我想创建依赖我的库的额外目标。我应该修改主要的CMakeLists以包括它们吗？

避免从库的CMakeLists中包括派生项目（尽管这在C++世界中很常见），因为这有效地颠倒了依赖关系树，使构建系统难以推理。
相反，用CMakeLists创建一个新的目录或项目，将该库作为依赖关系加入（例如，像[standalone](standalone/CMakeLists.txt)目录）。
根据不同的类型，可能有必要将这些组件移到一个单独的仓库中，并引用库的特定提交或版本。
这样做的好处是单个库和组件可以独立改进和更新。

> 你建议使用CPM.cmake来添加外部依赖。这是否会迫使我的库的用户也使用 CPM.cmake？

[CPM.cmake](https://github.com/TheLartians/CPM.cmake)对于库的用户来说应该是不可见的，因为它是一个独立的CMake脚本。
如果出现问题，用户可以通过定义 CMake 或 env 变量 [`CPM_USE_LOCAL_PACKAGES`](https://github.com/cpm-cmake/CPM.cmake#options) 来选择退出，这将覆盖所有对 `CPMAddPackage` 的调用，并使用相应的 `find_package` 调用。
这也使用户能够使用他们最喜欢的外部C++依赖管理器，如vcpkg或Conan来使用该项目。

> 我可以离线配置和构建我的项目吗？

构建项目不需要互联网连接，但是当使用CPM时，缺失的依赖项会在配置时下载。
为了避免重复下载，强烈建议设置一个CPM.cmake缓存目录，例如。`export CPM_SOURCE_CACHE=$HOME/.cache/CPM`。
这将使浅层克隆成为可能，并允许离线配置的依赖性在缓存中已经可用。

> 我可以使用CPack为我的项目创建一个软件包安装程序吗？

由于有很多可能的选项和配置，这不在本模板的范围内（尚未）。请参阅 [CPack 文档](https://cmake.org/cmake/help/latest/module/CPack.html) 以了解更多关于设置CPack安装器的信息。

> 这太多了，我只想玩玩C++代码和测试一些库。

也许 [MiniCppStarter](https://github.com/TheLartians/MiniCppStarter) 是适合你的东西!

## 相关项目和替代方案

- [**ModernCppStarter & PVS-Studio Static Code Analyzer**](https://github.com/viva64/pvs-studio-cmake-examples/tree/master/modern-cpp-starter): 关于如何使用ModernCppStarter与PVS-Studio静态代码分析器的官方说明。
- [**lefticus/cpp_starter_project**](https://github.com/lefticus/cpp_starter_project/)。一个流行的C++启动器项目，创建于2017年。
- [**filipdutescu/modern-cpp-template**](https://github.com/filipdutescu/modern-cpp-template)。一个最近的启动项目，使用更传统的方法进行CMake结构和依赖性管理。
- [**vector-of-bool/pitchfork**](https://github.com/vector-of-bool/pitchfork/)。Pitchfork是一套C++项目公约。
