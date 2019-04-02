# libcef文档1--CEF入门：cef windows 示例工程编译运行
要对libcef有一个直观的了解，快速的办法之一就是将其示例项目跑起来。下文介绍如何libcef的示例项目CefClient及CefSimple跑起来。[Libecef主页https://bitbucket.org/chromiumembedded/cef/overview](https://bitbucket.org/chromiumembedded/cef/overview)。其中概念性描述，请查看[https://bitbucket.org/chromiumembedded/cef/wiki/GeneralUsage](https://bitbucket.org/chromiumembedded/cef/wiki/GeneralUsage)
# 1. 环境准备
- Visual Studio
   - 本文使用VS2015。  Version 14.0.25431.01 Update3
   - ![image](https://note.youdao.com/yws/api/personal/file/D66FC253BACD49ED989B55F34DB6D894?method=download&shareKey=108911bf87a5a47a7170b55c5cd43622)
- 下载相关Cmake
   - [Cmake下载地址https://cmake.org/download/](https://cmake.org/download/)
   - 本文下载cmake-3.12.2-win32-x86.zip
- 下载libcef二进制版本
   - [libcef二进制版本下载地址http://opensource.spotify.com/cefbuilds/index.html](http://opensource.spotify.com/cefbuilds/index.html)
   - 本文下载的是cef_binary_3.3626.1895.g7001d56_windows32.tar.bz2, 与chrome72版本对应。
   - [libcef版本与chrome版本的对应关系，请查看https://bitbucket.org/chromiumembedded/cef/wiki/BranchesAndBuilding](https://bitbucket.org/chromiumembedded/cef/wiki/BranchesAndBuilding)
# 2. 解压Cmake及Libcef
  - 将cef及cmake解压至目录J:\ceftest
  - ![image](https://note.youdao.com/yws/api/personal/file/B2FD176B94584A50BB18414C132986DA?method=download&shareKey=b997c4a1c7e7a4ef5efee5991b6789f1)
# 3.使用cmake  配置及生成libcef示例工程
  -  双击运行cmake-gui
      -  路径为
      ```
      J:\ceftest\cmake-3.12.2-win32-x86\bin\cmake-gui.exe
      ```
  - 设置相关路径
      - 源码路径     J:/cef/test/cef_3626
      - 项目的生成文件的路径  J:/ceftest/cef_3626_vs2015
      - ![image](https://note.youdao.com/yws/api/personal/file/957DB6943E19429596195557790948B3?method=download&shareKey=7aa88f91f547725df1ac6efb457a6f80)
  - 配置configure
      - 占击 **Configure**按钮，并选择Visual Studio 14 2015 用于生成示例项目
      - ![image](https://note.youdao.com/yws/api/personal/file/F5117247D8824F81B6AFDCB77B2AC056?method=download&shareKey=7dfc9fc313020b5cd0b7694acfc0cc36)
  - 生成Generate
     - 点击**Generate**按钮，生成解决方案。
     - 打开路径J:/ceftest/cef_3626_vs2015就能看到看成的解决方案的相关文件。双击cef.sln文件（默认选vs2015打开.sln）即可打开解决方案。
     - cef.sln的解决方案里面会包括所有的示例工程。其中包括cefclient及cefsimple。后续可深入理解这两个例子。
     - ![image](https://note.youdao.com/yws/api/personal/file/4575AF2D8CC14EB19BAD2214F9A2205A?method=download&shareKey=93e2356745d5230853b54b4b47e8400d)
     - ![cef.sln解决方案文件列表](https://note.youdao.com/yws/api/personal/file/36F9B931E1834F35AF54CCC96CF892F5?method=download&shareKey=f554e8c3a3bee92e8a00d176c0d4dde7 "cef.sln解决方案文件列表")
     - ![cef.sln解决方案项目列表](https://note.youdao.com/yws/api/personal/file/1DB7FB5C5C89420F8728078A85D97968?method=download&shareKey=2f1d6a5486818f50ae4211df3357a982 "cef.sln解决方案项目列表")
# 4. 编译运行cefsimple
  - 在cefsimple项目上右键-->Debug-->Start new instance
      - ![启动cefsimple](https://note.youdao.com/yws/api/personal/file/258F2F5F4D5E432E8F7DB75CA113C6D9?method=download&shareKey=767257fc876459ef63c799372642c87a "启动cefsimple")
      - ![cefsimple](https://note.youdao.com/yws/api/personal/file/A0E26D99105848189F3103650F533360?method=download&shareKey=e75b3af5058452cae802b151d948586b "cefsimple")
  - cefsimple程序入口源码位置cefsimple_win.cc
     - 核心是要起码要实现CefApp、CefBrowserProcessHandler等回调。详细后续会开篇专门讲概念和程序结构。
  ```
  // Copyright (c) 2013 The Chromium Embedded Framework Authors. All rights
  // reserved. Use of this source code is governed by a BSD-style license that
  // can be found in the LICENSE file.
  
  #include <windows.h>
  #include "include/cef_sandbox_win.h"
  #include "tests/cefsimple/simple_app.h"
  
  // When generating projects with CMake the CEF_USE_SANDBOX value will be defined
  // automatically if using the required compiler version. Pass -DUSE_SANDBOX=OFF
  // to the CMake command-line to disable use of the sandbox.
  // Uncomment this line to manually enable sandbox support.
  // #define CEF_USE_SANDBOX 1
  
  #if defined(CEF_USE_SANDBOX)
  // The cef_sandbox.lib static library may not link successfully with all VS
  // versions.
  #pragma comment(lib, "cef_sandbox.lib")
  #endif
  
  // Entry point function for all processes.
  int APIENTRY wWinMain(HINSTANCE hInstance,
                        HINSTANCE hPrevInstance,
                        LPTSTR lpCmdLine,
                        int nCmdShow) {
    UNREFERENCED_PARAMETER(hPrevInstance);
    UNREFERENCED_PARAMETER(lpCmdLine);
  
    // Enable High-DPI support on Windows 7 or newer.
    CefEnableHighDPISupport();
  
    void* sandbox_info = NULL;
  
  #if defined(CEF_USE_SANDBOX)
    // Manage the life span of the sandbox information object. This is necessary
    // for sandbox support on Windows. See cef_sandbox_win.h for complete details.
    CefScopedSandboxInfo scoped_sandbox;
    sandbox_info = scoped_sandbox.sandbox_info();
  #endif
  
    // Provide CEF with command-line arguments.
    CefMainArgs main_args(hInstance);
  
    // CEF applications have multiple sub-processes (render, plugin, GPU, etc)
    // that share the same executable. This function checks the command-line and,
    // if this is a sub-process, executes the appropriate logic.
    int exit_code = CefExecuteProcess(main_args, NULL, sandbox_info);
    if (exit_code >= 0) {
      // The sub-process has completed so return here.
      return exit_code;
    }
  
    // Specify CEF global settings here.
    CefSettings settings;
  
  #if !defined(CEF_USE_SANDBOX)
    settings.no_sandbox = true;
  #endif
  
    // SimpleApp implements application-level callbacks for the browser process.
    // It will create the first browser instance in OnContextInitialized() after
    // CEF has initialized.
    CefRefPtr<SimpleApp> app(new SimpleApp);
  
    // Initialize CEF.
    CefInitialize(main_args, settings, app.get(), sandbox_info);
  
    // Run the CEF message loop. This will block until CefQuitMessageLoop() is
    // called.
    CefRunMessageLoop();
  
    // Shut down CEF.
    CefShutdown();
  
    return 0;
  }

  ```
# 5. 编译运行 cefclient
  - 在 cefclient 项目上右键-->Debug-->Start new instance
  - 会碰到 c2220错误，warning treated as error。
     - warning内容如下
     ```
     1>J:\ceftest\cef_3626\tests\cefclient\browser\browser_window_std_win.cc(68): error C2220: warning treated as error - no 'object' file generated
     1>J:\ceftest\cef_3626\tests\cefclient\browser\browser_window_std_win.cc(68): warning C4800: 'LONG': forcing value to bool 'true' or 'false' (performance warning)
     ```
     - ![错误](https://note.youdao.com/yws/api/personal/file/74012070421940CAAEE506A8BAFB440F?method=download&shareKey=c1013e0c208a7a276be7b410a96b435a)
  - **解决办法**：可以选择以下两种方式之一解决
     - 解决一：直接修改源码，消除告警。
        - 问题源码：:\ceftest\cef_3626\tests\cefclient\browser\browser_window_std_win.cc文件68行。
        ```C++
        const bool no_activate =
             GetWindowLongPtr(parent_handle, GWL_EXSTYLE) & WS_EX_NOACTIVATE;
        ```
        - 修复为如下 。
        ```
        const bool no_activate =
             GetWindowLongPtr(parent_handle, GWL_EXSTYLE) & WS_EX_NOACTIVATE ? true : false;
        ```
     - 解决二：修改工程设置，把warning treated as error关掉
        - 右键cefclient-->properties。在 Configuration-->C++-->General界面的Treat Warning as Error下拉框选择No
        - 重新编译运行即可。
        - ![设置](https://note.youdao.com/yws/api/personal/file/5E53FB46431C4CEE84BAAFC8888F6B03?method=download&shareKey=0cdcc4d740c7227d837f288f5b1e6403)
     