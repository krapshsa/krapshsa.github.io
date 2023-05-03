---
title: "在 template literal 中使用函數"
date: "2022-06-21T08:18:00Z"
tags: [javascript]
---

# 在 template literal 中使用函數

## 起因

在 Code Review 的時候，同事原本是使用多次字串相接

    const object1 = {
      a: 'somestring',
      b: 42
    };
    
    let html = `<ul>`;
    
    
    for (const [key, value] of Object.entries(object1)) {
      html += `<li>${key}: ${value}</li>`;
    }
    
    html += `</ul>`;

後來想要表達出 html 的階層結構，想用使用巢狀的方式使用 template literal

{{< br >}}

## 解方

要用 function 來做的話可以使用 IIFE

    const object1 = {
      a: 'somestring',
      b: 42
    };
    
    let html = `
    <ul>
      ${
        (function(){
          let list = '';
          for (const [key, value] of Object.entries(object1)) {
            list += `<li>${key}: ${value}</li>`;
          }
    
          return list;
        })()
      }
    </ul>
    `;

以我簡化的例子來說，使用 IIFE 看起來變得更複雜了，

但是用在比較長的 `html` 來說比較能看出結構

[JSFiddle](https://jsfiddle.net/4h2n9L8t/)

{{< br >}}

## 參考資料

[JavaScript Immediately-invoked Function Expressions (IIFE)](https://flaviocopes.com/javascript-iife/)