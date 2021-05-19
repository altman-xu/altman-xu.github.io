---
title:  算法-排序-选择
tags:
  - Sort
  - Algorithm
toc: true
date: 2021-05-18 14:35:01
categories:
  - Algorithm
---

## 代码

使用语言: go (1.13.15)
使用工具: GoLang
涉及文件: sort_bubble.go


```go
package algorithm

func SelectSort(array[]int)  {
	for i := 0; i < len(array); i++ {
		index := i
		for j := i + 1; j < len(array); j++ {
			// array[index] > array[j]:(大的往后排--升序)    array[index] < array[j]:(小的往后排--降序)
			if array[index] > array[j] {
				index = j;
			}
		}
		if i != index {
			swap(array, i, index)
		}
	}
}
```

## 测试

涉及文件: sotr_test.go

```go
package algorithm

import (
	"testing"
)

func Test_SelectSort(t *testing.T) {
	var array = []int{1,0,3,2,4}

	t.Logf("before SelectSort:%v", array)
	SelectSort(array)
	t.Logf("after  SelectSort:%v", array)

	array = []int{1,0,3,2,4}
	t.Logf("before SelectSort_recursion:%v", array)
	SelectSort_recursion(array, 0)
	t.Logf("after  SelectSort_recursion:%v", array)
}
```

## 测试输出

```shell
=== RUN   Test_SelectSort
--- PASS: Test_SelectSort (0.00s)
    sotr_test.go:40: before SelectSort:[1 0 3 2 4]
    sotr_test.go:42: after  SelectSort:[0 1 2 3 4]
    sotr_test.go:45: before SelectSort_recursion:[1 0 3 2 4]
    sotr_test.go:47: after  SelectSort_recursion:[0 1 2 3 4]
PASS
```