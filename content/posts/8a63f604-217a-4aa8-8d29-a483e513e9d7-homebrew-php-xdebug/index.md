---
title: "homebrew  安裝 php 再安裝 Xdebug 出錯 "
date: "2023-10-05T07:59:02Z"
tags: [PHP]
---

# homebrew  安裝 php 再安裝 Xdebug 出錯 

## 起因

Mac 安裝 PHP 的 Xdebug 時會噴錯：

```Bash
pecl install xdebug
```

```Go
ERROR: failed to mkdir /opt/homebrew/Cellar/php@8.1/8.1.21/pecl/20210902
```

{{< br >}}

發現 `/opt/homebrew/Cellar/php@8.1/8.1.21/pecl` 指向空的資料夾

```Go
pecl -> /opt/homebrew/lib/php/pecl
```

{{< br >}}

## 解方

參考 github 上的解法，我只做了 `postinstall` 沒做 `reinstall`：

```Bash
brew postinstall php@8.1
```

{{< br >}}

## 參考資料

[Installing Xdebug 3 on MacOS and Debug in VS Code](https://dev.to/scriptmint/installing-xdebug-3-on-macos-and-debug-in-vs-code-3l5h)