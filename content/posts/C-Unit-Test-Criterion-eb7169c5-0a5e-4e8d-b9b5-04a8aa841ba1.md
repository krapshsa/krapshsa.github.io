---
title: "C Unit Test - Criterion 簡單範例"
date: "2022-08-05T01:52:00+08:00"
draft: false
toc: true
autoCollapseToc: false
comment: true
categories: []
tags: [C]
---

## 起因

要用一個公司很久以前自己寫的 C Lib，沒有文件所以不知道行為是什麼。

使用的過程中有發現問題，就進行一些修正。 

想說幫他加上一些測試案例，幫助之後要使用的人。


{{< br >}}

## 環境

### 建置方法

由於公司是用 CentOS / RHEL / Oracle，我挑了一個差不多的 image 來建 CI

重點是 rpm 我從 github 上面裝，就可以直接 `-l` 使用他的 Lib

其他是這個 C lib 需要的其他 Lib，跟底下範例無關


{{< br >}}

`Dockerfile`

```
FROM oraclelinux:8.6

RUN yum install -y gcc make pcre pcre-devel
RUN yum --enablerepo=ol8_codeready_builder install -y glibc glibc-common glibc-devel glibc-headers glibc-static
RUN rpm -ivh https://github.com/samber/criterion-rpm-package/releases/download/2.3.3/libcriterion-devel-2.3.3-2.el7.x86_64.rpm
```

{{< br >}}

## Code

`sut.h`

```
#ifndef TEST_CRITERION_SUT_H
#define TEST_CRITERION_SUT_H
typedef struct result
{
    char *begin;
} Result;

extern int doSomething(char *pContent, Result **ppResult);

extern void sutFree(Result **ppResult);
#endif //TEST_CRITERION_SUT_H
```

{{< br >}}

`sut.c`

```
#include <stdlib.h>
#include <string.h>
#include "sut.h"

int doSomething(char *pContent, Result **ppResult)
{
    if (0 == strcmp(pContent, "INPUT_1")) {
        *ppResult = (Result*)malloc(sizeof(Result));
        (*ppResult)->begin = "OUTPUT_1";
        return 1;
    }

    return 0;
}

void sutFree(Result **ppResult)
{
    if(ppResult != NULL && *ppResult != NULL) {
        free(*ppResult);
        *ppResult = NULL;
    }
}
```

{{< br >}}

`test.c`

```
#include <string.h>
#include <criterion/criterion.h>
#include "sut.h"

int ret   = 0;
int rule  = -1;
Result *s = NULL;

void setup(void) {
    ret  = 0;
    rule = -1;
    s    = NULL;
}

void teardown(void) {
    if (NULL != s) {
        sutFree(&s);
    }
}

void givenContent(char *content) {
    ret = doSomething(content, &s);
}

void returnValueShouldBeSuccess() {
    cr_assert(ret > 0);
}

void returnValueShouldBeFailed() {
    cr_assert(ret <= 0);
}

void matchContentShouldBe(char *result) {
    cr_assert_eq(0, strcmp(result, s->begin));
}

TestSuite(single_rule_suite, .init = setup, .fini = teardown);

Test(single_rule_suite, test_success) {
    givenContent("INPUT_1");

    returnValueShouldBeSuccess();
    matchContentShouldBe("OUTPUT_1");
}

Test(single_rule_suite, test_failed) {
    givenContent("INPUT_2");

    returnValueShouldBeFailed();
}
```

{{< br >}}

`Makefile`

```
CC = gcc -Wall

sut.o: sut.c
	${CC} -c sut.c

test.o: test.c
	${CC} -c test.c

test: clean test.o sut.o
	${CC} -o test test.o sut.o -lcriterion
	./test

clean:
	rm -f *.o
```

{{< br >}}

## 結果

```
./test
[====] Synthesis: Tested: 2 | Passing: 2 | Failing: 0 | Crashing: 0
```

{{< br >}}

## 簡易說明

[Getting started - Criterion 2.4.1-rc-1-g56f8f1a-dirty documentation](https://criterion.readthedocs.io/en/master/starter.html?highlight=suite#configuration-reference)

{{< br >}}

### Test 基本用法

```
Test(suite_name, test_name, .init = setup, .fini = teardown) {
    // test contents
}
```

{{< br >}}

### Test Suite 基本用法

官方給出的例子

```
TestSuite(suite_name, [params...]);

Test(suite_name, test_1) {
}

Test(suite_name, test_2) {
}
```

{{< br >}}

我實際上是這樣用：

```
TestSuite(single_rule_suite, .init = setup, .fini = teardown);

Test(single_rule_suite, test_success) {
    ...
}

Test(single_rule_suite, test_failed) {
    ...
}
```

因為大家的初始化跟銷毀都一樣，不需要在 `Test` 裡面重複寫


{{< br >}}

## 參考資料