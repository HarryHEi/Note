---
title: cmake
date: 2018-2-3 9:59:36
tags: [cpp]
---

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

# Gstreamer查询脚本

文件: FindGstreamer.cmake

找到PkgConfig
```

find_package(PkgConfig)

# Helper macro to find a GStreamer plugin (or GStreamer itself)
#   _component_prefix is prepended to the _INCLUDE_DIRS and _LIBRARIES variables (eg. "GSTREAMER_AUDIO")
#   _pkgconfig_name is the component's pkg-config name (eg. "gstreamer-1.0", or "gstreamer-video-1.0").
#   _library is the component's library name (eg. "gstreamer-1.0" or "gstvideo-1.0")
macro(FIND_GSTREAMER_COMPONENT _component_prefix _pkgconfig_name _library)

    string(REGEX MATCH "(.*)>=(.*)" _dummy "${_pkgconfig_name}")
    if ("${CMAKE_MATCH_2}" STREQUAL "")
        pkg_check_modules(PC_${_component_prefix} "${_pkgconfig_name} >= ${GStreamer_FIND_VERSION}")
    else ()
        pkg_check_modules(PC_${_component_prefix} ${_pkgconfig_name})
    endif ()
    set(${_component_prefix}_INCLUDE_DIRS ${PC_${_component_prefix}_INCLUDE_DIRS})

    find_library(${_component_prefix}_LIBRARIES
        NAMES ${_library}
        HINTS ${PC_${_component_prefix}_LIBRARY_DIRS} ${PC_${_component_prefix}_LIBDIR}
    )
endmacro()
```

找到Gstreamer，这里保留了参数${Gstreamer_Root_Dir}
```
FIND_GSTREAMER_COMPONENT(GSTREAMER gstreamer-1.0 gstreamer-1.0)
FIND_GSTREAMER_COMPONENT(GSTREAMER_BASE gstreamer-base-1.0 gstbase-1.0)

find_path(Gstreamer_Include gst/gst.h
  ${Gstreamer_Root_Dir}/gstreamer/1.0/x86_64/include/gstreamer-1.0
)

find_path(Gstreamer_Gl_Include gst/gl/gstglconfig.h
  ${Gstreamer_Root_Dir}/gstreamer/1.0/x86_64/lib/gstreamer-1.0/include
)

find_path(Gstreamer_Glib_Include glib.h
  ${Gstreamer_Root_Dir}/gstreamer/1.0/x86_64/include/glib-2.0
)

find_path(Gstreamer_Xml_Include libxml/xmlIO.h
  ${Gstreamer_Root_Dir}/gstreamer/1.0/x86_64/include/libxml2
)

find_path(Gstreamer_X264_Include x264.h
  ${Gstreamer_Root_Dir}/gstreamer/1.0/x86_64/include
)

find_path(Gstreamer_Glibconfig_Include glibconfig.h
  ${Gstreamer_Root_Dir}/gstreamer/1.0/x86_64/lib/glib-2.0/include
)
```

找到元件:
```
# -------------------------
# 2. Find GStreamer plugins
# -------------------------

FIND_GSTREAMER_COMPONENT(GSTREAMER_APP gstreamer-app-1.0 gstapp-1.0)
FIND_GSTREAMER_COMPONENT(GSTREAMER_AUDIO gstreamer-audio-1.0 gstaudio-1.0)
FIND_GSTREAMER_COMPONENT(GSTREAMER_FFT gstreamer-fft-1.0 gstfft-1.0)
FIND_GSTREAMER_COMPONENT(GSTREAMER_GL gstreamer-gl-1.0 gstgl-1.0)
FIND_GSTREAMER_COMPONENT(GSTREAMER_MPEGTS gstreamer-mpegts-1.0>=1.4.0 gstmpegts-1.0)
FIND_GSTREAMER_COMPONENT(GSTREAMER_PBUTILS gstreamer-pbutils-1.0 gstpbutils-1.0)
FIND_GSTREAMER_COMPONENT(GSTREAMER_TAG gstreamer-tag-1.0 gsttag-1.0)
FIND_GSTREAMER_COMPONENT(GSTREAMER_VIDEO gstreamer-video-1.0 gstvideo-1.0)
FIND_GSTREAMER_COMPONENT(GSTREAMER_LIB gstreamer-lib-1.0 glib-2.0)
FIND_GSTREAMER_COMPONENT(GSTREAMER_OBJECT gstreamer-object-1.0 gobject-2.0)
```

