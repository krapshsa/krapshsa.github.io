---
title: "WSL 的 cypress 白畫面"
date: "2023-10-04T07:47:57Z"
tags: [javascript,cypress]
---

# WSL 的 cypress 白畫面

## 起因

`WSL` 裡面 clone 了 source code，包含 cypress 寫的 e2e test，

執行起來雖然有視窗跳出來 (`WSL` 的顯示 Linux GUI 功能正常)，但是畫面一片空白。

{{< br >}}

## 解方

我用的是 ubuntu in WSL，參照官方的教學需要安裝一些額外套件：

```JavaScript
apt-get install libgtk2.0-0 libgtk-3-0 libgbm-dev libnotify-dev libgconf-2-4 libnss3 libxss1 libasound2 libxtst6 xauth xvfb
```

其他環境可以參考

[Installing Cypress | Cypress Documentation](https://docs.cypress.io/guides/getting-started/installing-cypress)

{{< br >}}

並且 Node 要是 18 或 20+，但是 ubuntu 預設 `apt` 裝起來的是 12

1. 清理舊的 `nodejs`

    ```Bash
    sudo apt remove nodejs
    ```
2. 安裝 `nvm`

    ```Bash
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.5/install.sh | bash
    ```

    最新請參考：

    [GitHub - nvm-sh/nvm: Node Version Manager - POSIX-compliant bash script to manage multiple active node.js versions](https://github.com/nvm-sh/nvm#installing-and-updating)
3. 安裝 `nodejs` (想要什麼版本自己填)

    ```Bash
    nvm install 18
    ```
4. 如果有裝多個版本可以切換

    ```Bash
    nvm use 18
    ```
5. 確認版本

    ```Bash
    node -v
    ```