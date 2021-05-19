---
title: 算法-排序-插入
tags:
  - Sort
  - Algorithm
toc: true
date: 2021-05-19 10:29:45
categories:
  - Algorithm
---


## 代码

使用语言: go (1.13.15)
使用工具: GoLang
涉及文件: sort_insert.go

### 直接插入

```go
package algorithm

func InsertSort_simple(array []int) {
	for i := 1; i < len(array); i++ {
		key := array[i]
		j := i
		for ; j > 0 ; j-- {
			// array[j-1] > key: (大的往后排--升序)    array[j-1] < key: (小的往后排--降序)
			if array[j-1] > key {
				array[j] = array[j-1]
			} else {
				break
			}
		}
		if i != j {
			array[j] = key
		}
	}
}
```

### 二分插入
```go

func InsertSort_binary(array []int) {
	for i := 1; i < len(array); i++ {
		// array[i] < array[i-1] && key < array[mid] : (小的往前排--升序)     array[i] > array[i-1] && key > array[mid] : (大的往前排--降序)
		if array[i] < array[i-1] {
			key := array[i]
			low := 0
			high := i - 1
			for low <= high {
				mid := low + (high - low)/2
				if key < array[mid] {
					high = mid - 1
				} else {
					low = mid + 1
				}
			}
			for j := i; j > low; j-- {
				array[j] = array[j - 1]
			}
			array[low] = key
		}
	}
}
```

### 递归版
```go
func InsertSort_recursion(array []int, n int) {
	if n < 2 {
		return
	}
	n--
	InsertSort_recursion(array, n)
	key := array[n]
	index := n -1
	for index >= 0 && array[index] > key {
		array[index + 1] = array[index]
		index --
	}
	array[index + 1] = key
}
```

## 测试

涉及文件: sotr_test.go

```go
func Test_InsertSort(t *testing.T) {
	var array = []int{1,0,3,2,4}

	t.Logf("before InsertSort_simple:%v", array)
	InsertSort_simple(array)
	t.Logf("after  InsertSort_simple:%v", array)

	array = []int{1,0,3,2,4}
	t.Logf("before InsertSort_binary:%v", array)
	InsertSort_binary(array)
	t.Logf("after  InsertSort_binary:%v", array)


	array = []int{1,0,3,2,4}
	t.Logf("before InsertSort_recursion:%v", array)
	InsertSort_recursion(array, len(array))
	t.Logf("after  InsertSort_recursion:%v", array)
}
```

## 测试输出
```shell
=== RUN   Test_InsertSort
--- PASS: Test_InsertSort (0.00s)
    sotr_test.go:55: before InsertSort_simple:[1 0 3 2 4]
    sotr_test.go:57: after  InsertSort_simple:[0 1 2 3 4]
    sotr_test.go:60: before InsertSort_binary:[1 0 3 2 4]
    sotr_test.go:62: after  InsertSort_binary:[0 1 2 3 4]
    sotr_test.go:66: before InsertSort_recursion:[1 0 3 2 4]
    sotr_test.go:68: after  InsertSort_recursion:[0 1 2 3 4]
PASS
```