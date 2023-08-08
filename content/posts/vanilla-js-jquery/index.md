---
title: "Vanilla JS & jQuery 的前端測試"
date: "2023-08-07T10:21:05Z"
tags: [javascript]
---

# Vanilla JS & jQuery 的前端測試

---

{{< br >}}

# 起因

公司的產品前端用的是 Vanilla JS 搭配 jQuery 刻的，而  html 則是 Server Side Render。

引入 JS 的方式就是傳統的 `<script>` 標籤：

    <script type="text/javascript" src="/assests/legacy.js"></script>

部分新功能則是用上了 js module：

    <script type="module" src="/assets/new.js"></script>

{{< br >}}

現在需要對這兩種系統的進行測試，並且在：

1. CI (linux container 中) console 可以執行
2. IDE 可以執行

{{< br >}}

作為一個前端小白實在是搞不定 webpack 之類的來滿足 js module，

而且在網上 Google 了一陣都沒有滿意的完整簡單教學可以套用，

自己做了一些嘗試之後，終於能夠簡單地 (?) 把測試跑了起來，以下詳述整個專案的設定。

{{< br >}}

# 需求

在執行測試的環境用 Node.js 執行，

測試框架的部分試了幾種，最後只有 `Jest` 可以滿足。

## 專案架構

    src/
    ├── package.json
    ├── .babelrc
    ├── jest.config.js
    ├── jest.setup.js
    └── assets
        ├── jquery-3.6.4.min.js
        ├── legacy.js
        ├── new.js
        └── test
            ├── legacy.test.js
    				└── new.test.js

{{< br >}}

# 設定

## `package.json`

    {
      "name": "OOXX",
      "version": "1.0.0",
      "license": "MIT",
      "scripts": {
        "test": "jest"
      },
      "devDependencies": {
        "@babel/preset-env": "^7.20.2",
        "jest": "^29.3.1",
        "jest-environment-jsdom": "^29.3.1",
        "@jest/globals": "^29.3.1"
      }
    }

接下來大致解說一下我選用了哪些套件及其原因。

{{< br >}}

### **`babel/preset-env`**

儘管是前端的 code ，但是是用 `Node.js` 去跑，而 `Node.js` 不支援 ES6 的 **`import`** 和 **`export`**

(當時的版本是這樣，我不知道現在最新版怎麼樣了)

這個東西能夠轉換 ES6+ 到更早版本的 ECMAScript，可以讓程式碼在不支援最新語法的環境中運行。

{{< br >}}

### `jest-environment-jsdom`

因為不是在 browser 中測試，但是前端的 code 大部分都是在對 dom 操作，所以引入 `jsdom` 來模擬瀏覽器 dom 環境。

{{< br >}}

## `.babelrc`

    {
      "presets": ["@babel/preset-env"]
    }

{{< br >}}

## `jest.config.js`

    module.exports = {
        "testEnvironment": "jsdom",
        "setupFilesAfterEnv": ['./jest.setup.js'],
        "testMatch": [
            "<rootDir>/assets/test/**/*.test.js"
        ]
    };

{{< br >}}

## `jest.setup.js`

    import $ from './assets/jquery-3.6.4.min.js';
    
    global.$ = $;
    global.jQuery = $;
    global.Settings = {};
    
    // mock t()
    global.t = function (app, text, vars) {
        return text;
    }
1. 注入 `jQuery` 讓我操作前端 dom 的 code 可以正常執行。
2. 注入 `Settings` 這個是我們產品中一定要引入的 js 會產生的全域變數，在這邊造假掉
3. 造假掉一個叫做 `t` 的全域 function，這個是我們處理多語系在用的，但是我沒有引入所以造假掉

{{< br >}}

# 測試 `legacay.js` (應對傳統的 case)

