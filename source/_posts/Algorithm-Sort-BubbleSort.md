---
title: 算法-排序-冒泡
tags:
  - Sort
  - Algorithm
toc: true
date: 2021-05-17 12:16:57
categories:
  - Algorithm
---

## 代码

使用语言: go (1.13.15)
使用工具: GoLang
涉及文件: sort_bubble.go


```go
package algorithm

func swap(array []int, i, j int) {
	if (i != j) { 	// 一定要检查两个地址是否相同，否则若两个相同，改地址数据会被 异或 清零
		array[i] ^= array[j]
		array[j] ^= array[i]
		array[i] ^= array[j]
	}
}
```

### 普通版本 往前排
```go
// 往前排
func BubbleSort_front(array []int) {
	for i := 0; i < len(array)-1; i++ {
		for j := i + 1; j < len(array); j++ {
			// array[j] < array[i]:(小的往前排--升序)     array[j] > array[i]:(大的往前排--降序)
			if array[j] < array[i] {
				swap(array, i, j)
			}
		}
	}
}
```

### 普通版本 往后排
```go
// 往后排
func BubbleSort_back(array []int) {
	for i := 1; i < len(array); i++ {
		for j := 0; j < len(array)-i; j++ {
			// array[j] > array[j + 1]:(大的往后排--升序)   array[j] < array[j + 1]:(小的往后排--降序)
			if array[j] > array[j+1] {
				swap(array, j, j+1)
			}
		}
	}
}
```

### 优化版本
```go
// 优化 (当后面的已经排序好的时候，可提前终止循环)
func BubbleSort_optimize(array []int) {
	var location int
	var count = len(array) - 1	// 初始化最后交换位置为最后一个元素
	for i := 0; i < len(array) - 1; i++ {
		location = count
		for j := 0; j < location; j++ {
			if array[j] > array[j + 1] {
				swap(array, j, j + 1)
				count = j		// 记录无需位置的结束，有序从 j+1 位置开始
			}
		}
		if count == location {	// 没有次序交换，排序完成
			break
		}
	}
}
```
### 递归版本
```go
// 递归版本
func BubbleSort_recursion(array []int, n int) {
	if n == 1 || len(array) == 0 {
		return
	}
	// 逐渐减少n,每次都把最大的放到最后面,直到n为1
	for i := 0; i < n - 1; i++ {
		if array[i] > array[i + 1] {
			swap(array, i, i + 1)
		}
	}
	BubbleSort_recursion(array, n - 1)
}

```

## 测试

涉及文件: sotr_test.go

```go
package algorithm

import (
	"testing"
)

func Test_bubbleSort(t *testing.T) {
	var array = []int{1,0,3,2,4}
	t.Logf("before BubbleSort_front:%v", array)
	BubbleSort_front(array)
	t.Logf("after  BubbleSort_front:%v", array)

	array = []int{1,0,3,2,4}
	t.Logf("before BubbleSort_back:%v", array)
	BubbleSort_back(array)
	t.Logf("after  BubbleSort_back:%v", array)

	array = []int{1,0,3,2,4}
	t.Logf("before BubbleSort_recursion:%v", array)
	BubbleSort_recursion(array, len(array))
	t.Logf("after  BubbleSort_recursion:%v", array)
}
```

## 测试输出

```shell
=== RUN   Test_bubbleSort
--- PASS: Test_bubbleSort (0.00s)
    sotr_test.go:22: before BubbleSort_front:[1 0 3 2 4]
    sotr_test.go:24: after  BubbleSort_front:[0 1 2 3 4]
    sotr_test.go:27: before BubbleSort_back:[1 0 3 2 4]
    sotr_test.go:29: after  BubbleSort_back:[0 1 2 3 4]
    sotr_test.go:32: before BubbleSort_recursion:[1 0 3 2 4]
    sotr_test.go:34: after  BubbleSort_recursion:[0 1 2 3 4]
PASS
```