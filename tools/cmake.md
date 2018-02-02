# 常用配置

限定cmake最旧版本，指定工程名:
```
cmake_minimum_required(VERSION 3.9.0)
project(imx6q_pro)
```

指定C++版本号:
```
set(CMAKE_CXX_STANDARD 11)
```

使能版本设置:
```
set(CMAKE_CXX_STANDARD_REQUIRED ON)
```

设置自动添加构建路径，不会访问子目录，通过$(CMAKE_CURRENT_SOURCE_DIR)和$(CMAKE_CURRENT_BINARY_DIR)访问。默认关闭:
```
set(CMAKE_INCLUDE_CURRENT_DIR ON)
```

设置构建目标目录:
```
if(NOT CMAKE_SYSTEM_PROCESSOR)
  set(CMAKE_SYSTEM_PROCESSOR x86_64)
endif()

message("Build for ${CMAKE_SYSTEM_NAME}-${CMAKE_SYSTEM_PROCESSOR}")

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/bin/${CMAKE_SYSTEM_PROCESSOR}")
```

添加子路径，指定CMakeLists.txt的位置:
```
add_subdirectory(src/tests)
```

在项目中添加一个可执行文件:
```
add_executable(
		test
		main.cpp
	)
```

添加库:
```
find_package(Threads)
target_link_libraries(test ${CMAKE_THREAD_LIBS_INIT})
```

# 工具链

如果用到交叉编译，可以指定对应的交叉编译工具。

创建一个配置文件，例如CMakeToolchainIMX6.txt。

```
# this one is important
SET(CMAKE_SYSTEM_NAME Linux)
#this one not so much
SET(CMAKE_SYSTEM_VERSION 1)
```

放一个标志，用于识别:
```
set(CMAKE_SYSTEM_PROCESSOR ARM)
```

指定编译工具:
```
# specify the cross compiler
SET(CMAKE_C_COMPILER    /.../arm-imx6-linux-gnueabi-gcc)
SET(CMAKE_CXX_COMPILER  /.../arm-imx6-linux-gnueabi-g++)

# where is the target environment 
SET(CMAKE_FIND_ROOT_PATH   /.../x-tools/)
```

自动寻找头文件、库和包:
```
# search for programs in the build host directories
SET(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
# for libraries and headers in the target directories
SET(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
SET(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
```

生成时指定Toolchain:
```
cmake . -Bbuild -DCMAKE_TOOLCHAIN_FILE=CMakeToolchainIMX6.txt
```
