# CMake学习

### 任务目标：

1. 引言
2. 理解 CMake 的作⽤及⼯作机制
3. 熟练编写 CMakeLists.txt
4. 学会源码外构建
5. 学会安装与配置
6. 学会排查问题
7. 感悟
8. 鸣谢

### 一、引言

笔者在Linux系统中前前后后安装、配置、学习了各种命令与工具，那么这次接触到的CMake又是什么呢？

### 二、理解 CMake 的作⽤及⼯作机制

> ### **CMake 是什么？**
>
> - **构建系统生成器**：不是直接构建项目，而是生成其他构建系统需要的文件（如 Makefile、Visual Studio 项目等）
> - **跨平台**：同一套 CMake 配置可以在 Linux、Windows、macOS 上使用
> - **管理复杂依赖**：自动处理库依赖、编译器标志、安装规则等

按照笔者理解，CMake的工作机制就是通过对CMakeLists.txt的编辑来进行我们c++程序的配置，是在编程之前必不可少的一项操作。

### 三、CMakeLists.txt

通常情况下，最最基础的CMakeLists.txt需要包含三个部分：cmake_minimum_required、project、add_executable。

``` cmake
# 指定 CMake 最低版本要求
cmake_minimum_required(VERSION 3.10)

# 定义项目名称和使用的编程语言
project(MyProject CXX)

# 添加可执行文件
add_executable(my_app main.cpp)
```

第一行用于确保CMake的版本符合要求，第二行用来告诉机器你要使用的编程语言是什么，这里“CXX”指的就是c++，“C”指的是C语言。第三行用来添加可执行文件，这样我们就获得了一个最基础的CMakeLists.txt。

另外，还有一种操作：

``` cmake
# 创建静态库
add_library(my_lib STATIC lib1.cpp lib2.cpp)

# 创建动态库（共享库）
add_library(my_lib SHARED lib1.cpp lib2.cpp)

# 创建仅头文件的库
add_library(my_header_lib INTERFACE)
target_include_directories(my_header_lib INTERFACE include/)


# 将库链接到可执行文件
add_executable(my_app main.cpp)
add_library(math_lib STATIC math.cpp)

target_link_libraries(my_app PRIVATE math_lib)

# 链接系统库
target_link_libraries(my_app PRIVATE pthread m)
```

依照笔者的理解，这里的库应该可以认为是与c++中的头文件类似的一种东西，当我们要使用一些已经被封装好了的东西时就可以调用库文件。

另外，CMake还有多模块工程管理能力，也就是在一个CMake文件中分为不同的模块，每个模块有自己的功能，自己的代码，以及自己的CMakeLists.txt文件。操作多模块也有简便的方法，下面以根目录的CMakeLists.txt文件内容为例：

``` cmake
cmake_minimum_required(VERSION 3.10)			#必需的代码
project(MyProject VERSION 1.0.0 LANGUAGES CXX)   #必需的代码

# 添加子目录（会自动调用子目录中的 CMakeLists.txt）
add_subdirectory(math)
add_subdirectory(utils)

# 创建主可执行文件
add_executable(my_app main.cpp)

# 链接子目录中创建的库
target_link_libraries(my_app PRIVATE math_lib utils_lib)

# 包含头文件路径
target_include_directories(my_app PRIVATE
    ${CMAKE_SOURCE_DIR}/include
)
```

另外，当我们觉得电脑中有的库不够用的时候，我们也可引入第三方库。我们先前安装的openCV中就可以找到一些第三方库，这时我们就可用` find_package()` 来查找并链接第三方库了：

``` cmake
# 查找 OpenCV，REQUIRED 表示必须找到，否则报错
find_package(OpenCV REQUIRED)

# 使用找到的包
target_include_directories(my_app PRIVATE ${OpenCV_INCLUDE_DIRS})
target_link_libraries(my_app PRIVATE ${OpenCV_LIBS})

# 可以指定组件
find_package(Boost REQUIRED COMPONENTS filesystem system)
```

### 四、学会源码外构建

源码外构建顾名思义，是指在源代码之外进行调试的操作。这样一来我们就可以有后悔的机会，不至于牵一发而动全身，小小更改一个地方就满盘皆输。同时，源码外构建也让我们可以同时Debug、Release。

源码外构建的标准流程：

``` cmake
# 1. 创建构建目录（与源码分离）
mkdir build
cd build

# 2. 配置项目（生成构建系统文件）
cmake ..

# 3. 编译项目
make

# 4. 运行程序
./my_app
```

### 五、学会安装与配置

##### 安装

这里的安装和终端里面的安装指令就不一样了，这里用的是` install` 来进行安装。以下这些是笔者整理的较为常用的install指令：

``` cmake
# 安装可执行文件到 bin 目录
install(TARGETS my_app
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
)

# 安装头文件
install(DIRECTORY include/
    DESTINATION include
    FILES_MATCHING PATTERN "*.h"
)

# 安装库
install(TARGETS math_lib utils_lib
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
)

# 安装配置文件
install(FILES config.ini DESTINATION etc)
```

具体安装流程：

``` shell
# 配置时指定安装路径
cmake -DCMAKE_INSTALL_PREFIX=/usr/local ..

# 编译
make

# 安装（可能需要sudo）
sudo make install

# 或者直接使用cmake安装
cmake --install . --prefix /my/install/path
```

### 六、问题排查

笔者在这里利用deepseek进行模拟那些问题出现时的情况。

##### 问题一：路径错误

路径错误可能是把终端指令与cmake指令搞混了导致的错误，比如试图在cmake中使用相对路径的关键字等。

这时我们可以使用更加合乎位置的cmake指令来编写：

``` cmake
include_directories(../include)  # 在子目录中可能出错

# 改为
include_directories(${CMAKE_SOURCE_DIR}/include)
target_include_directories(my_lib PUBLIC 
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:include>
)
```

##### 问题二：找不到依赖

如果在编写的时候机器告诉你找不到依赖，这时候我们就要手动定位我们的依赖在什么文件夹里面，然后用如下命令更改：

``` cmake
# 设置查找路径
list(APPEND CMAKE_PREFIX_PATH "/opt/opencv")
list(APPEND CMAKE_LIBRARY_PATH "/opt/opencv/lib")
```

###### 问题三、版本不兼容

当我们发现版本出现问题的时候，应该先检查自己的版本们看看是真的出问题了还是错误报告。如果真的出问题了，吗，额

``` cmake
# 检查版本
find_package(OpenCV REQUIRED)
if(${OpenCV_VERSION} VERSION_LESS "4.0.0")
    message(FATAL_ERROR "OpenCV 版本太低，需要 4.0.0 或更高")
endif()
```

### 七、感悟

至此笔者对CMake的学习暂时告一段落。通过学习CMake，笔者了解了源码外构建的奥妙之处，学会了预测可能出现的问题等等。~~（什么你说怎么没有实机演示截图？没关系，后面的实践任务弥补了这一点）~~ 

### 八、鸣谢

~~(tips:笔者其实并不知道应不应该有这部分，但是笔者决定加上)~~

在此次学习过程中笔者主要要感谢群中学长拓展了笔者的视野。

其次要感谢deepseek以及网上各路大神的帖子帮助了我的学习。

# THE END