## `legacy.js`

    Share = {
        statuses: [],
    
        updateInfo: function () {
            let statuses = this.statuses;
    
            $('#tbl > tbody  > tr').each(function() {
              let html;
    
              if (Settings.currentUser == setatuses.sharer) {
                  html = `Shared to <span>${statuses.sharee}</span>`;
              } else {
                  html = `Shared from <span>${statuses.sharer}</span>`;
              }
    
    					$(this).children('.info').html(html)
            });
        }
    }

`legacy.js` 會污染全域產生一個 `Share` 的 Object

我這邊的功能是顯示分享給別人或被分享

{{< br >}}

## `legacy.test.js`

    import '../legacy.js';
    import {expect} from "@jest/globals";
    
    describe('test share icon', () => {
        beforeEach(() => {
            Share.statuses = {
                "sharer": "user001",
                "sharee": "user002"
            };
    
            document.body.innerHTML = `
                <table id="tbl">
                    <tbody>
                    <tr><td class="info"></td></tr>
                    </tbody>
                </table>`;
        });
    
        test('render sharer', () => {
            Settings.currentUser = 'user001';
    
            Share.updateInfo();
    
            expect($('.info').html())
                .toEqual(`Shared to <span>user002</span>`);
        });
    
        test('render sharee', () => {
            Settings.currentUser = 'user002';
    
            Share.updateInfo();
    
            expect($('.info').html())
                .toEqual(`Shared from <span>user001</span>`);
        });
    });

`import '../legacy.js` 就相當於使用 `<script>` 去引入 `javascript`

所以注意事項也是相同的：

1. 可能會污染全域
    1. 可能會蓋掉全域變數之類的東西
2. 依賴必須先被處理，例如我有用到 `Settings` 所以我比需在一個更早的時機點 (我這邊是 Setup) 去建構好他的依賴。

我必須自己建構一段 `html` 來讓他操作，因為 document 現在就是空空如也。

{{< br >}}

# 測試 `new.js` (應對 js module)

## `new.js`

    export function renderEngineInfo(engineInfo) {
        $('#engine_version').html(engineInfo.engineVersion);
        $('#virus_version').html(engineInfo.virusVersion);
        $('#virus_time').html(engineInfo.virusLastUpdateTime);
    }

## `new.test.js`

    import {renderEngineInfo} from "../new";
    import {describe, expect, test} from '@jest/globals';
    
    describe('render test', () => {
        beforeEach(() => {
            document.body.innerHTML = `
                <div id="test-div">
                    <div id="engine_version"></div>
                    <div id="virus_version"></div>
                    <div id="virus_time"></div>
                </div>`;
        });
    
        test('render engine info', () => {
            let engineInfo = {
                engineVersion: 'v1',
                virusVersion: 'v2',
                virusLastUpdateTime: '123456789',
            };
            let result = `
                <div id="engine_version">${engineInfo.engineVersion}</div>
                <div id="virus_version">${engineInfo.virusVersion}</div>
                <div id="virus_time">${engineInfo.virusLastUpdateTime}</div>`;
    
            renderEngineInfo(engineInfo);
    
            let actual = document.querySelector('#test-div').innerHTML;
    
            expect(actual.replace(/[\r\n]/gm, ''))
                .toEqual(result.replace(/[\r\n]/gm, ''));
        });
    });

這沒什麼好解釋的，就是引入模組進行測試，我們現在的目標就是盡量模組化。

不然必須在 html 控制引入的順序以及注意不要有人寫出相同名稱的 function。

舊的專案其實可以先學 `OwnCloud` ，用 namespace 的方式避免引入衝突：

`oc.js`

    OC = {};

`share.js`

    OC.Share = {
        doSomeThing: function() { ... }
    };

`foo.js`

    OC.Foo = {
        doSomeThing: function () { ... }
    }

如此一來就不會有覆蓋的問題，但還是有順序跟污染的問題。

{{< br >}}

# References

基礎認識：

[十分鐘上手前端單元測試 - 使用 Jest](https://www.casper.tw/development/2020/02/02/jest-intro/)

{{< br >}}

這篇的方法我後來沒有使用：

[Jest使用ES6语法配置 - 掘金](https://juejin.cn/post/6990172738853797902)