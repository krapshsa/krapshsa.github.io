---
title: "Mac Build Nextcloud Desktop"
date: "2023-04-24T02:31:08Z"
tags: [Nextcloud]
---

# Mac Build Nextcloud Desktop

---

# 前言

照官方的指引，沒有辦法 Build 起來，在此紀錄踩過的坑。

{{< br >}}

# 官方文件

[Appendix A: Building the Client — Nextcloud Client Manual 3.8.1 documentation](https://docs.nextcloud.com/desktop/3.8/building.html#macos-development-build)

[System requirements for compiling the desktop client](https://github.com/nextcloud/desktop/wiki/System-requirements-for-compiling-the-desktop-client)

{{< br >}}

# 坑

## `qtkeychain` 依賴 qt6

`Nextcloud` 需要用 qt5 去 build，但是用 `brew` 安裝 `qtkeychain` 的話，

最新版本 bottle 內只有 `libqt6keychain.dylib` 而沒有 `libqt5keychain.dylib` 。

{{< br >}}

### 解法

從 `github` 上面抓舊版本的 `qtkeychain.rb` 來用可以解決：

[homebrew-core/qtkeychain.rb at 11917de6acdc546c28dcf36652f5467c47136a79 · Homebrew/homebrew-core](https://github.com/Homebrew/homebrew-core/blob/11917de6acdc546c28dcf36652f5467c47136a79/Formula/qtkeychain.rb)

{{< br >}}

下載之後安裝：

    brew install qtkeychain.rb

{{< br >}}

{{< br >}}

## openssl 需要 1.1 版

### 解法ㄧ

這個在 github 有寫到，但是官網關於環境變數的文件寫錯了：

`.nextcloud_build_variables` 

官網的：

    export OPENSSL_ROOT_DIR=$(brew --prefix openssl)

要改成：

    export OPENSSL_ROOT_DIR=$(brew --prefix openssl@1.1)

{{< br >}}

### 解法二

但是我不喜歡第一個做法，所以我這樣做：

`cmake` 加上參數

    -DOPENSSL_ROOT_DIR=$(brew --prefix openssl@1.1)

完整指令 (在 build 資料夾內下)

    cmake -DOPENSSL_ROOT_DIR=$(brew --prefix openssl@1.1) -DCMAKE_PREFIX_PATH=/opt/homebrew/opt/qt@5/lib/cmake -DCMAKE_BUILD_TYPE="Debug" ..

{{< br >}}

{{< br >}}

- 問題四：inkscape
    - brew install inkscape

    {{< br >}}

{{< br >}}

## Qt5Core 找不到 `macx-clang`，Qt5Network 找不到 `libqgenericbearer.dylib`，Qt5Gui 找不到 `libqcocoa.dylib` 等

### 錯誤訊息

    CMake Error at /opt/homebrew/lib/cmake/Qt5Core/Qt5CoreConfig.cmake:14 (message):
      The imported target "Qt5::Core" references the file
    
         "/opt/homebrew/.//mkspecs/macx-clang"
    
      but this file does not exist.  Possible reasons include:
    
      * The file was deleted, renamed, or moved to another location.
    
      * An install or uninstall procedure did not complete successfully.
    
      * The installation package was faulty and contained
    
         "/opt/homebrew/lib/cmake/Qt5Core/Qt5CoreConfigExtras.cmake"
    
      but not all the files it references.
    
    Call Stack (most recent call first):
      /opt/homebrew/lib/cmake/Qt5Core/Qt5CoreConfigExtras.cmake:56 (_qt5_Core_check_file_exists)
      /opt/homebrew/lib/cmake/Qt5Core/Qt5CoreConfig.cmake:227 (include)
      src/CMakeLists.txt:9 (find_package)
    

    CMake Error at /opt/homebrew/lib/cmake/Qt5Network/Qt5NetworkConfig.cmake:14 (message):
      The imported target "Qt5::Network" references the file
    
         "/opt/homebrew/plugins/bearer/libqgenericbearer.dylib"
    
      but this file does not exist.  Possible reasons include:
    
      * The file was deleted, renamed, or moved to another location.
    
      * An install or uninstall procedure did not complete successfully.
    
      * The installation package was faulty and contained
    
         "/opt/homebrew/lib/cmake/Qt5Network/Qt5Network_QGenericEnginePlugin.cmake"
    
      but not all the files it references.
    
    Call Stack (most recent call first):
      /opt/homebrew/lib/cmake/Qt5Network/Qt5NetworkConfig.cmake:214 (_qt5_Network_check_file_exists)
      /opt/homebrew/lib/cmake/Qt5Network/Qt5Network_QGenericEnginePlugin.cmake:5 (_populate_Network_plugin_properties)
      /opt/homebrew/lib/cmake/Qt5Network/Qt5NetworkConfig.cmake:223 (include)
      src/CMakeLists.txt:15 (find_package)
    

    CMake Error at /opt/homebrew/lib/cmake/Qt5Gui/Qt5GuiConfig.cmake:14 (message):
      The imported target "Qt5::Gui" references the file
    
         "/opt/homebrew/plugins/platforms/libqcocoa.dylib"
    
      but this file does not exist.  Possible reasons include:
    
      * The file was deleted, renamed, or moved to another location.
    
      * An install or uninstall procedure did not complete successfully.
    
      * The installation package was faulty and contained
    
         "/opt/homebrew/lib/cmake/Qt5Gui/Qt5Gui_QCocoaIntegrationPlugin.cmake"
    
      but not all the files it references.
    
    Call Stack (most recent call first):
      /opt/homebrew/lib/cmake/Qt5Gui/Qt5GuiConfig.cmake:214 (_qt5_Gui_check_file_exists)
      /opt/homebrew/lib/cmake/Qt5Gui/Qt5Gui_QCocoaIntegrationPlugin.cmake:5 (_populate_Gui_plugin_properties)
      /opt/homebrew/lib/cmake/Qt5Gui/Qt5GuiConfig.cmake:223 (include)
      /opt/homebrew/lib/cmake/Qt5Quick/Qt5QuickConfig.cmake:93 (find_package)
      /opt/homebrew/lib/cmake/Qt5WebEngineCore/Qt5WebEngineCoreConfig.cmake:93 (find_package)
      /opt/homebrew/lib/cmake/Qt5WebEngineWidgets/Qt5WebEngineWidgetsConfig.cmake:93 (find_package)
      src/CMakeLists.txt:33 (find_package)

{{< br >}}

### 解法

`cmake` 加上參數

    -DCMAKE_PREFIX_PATH=$(brew --prefix qt5)/lib/cmake

完整指令 (在 build 資料夾內下)

    cmake -DOPENSSL_ROOT_DIR=$(brew --prefix openssl@1.1) -DCMAKE_PREFIX_PATH=/opt/homebrew/opt/qt@5/lib/cmake -DCMAKE_BUILD_TYPE="Debug" ..

{{< br >}}

## 找不到 `KF5Archive`

官方沒說要裝這個，但 Mac 版好像真用不到這個模組？

也許可以參照 `argp.h` 的處理方式，讓她跳過，但是我選擇先不改他的 `CMakeLists.txt`

{{< br >}}

### 錯誤訊息

    CMake Error at src/gui/CMakeLists.txt:3 (find_package):
      By not providing "FindKF5Archive.cmake" in CMAKE_MODULE_PATH this project
      has asked CMake to find a package configuration file provided by
      "KF5Archive", but CMake did not find one.
    
      Could not find a package configuration file provided by "KF5Archive" with
      any of the following names:
    
        KF5ArchiveConfig.cmake
        kf5archive-config.cmake
    
      Add the installation prefix of "KF5Archive" to CMAKE_PREFIX_PATH or set
      "KF5Archive_DIR" to a directory containing one of the above files.  If
      "KF5Archive" provides a separate development package or SDK, be sure it has
      been installed.

{{< br >}}

### 解法

    brew install karchive

{{< br >}}

## 缺少轉 Icon 的軟體

### 錯誤訊息

    CMake Error at cmake/modules/GenerateIconsUtils.cmake:6 (find_program):
      Could not find SVG_CONVERTER using the following names: inkscape,
      inkscape.exe, rsvg-convert
    Call Stack (most recent call first):
      src/gui/CMakeLists.txt:366 (include)

{{< br >}}

### 解法

我目前是用 `inkscape` 這個軟體處理，但是他會在圖形介面上開啟，轉完後自動關閉。
在 SSH 登入的情況下能不能動不知道。

    brew install inkscape

{{< br >}}