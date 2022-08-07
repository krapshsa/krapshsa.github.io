---
title: "簡易 C Mock — wrapper function"
date: "2022-08-06T11:12:00+08:00"
draft: false
toc: true
autoCollapseToc: false
comment: true
categories: []
tags: [C]
---

## 起因

幫一個舊的 Lib 寫測試的時候會去讀 config，

但是這個路徑在我的 CI 環境並不存在，

通常我寫 php 的時候，針對這種外部相依就用 function 包起來再 Mock 掉，

但是 C 我已經忘得差不多了，想找一個簡單可以做到類似效果的方法。


{{< br >}}

## 目前解法

不囉唆，先上 code

[GitHub - krapshsa/c-native-mock](https://github.com/krapshsa/c-native-mock)

這個簡單的 Case 是這樣：

1. `main()` 呼叫 `foo()` ， `foo()` 呼叫 `config_load()`
![](c-mock-9adcb40d-b2d3-4874-9829-170784e9f812.png)
2. 我想要不改變 `foo.c` ，讓測試 (`main.c` ) 可以不去真的用到 `config_load()` 

{{< br >}}

其實這就相當於寫測試：

 `main()` 是 Test，而 `foo.c` 的 `foo()` 是我的 SUT (System Under Test)。


{{< br >}}

手法就是 Wrap，根據 `ld` 的 Man Page：

> `--wrap=symbol`
>        Use a wrapper function for symbol.  Any undefined reference to symbol will be resolved to "`__wrap_symbol`".  Any undefined reference to "`__real_symbol`" will be resolved to symbol.

{{< br >}}

操作步驟：

1. Caller 寫一個加上 `__wrap_` 開頭的函式，用來換掉某一個實作。

    例如替換掉 `config_load` 我就要定義一個 `__wrap_config_load`。

2. Caller 可以使用加上 `__real_` 開頭的函式，就可以呼叫到原始的實作 (Optional)。
3. 編譯的參數加上 `-Wl,--wrap=<symbol>`

{{< br >}}

可以用 nm 指令看一下編出來的 `.o`

T:  表示在 Text Section 找得到這個 symbol

U: 表示這個 symbol 是 undefined

```
# nm config.o
0000000000000000 T config_load
                 U fclose
                 U fopen
                 U fseek
```
```
# nm foo.o
                 U config_load
0000000000000000 T foo
```
```
# nm main.o
0000000000000000 T __wrap_config_load
                 U foo
0000000000000011 T main
                 U puts
```

{{< br >}}

## 常犯錯誤

如果 `config_load` 定義在 `foo.c` 裡面，試著跑看看：

```
# make main
gcc -c main.c
gcc -c foo.c
gcc -Wl,--wrap=config_load -o main main.o foo.o
# ./main
Segmentation fault
```

結果這樣是不行的，用 nm 查看：

```
# nm foo.o
0000000000000011 T config_load
                 U fclose
0000000000000000 T foo
                 U fopen
                 U fseek
```
```
# nm main.o
0000000000000000 T __wrap_config_load
                 U foo
0000000000000011 T main
                 U puts
```

要把 ld 告訴我們的使用方法放在心上：

> Any undefined reference to symbol will be resolved to "`__wrap_symbol`"

呼叫 `config_load` 並不是 undefined，

所以這告訴我們實作 & 寫測試的時候把要 Mock 的東西切出去，

就可以用這個手法來替換掉實作，也就是說要把相依但是職責不應該屬於我的 Code 另外放。


{{< br >}}

## 另外一個方法： `#ifndef ... #define ... #endif`

延伸自 `undefined` 的想法，我們也可以直接改寫 `foo.c` 

`foo.c` 

```
#include "stdio.h"
#include "config.h"
#include "foo.h"

#ifndef config_load
    #define config_load() my_config_load()
#endif

void my_config_load() {
    printf("wrap\n");
}

void foo() {
    config_load();
}
```

`main.c`

```
#include "foo.h"

int main() {
    foo();

    return 0;
}
```

`Makefile`

```
config.o:
	gcc -c config.c

foo.o:
	gcc -c foo.c

main.o:
	gcc -c main.c

main: main.o foo.o config.o
	gcc -o main main.o foo.o config.o

clean:
	rm *.o main
```

{{< br >}}

應該是可以搭配 Target-specific Variable Values 編出 production / development 的 `foo.o`

感謝大神同事給的參考資料：

[Target-specific (GNU make)](https://www.gnu.org/software/make/manual/html_node/Target_002dspecific.html)

{{< br >}}

## 參考資料

[How to mock function in C when its caller function is defined in same file?](https://stackoverflow.com/questions/31156327/how-to-mock-function-in-c-when-its-caller-function-is-defined-in-same-file)

[Override a function call in C](https://stackoverflow.com/questions/617554/override-a-function-call-in-c)

[How to wrap existing function in C](https://stackoverflow.com/questions/43183060/how-to-wrap-existing-function-in-c)

[wrap function的使用 @ Orion's blog :: 痞客邦 ::](https://orionlin.pixnet.net/blog/post/96013596-wrap-function%E7%9A%84%E4%BD%BF%E7%94%A8)