保存找到的目录:
```
# ------------------------------------------------
# 3. Process the COMPONENTS passed to FIND_PACKAGE
# ------------------------------------------------
set(_GSTREAMER_REQUIRED_VARS GSTREAMER_INCLUDE_DIRS GSTREAMER_LIBRARIES GSTREAMER_VERSION GSTREAMER_BASE_INCLUDE_DIRS GSTREAMER_BASE_LIBRARIES)

foreach (_component ${GStreamer_FIND_COMPONENTS})
    set(_gst_component "GSTREAMER_${_component}")
    string(TOUPPER ${_gst_component} _UPPER_NAME)

    list(APPEND _GSTREAMER_REQUIRED_VARS ${_UPPER_NAME}_INCLUDE_DIRS ${_UPPER_NAME}_LIBRARIES)
endforeach ()

include(FindPackageHandleStandardArgs)
FIND_PACKAGE_HANDLE_STANDARD_ARGS(GStreamer REQUIRED_VARS _GSTREAMER_REQUIRED_VARS
                                            VERSION_VAR   GSTREAMER_VERSION)

mark_as_advanced(
    GSTREAMER_APP_INCLUDE_DIRS
    GSTREAMER_APP_LIBRARIES
    GSTREAMER_AUDIO_INCLUDE_DIRS
    GSTREAMER_AUDIO_LIBRARIES
    GSTREAMER_BASE_INCLUDE_DIRS
    GSTREAMER_BASE_LIBRARIES
    GSTREAMER_FFT_INCLUDE_DIRS
    GSTREAMER_FFT_LIBRARIES
    GSTREAMER_GL_INCLUDE_DIRS
    GSTREAMER_GL_LIBRARIES
    GSTREAMER_INCLUDE_DIRS
    GSTREAMER_LIBRARIES
    GSTREAMER_MPEGTS_INCLUDE_DIRS
    GSTREAMER_MPEGTS_LIBRARIES
    GSTREAMER_PBUTILS_INCLUDE_DIRS
    GSTREAMER_PBUTILS_LIBRARIES
    GSTREAMER_TAG_INCLUDE_DIRS
    GSTREAMER_TAG_LIBRARIES
    GSTREAMER_VIDEO_INCLUDE_DIRS
    GSTREAMER_VIDEO_LIBRARIES
    GSTREAMER_LIB_INCLUDE_DIRS
    GSTREAMER_LIB_LIBRARIES
    GSTREAMER_OBJECT_INCLUDE_DIRS
    GSTREAMER_OBJECT_LIBRARIES
)
```

文件: CMakeLists.txt

设置查询脚本所在位置，这里放在cmake目录下:
```
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)
```

设置查询脚本预留的位置变量:
```
set(Gstreamer_Root_Dir d:)
```

开始查询Gstreamer:
```
find_package(GStreamer)
```

得到了所有的Gstreamer库文件:
```
set(
	GSTREAMER_LIBRARIES
	${GSTREAMER_LIBRARIES}
	${GSTREAMER_APP_LIBRARIES}
	${GSTREAMER_AUDIO_LIBRARIES}
	${GSTREAMER_VIDEO_LIBRARIES}
	${GSTREAMER_LIB_LIBRARIES}
	${GSTREAMER_OBJECT_LIBRARIES}
)
```

包含所有Gstreamer头文件:
```
include_directories(${Gstreamer_Include})
include_directories(${Gstreamer_Gl_Include})
include_directories(${Gstreamer_Glib_Include})
include_directories(${Gstreamer_Xml_Include})
include_directories(${Gstreamer_X264_Include})
include_directories(${Gstreamer_Glibconfig_Include})
```

添加执行文件、库:
```
add_executable(${TARGET} ${SOURCES})

target_link_libraries(${TARGET} ${GSTREAMER_LIBRARIES})
```

# QT MOC、qrc以及ui文件支持

开启自动查询:
```
set(CMAKE_AUTOMOC ON)
set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTORCC ON)
```

找到Qt库:
```
find_package(Qt5Widgets)
find_package(Qt5Core)
find_package(Qt5Network)
find_package(Qt5Gui)
```

设置库:
```
set(
	QT5_LIBRARIES
	Qt5::Widgets 
	Qt5::Core 
	Qt5::Network  
	Qt5::Gui
	Qt5::Sql
)
```

设置源文件，这里要添加一个WIN32，否则运行界面程序会弹出命令行:
```
set(
	SOURCES
	WIN32
	${PROJECT_SOURCE_DIR}/main.cpp
)
```

设置UI文件:
```
set(
	UI_SOURCES
	${PROJECT_SOURCE_DIR}/test.ui
)
```

包装ui:
```
qt5_wrap_ui(UI_GENERATED_HEADERS ${UI_SOURCES})
```

添加执行文件:
```
add_executable(${TARGET} ${SOURCES} ${UI_GENERATED_HEADERS})
```

连接库:
```
target_link_libraries(${TARGET} ${QT5_LIBRARIES})
```

编译:
```
在主目录下运行cmake . -G "Visual Studio 14 2015 Win64" -Bbuild
在主目录下运行cmake --build build
```
