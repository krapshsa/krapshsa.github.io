---
title: "Cypress 錄製操作與批次執行"
date: "2023-10-04T07:45:51Z"
tags: [javascript,cypress]
---

# Cypress 錄製操作與批次執行

## `cypress.config.js`

```JavaScript
const { defineConfig } = require("cypress");

module.exports = defineConfig({
  e2e: {
    setupNodeEvents(on, config) {
      // implement node event listeners here
    },
    experimentalStudio: true, // 加上這行支援錄製操作
    experimentalRunAllSpecs: true, // 加上這行支援批次執行
  },
});
```

{{< br >}}

這是到今天為止的設定方式，cypress 更新的非常快，
很多掛 `experimental` 前綴的功能過一陣子就是預設的了。

可以看這邊知道最新版有哪些試驗功能：

[Experiments | Cypress Documentation](https://docs.cypress.io/guides/references/experiments#Testing-Type-Specific-Experiments)

網路上教學的試驗功能設定找不到了可以看 Changelog 是否已經收入標準：

[Changelog | Cypress Documentation](https://docs.cypress.io/guides/references/changelog)

{{< br >}}

## 錄製操作

新增或打開一個 Specs，如果有開啟設定之後會多出一個 `Add Commands to Test`，點下去就會開始錄製。

比較麻煩的是他會重跑整個測試，有時候我前置作業先寫好一點了他就會重複執行。

![](Screenshot_2023-10-04_at_4-df274fc7-4da7-41a1-8c6b-0a86448f4f29.21.22_PM.png)

`Save Command` 之後他就會自動幫你加進腳本中：

```JavaScript

```

## 批次執行

可以用數字當作 prefix 來確保執行的順序，有點像 systemd 那種感覺。

批次執行的對象可以是上層資料夾，也可以是下層資料夾，只要滑鼠滑到名稱旁邊就會出現批次的選項。

![](Screenshot_2023-10-04_at_4-1d10a6e6-6b91-4f60-baa5-3b11cecbeb22.30.23_PM.png)