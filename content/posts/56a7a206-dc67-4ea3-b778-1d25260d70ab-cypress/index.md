---
title: "我瞎了 Cypress 沒瞎"
date: "2023-09-27T10:13:20Z"
tags: [javascript,cypress]
---

# 我瞎了 Cypress 沒瞎

## 起因

頁面上有一個 Radio Button，在錄製操作的時候他幫我寫了這一段：

```JavaScript
cy.get('#raido_button_1').click();
```

但是重播操作的時候他會跟我說 `#radio_button_1` 是 invisible，腳本就中止了。

加上強制觸發一切又會正常：

```JavaScript
cy.get('#raido_button_1').click({force: true});
```

{{< br >}}

類似的錯：

[The following error originated from your application code, not from Cypress](https://stackoverflow.com/questions/62980435/the-following-error-originated-from-your-application-code-not-from-cypress)

{{< br >}}

原來是因為公司的 radio button 已經統一調整成 input 是 `display: none`

由偽元素來呈現效果的做法了：

[https://www.youtube.com/watch?v=5K7JefKDa4s](https://www.youtube.com/watch?v=5K7JefKDa4s)

{{< br >}}

錄製之後還是要重試一下再把測試加回 source control。