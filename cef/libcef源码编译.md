# 从源码编译 libcef.dll，32位
  - 编译特定版本的，比如3202版本。
      - 要求3202	62	Win 7+, VS2015u3, Win10.0.14393 SDK, Ninja
      - 各版本的系统和开发环境要求，请参考https://bitbucket.org/chromiumembedded/cef/wiki/BranchesAndBuilding
  - 开发环境要求：
     1. 安装python，注意是32位的2.7.x版本的python。
     1. 安装vs2015u3
        - 注意，最好使用默认位置安装。
        - 本机安装位置 D:\Program Files (x86)\Microsoft Visual Studio 14.0
     1. 安装windowsdk 10.0.14393
        - 注意，最好使用默认位置安装。
        - 本机安装位置 C:\Program Files (x86)\Windows Kits\
     1. 安装git，注意要安装较新版本。
        - 本机版本：git version 2.18.0.windows.1
     1. 请预留100GB磁盘空间。
  - 见官方构建文档
     - https://bitbucket.org/chromiumembedded/cef/wiki/MasterBuildQuickStart.md
     - 官方文档这个讲的是主干的构建，构建特定分支，需要做一些参数上的调整
        - 分支的构建有一些不同，注意事项
            - 请参见https://bitbucket.org/chromiumembedded/cef/wiki/BranchesAndBuilding里面的Build Notes
        - 比如--branch  3202
        - 比如--ide=vs2015
        - 比如 VS2015 set GN_DEFINES=is_win_fastlink=true for improved compile and link time.
