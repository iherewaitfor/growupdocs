# libcef文档2--CEF入门： 构建自己的cefsimple工程
该文档主要描述如何从cef工程中，去掉多余的工程，只保留cefsimple工程。并将链接运行库从MT修改为MD。
## 1. 移动cefsimple工程到要目录并删除test
1. 将cef_binary_3.3626.1895.g7001d56_windows32.tar.bz2解压到J:\ceftest\cef_3626_3
2. 将cefsimple从test文件夹中复制到cef_3626_3文件中。此时根目录变成如下
```C++
cefsimple
cef_paths.gypi
cef_paths2.gypi
cmake
CMakeLists.txt
Debug
include
libcef_dll
LICENSE.txt
README.txt
Release
Resources
```
## 2. 修改要目录CMakeLists.txt
修改将J:\ceftest\cef_3626_3\CMakeLists.txt。只保留cefsimple工程，并修改相关路径。具体操作如下，将J:\ceftest\cef_3626_3\CMakeLists.txt文件中的。
```C++
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/tests")
  add_subdirectory(tests/cefclient)
  add_subdirectory(tests/cefsimple)
  add_subdirectory(tests/gtest)
  add_subdirectory(tests/ceftests)
endif()
```
修改为
```C++
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}")
  add_subdirectory(cefsimple)
endif()
```
或直接修改为
```C++
add_subdirectory(cefsimple)
```
## 3. 修改编译选项
修改文件J:\ceftest\cef_3626\cmake\cef_variables.cmake,
### 将链接运行库的MT修改为MD
将
```C++
  set(CEF_RUNTIME_LIBRARY_FLAG "/MT" CACHE STRING "Optional flag specifying which runtime to use")
```
修改为
```C++
  set(CEF_RUNTIME_LIBRARY_FLAG "/MD" CACHE STRING "Optional flag specifying which runtime to use")
```
### 修改配置，把sanbox改为off
Sanbox只支持MT，使用MD后，要关掉SANBOX。可参考文档[https://bitbucket.org/chromiumembedded/cef/wiki/LinkingDifferentRunTimeLibraries.md](https://bitbucket.org/chromiumembedded/cef/wiki/LinkingDifferentRunTimeLibraries.md)。具体操作如下。
将
```C++
option(USE_SANDBOX "Enable or disable use of the sandbox." ON)
```
修改为
```C++
option(USE_SANDBOX "Enable or disable use of the sandbox." OFF)
```
## 4. 使用cmake配置和生成工程
参考《libcef文档1--CEF入门：cef windows 示例工程编译运行》使用cmake先configure,后generate生成解决方案。此时只包括4个工程，分别是
1. **ALL_BUILD**： cmake相关。
2. **cefsimple**: 我们的cefsimple工程。
3. **libcef_dll_wrapper**: libcef相关支持。
4. **ZERO_CHECK**: cmake相关。

## 5. 替换掉cefsimple里面的所有.h、.cc文件里的 test/
由于我们移动了cefsimple工程的位置 ，所以 需要修改所有.h及.cpp相关#include"test/****"中的test/删掉，以保证#include正确。
此时再编译运行就可以了。
最终源码可查看github上cefsimple项目[https://github.com/iherewaitfor/cefsimple](https://github.com/iherewaitfor/cefsimple)
## 总结
如果要添加自己的工程，可以参考 cefsimple，修改相关的CMakeList，就可以了。