# 手动构建步骤
  - 参考https://bitbucket.org/chromiumembedded/cef/wiki/MasterBuildQuickStart.md
  1. Create the following directories.
     - 本机的目录结构
     ```
     E:\libcef\automate
     E:\libcef\chromium_git
     ```
  2. 下载[depot_tools.zip](https://storage.googleapis.com/chrome-infra/depot_tools.zip),并解压到"E:\libcef\depot_tools"
     - 下载地址https://storage.googleapis.com/chrome-infra/depot_tools.zip
     - 注意，需要直接解压到文件夹"E:\libcef\depot_tools"，不要复制粘贴、拖动。因为可能丢失.git信息。
     - 可以考虑使用[7z](http://www.7-zip.org/download.html)去解压
  3. Run "update_depot_tools.bat" to install Python, Git and SVN.
     - 该命令会去更新该工具，并安装后面用到的Python。（2.7.6版本）、git等。
     ```
     cd E:\libcef\depot_tools
     update_depot_tools.bat
     ```
  4.  将"E:\libcef\depot_tools" 文件夹添加到环境变量PATH中
     - 在命令行中输入命令SystemPropertiesAdvanced。或者右键“计算机”-->点击属性-->点击高级系统设置
     - 点击环境变量按钮
     - 双击Path，并编辑，将"E:\libcef\depot_tools" 添加进行，
  5. Download the [automate-git.py](https://bitbucket.org/chromiumembedded/cef/raw/master/tools/automate/automate-git.py) script to "E:\libcef\automate\automate-git.py".
  6. Create the "E:\libcef\chromium_git\update.bat" script with the following contents.
     ```
     set DEPOT_TOOLS_UPDATE=0
     set GN_DEFINES=is_win_fastlink=true
     set GN_ARGUMENTS=--ide=vs2015 --sln=cef --filters=//cef/*
     python ..\automate\automate-git.py --download-dir=E:\libcef\chromium_git --depot-tools-dir=E:\libcef\depot_tools --no-distrib --no-build   --branch=3202
     ```
     - 设置git的配置项。以保证gi下载源码成功。
      ```
        git config --global core.compression 0
        git config --global http.postBuffer 524288000
      ```
     - 运行 “update.bat"，等待其下载cef及chromium的源码。cef源码会下载到"E:\libcef\cef"，chromium的源码会下载到"E:\libcef\chromium_git\chromium\src"。下完之后，cef的源码会被复制到"E:\libcef\chromium_git\chromium\src\cef"
     ```
     cd E:\libcef\chromium_git
     update.bat
     ```
  7. 创建脚本 文件"E:\libcef\chromium_git\chromium\src\cef\create.bat" ，并将以下内容保存到该脚本。
      ```
      set GN_DEFINES=is_win_fastlink=true
      set GN_ARGUMENTS=--ide=vs2015 --sln=cef --filters=//cef/*
      call cef_create_projects.bat
      ```
      - 运行脚本"create.bat" 生成 Ninja 和 Visual Studio 工程文件.
      ```
      cd E:\libcef\chromium_git\chromium\src\cef
      create.bat
      ```
      - 这个脚本会生成文件"E:\libcef\chromium_git\chromium\src\out\Debug_GN_x86\cef.sln" 。
      - This will generate a "c:\code\chromium_git\chromium\src\out\Debug_GN_x86\cef.sln" file that can be loaded in Visual Studio for debugging and compiling individual files. Replace “x86” with “x64” in this path to work with the 64-bit build instead of the 32-bit build. Always use Ninja to build the complete project. Repeat this step if you change the project configuration or add/remove files in the GN configuration (BUILD.gn file).
    8. Create a Debug build of CEF/Chromium using Ninja. Replace “x86” with “x64” in the below example to generate a 64-bit build instead of a 32-bit build. Edit the CEF source code at "c:\code\chromium_git\chromium\src\cef" and repeat this step multiple times to perform incremental builds while developing.
        ```
        cd E:\libcef\chromium_git\chromium\src
        ninja -C out\Debug_GN_x86 cef
        ```
        - release 版本，则执行
         ```
        cd E:\libcef\chromium_git\chromium\src
        ninja -C out\Release_GN_x86 cef
        ```  
    9. Run the resulting cefclient sample application.
       ```
       cd E:\libcef\chromium_git\chromium\src
       out\Debug_GN_x86\cefclient.exe
       ```
    10. 打包
        ```
        cd E:\libcef\chromium\src\cef\tools
        make_distrib.bat --ninja-build
        ```
        - 参考Manual Packaging部分。 https://bitbucket.org/chromiumembedded/cef/wiki/BranchesAndBuilding.md#markdown-header-release-branches
       

# 碰到问题解决
 - git下载chrome源码问题
    ```
    remote: internal server error
    fatal: early EOF
    fatal: index-pack failed
    ```
    ```
    F:\chromesrc2>git clone --progress https://chromium.googlesource.com/chromium/src
    Cloning into 'src'...
    remote: Sending approximately 10.83 GiB ...
    remote: Counting objects: 110796, done
    remote: Finding sources: 100% (140/140)
    error: RPC failed; curl 18 transfer closed with outstanding read 
    data     remaining
    ffatal: The remote end hung up unexpectedly
    atal: early EOF
    fatal: index-pack failed
    
    F:\chromesrc2>
    ```
    - 解决 
        - 使用如下配置
        ```
        core.compression=0
        http.postbuffer=524288000
        ```
        ```
        git config --global core.compression 0
        git config --global http.postBuffer 524288000
        ```
    - 参考
        - https://zhuanlan.zhihu.com/p/24911872
- gclient错误
    ```
    E:\libcef\chromium_git>update.bat > out4.txt
    remote: Counting objects: 18005, done
    remote: Finding sources: 100% (589/589)
    remote: Total 589 (delta 131), reused 502 (delta 131)
    Receiving objects: 100% (589/589), 3.19 MiB | 4.91 MiB/s, done.
    Resolving deltas: 100% (131/131), completed with 67 local objects.
    From https://chromium.googlesource.com/chromium/src
       cefbc0542808..11e0af717de3  lkgr                   -> origin/lkgr
       562fb11a3c10..45116fe1dd62  master                 -> origin/master
       28036bba9b31..1e1ed00fd27d  refs/branch-heads/3497 -> branch-heads/3497
       3413c1e63f82..b45f0ee4c1a5  refs/branch-heads/3534 -> branch-heads/3534
    remote: Counting objects: 69218, done
    remote: Finding sources: 100% (7/7)
    remote: Total 7 (delta 0), reused 5 (delta 0)
    Unpacking objects: 100% (7/7), done.
    From https://chromium.googlesource.com/chromium/src
       45116fe1dd62..233a3244cd0d  master     -> origin/master
    Checking out files: 100% (207312/207312), done.
    Previous HEAD position was 562fb11a3c10 Replace erase(std::remove()) to     base::Erase() in media
    HEAD is now at 937db09514e0 Publish DEPS for Chromium 62.0.3202.94
    Error: Command 'E:\\libcef\\depot_tools\\win_tools-2_7_6_bin\\python\\bin\\py    thon.exe src/build/get_syzygy_binaries.py     --output-dir=src/third_party/syzygy/binaries     --revision=190dbfe74c6f5b5913820fa66d9176877924d7c5 --overwrite     --copy-dia-binaries' returned non-zero exit status 1 in     E:\libcef\chromium_git\chromium
    Traceback (most recent call last):
      File "..\automate\automate-git.py", line 986, in <module>
        chromium_dir, depot_tools_dir)
      File "..\automate\automate-git.py", line 55, in run
        args, cwd=working_dir, env=env, shell=(sys.platform == 'win32'))
      File "C:\Python27\lib\subprocess.py", line 504, in check_call
        raise CalledProcessError(retcode, cmd)
    subprocess.CalledProcessError: Command '['gclient', 'sync',     '--with_branch_heads', '--disable-syntax-validation', '--jobs', '16']'     returned non-zero exit status 2
    
    E:\libcef\chromium_git>u
    ```
    - 解决
        - 升级git至最新版本。
        - 请确保E:\libcef\depot_tools，在环境变量中。
            - 运行echo %path%
            - 确保从输出中，能看到E:\libcef\depot_tools
        - 确保E:\libcef\depot_tools在环境变量中，再运行update.bat
        - 失败，可重试运行updat.bat脚本。
    - 参考
- node.exe找不到
    - 执行 ninja -C out\Release_GN_x86 cef时，报如下错误 
    ```
    ninja: Entering directory `out\Release_GN_x86'
    [1/30497] STAMP obj/components/toolbar/toolbar_vector_icons.stamp
    [2/30497] ACTION //chrome/browser/resources/md_bookmarks:build(//build/toolc    hain/win:x86)
    FAILED: gen/chrome/browser/resources/md_bookmarks/vulcanized.html     gen/chrome/browser/resources/md_bookmarks/crisper.js 
    E:/libcef/depot_tools/win_tools-2_7_6_bin/python/bin/python.exe     ../../chrome/browser/resources/vulcanize_gn.py --host bookmarks --input     ../../chrome/browser/resources/md_bookmarks --out_folder     gen/chrome/browser/resources/md_bookmarks --depfile     gen/chrome/browser/resources/md_bookmarks/build.d --html_in_files     bookmarks.html --html_out_files vulcanized.html --js_out_files crisper.js
    ..\..\third_party\node\win\node.exe     ..\..\third_party\node\node_modules\polymer-bundler\lib\bin\polymer-bundler     --exclude chrome://resources/html/polymer.html --exclude     chrome://resources/polymer/v1_0/polymer/polymer.html --exclude     chrome://resources/polymer/v1_0/polymer/polymer-micro.html --exclude     chrome://resources/polymer/v1_0/polymer/polymer-mini.html --exclude     chrome://resources/polymer/v1_0/web-animations-js/web-animations-next-lite.m    in.js --exclude chrome://resources/css/roboto.css --exclude     chrome://resources/css/text_defaults.css --exclude     chrome://resources/css/text_defaults_md.css --exclude     chrome://resources/js/load_time_data.js --inline-css --inline-scripts     --rewrite-urls-in-templates --strip-comments --redirect     "chrome://resources/cr_components/|..\..\ui\webui\resources\cr_components"     --redirect "chrome://resources/cr_elements/|..\..\ui\webui\resources\cr_elem    ents" --redirect "chrome://resources/css/|..\..\ui\webui\resources\css"     --redirect "chrome://resources/html/|..\..\ui\webui\resources\html"     --redirect "chrome://resources/js/|..\..\ui\webui\resources\js" --redirect     "chrome://resources/polymer/v1_0/|..\..\third_party\polymer\v1_0\components-    chromium" --exclude strings.js --exclude chrome://bookmarks/strings.js     --manifest-out E:\libcef\chromium_git\chromium\src\out\Release_GN_x86\gen\ch    rome\browser\resources\md_bookmarks\bookmarks_requestlist.txt --root     E:\libcef\chromium_git\chromium\src\chrome\browser\resources\md_bookmarks     --redirect "chrome://bookmarks/|E:\libcef\chromium_git\chromium\src\chrome\b    rowser\resources\md_bookmarks" --out-dir     gen\chrome\browser\resources\md_bookmarks\bundled --shell bookmarks.html     --in-html bookmarks.html failed: '..\..\third_party\node\win\node.exe'     不是内部或外部命令，也不是可运行的程序或批处理文件。
    ```
    - 解决
        - 直接复制 你本地的node.exe到该目录"E:\libcef\chromium_git\chromium\src\third_party\node\win"
        - 然后，又会碰到缺少polymer-bundler等 npm包
        - 直接切到目录“E:\libcef\chromium_git\chromium\src\third_party\node”，然后缺少什么安装什么npm install polymer-bundler
        ```
        cd E:\libcef\chromium_git\chromium\src\third_party\node
        npm install polymer-bundler
        ```
- windows sdk 10.0.14393.0安装
    - 使用默认安装，安装至C:\Program Files (x86)\Windows Kits
- failed to find vcvars
   - 解决set CEF_VCVARS=D:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\bin\vcvars32.bat
   ```
   E:\libcef\chromium_git\chromium\src\cef\tools>set CEF_VCVARS=D:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\bin\vcvars32.bat
   ```
   - 原因，本机的vs2015没有使用默认安装，msvs_env.bat脚本 没找到vcvars 
      - 查看脚本E:\libcef\chromium_git\chromium\src\cef\tools\msvs_env.bat，可以看到vcvars，其使用CEF_VCVARS环境变量。并且会去vs的默认安装位置去找。而本地，即没设置 CEF_VCVARS环境变量，vs2015（Microsoft Visual Studio 14.0）又没安装在默认位置。
      ```
      :: In case vcvars is already provided via the environment.
      set vcvars="%CEF_VCVARS%"
      if exist %vcvars% goto found_vcvars
      if %vcvars% == "none" goto found_vcvars
      
      if "%1" == "win64" goto check_win64
      
      :: Hardcoded list of MSVS paths for VS2017 32-bit builds.
      set vcvars="%PROGRAMFILES(X86)%\Microsoft Visual       Studio\2017\Professional\VC\Auxiliary\Build\vcvars32.bat"
      if exist %vcvars% goto found_vcvars
      set vcvars="%PROGRAMFILES%\Microsoft Visual       Studio\2017\Professional\VC\Auxiliary\Build\vcvars32.bat"
      if exist %vcvars% goto found_vcvars
      
      :: Hardcoded list of MSVS paths for VS2015 32-bit builds.
      set vcvars="%PROGRAMFILES(X86)%\Microsoft Visual Studio       14.0\VC\bin\vcvars32.bat"
      if exist %vcvars% goto found_vcvars
      set vcvars="%PROGRAMFILES%\Microsoft Visual Studio 14.0\VC\bin\vcvars32.bat"
      if exist %vcvars% goto found_vcvars
      ```
