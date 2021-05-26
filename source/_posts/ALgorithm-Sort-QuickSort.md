---
title: 算法-排序-快排
tags:
  - Sort
  - Algorithm
toc: true
date: 2021-05-26 15:15:11
categories:
  - Algorithm
---

## 代码

使用语言: go (1.13.15)
使用工具: GoLang
涉及文件: sort_quick.go

### 中枢不动 v1
原理: 找到一个中枢，保持不动，把小于中枢的值放到他左边，大于中枢的值放到他右边，然后再以此方法对这两部分数据分别进行快速排序
```go
func QuickSort_v1_1(array []int, start, end int) {
	if start < end {
		key := array[start]                   // 用待排序数组的第一个作为中枢
		i := start
		for j := start + 1; j <= end; j ++ {  // 双指针往后移动 j指针指向>中枢的元素, i指针指向<=中枢的元素, 中枢一直不动，在第一个元素
			if key > array[j] {
				i ++
				swap(array, j, i)
			}
		}
		swap(array, i, start)                 // 将中枢放到指定位置
		QuickSort_v1_1(array, start, i - 1)
		QuickSort_v1_1(array, i + 1, end)
	}
}
```

此方法与上面 QuickSort_Simple 原理一样，只是换了种写法
```go
func QuickSort_v1_2(array []int, start, end int) {
	if start < end {
		pivot := QuickSort_v1_2_partition(array, start, end)
		QuickSort_v1_2(array, start, pivot - 1)
		QuickSort_v1_2(array, pivot + 1, end)
	}
}
func QuickSort_v1_2_partition(array []int, start, end int) int{
	key := array[start]
	i := start
	for j := start + 1; j <= end ; j++ {
		if key > array[j] {
			i ++
			swap(array, j, i)
		}
	}
	swap(array, i, start)
	return i
}
```

### 中枢变化 v2
```go
func QuickSort_v2_1(array []int, start, end int)  {
	if start < end {
		pivot := QuickSort_v2_1_partition(array, start ,end)
		QuickSort_v2_1(array, start, pivot - 1)
		QuickSort_v2_1(array, pivot + 1, end)
	}
}
func QuickSort_v2_1_partition(array []int, start, end int) int {
	pivot := start
	for start != end {
		if pivot != end {
			/**
			第一次循环的时候用第一个元素作为中枢，和最后一个进行对比，
			  如果小于最后一个元素，执行end-- 就是和倒数第二个元素进行对比，以此类推。
			  如果大于最后一个元素，就和最后一个元素交互，然后让pivot指向最后一个元素
			下一轮循环的时候回指向下面的else方法和前面的元素进行对比。
			也就是说这个中枢的位置始终是在变动的，所以这一轮执行了之后小于中枢的值就会放到他的前面，大于中枢的值就会放到他的后面
			 */
			if array[end] < array[pivot] {
				swap(array, end, pivot)
				pivot = end
			} else {
				end --
			}
		} else {
			if array[start] > array[pivot] {
				swap(array, start, pivot)
				pivot = start
			} else {
				start ++
			}
		}
	}
	return pivot
}
```

```go
func QuickSort_v2_2(array []int, start, end int) {
	if start < end {
		pivot := QuickSort_v2_2_partition(array, start ,end)
		QuickSort_v2_2(array, start, pivot - 1)
		QuickSort_v2_2(array, pivot + 1, end)
	}
}
func QuickSort_v2_2_partition(array []int, start, end int) int {
	pivot := array[start] // 采用子序列的第一个元素作为中枢
	for start < end {
		for start < end && array[end] >= pivot { // 从后往前在后半部分中寻找第一个小于中枢的元素
			end --
		}
		swap(array, start, end)                     // 将这个元素交换到前半部
		for start < end && array[start] <= pivot {  // 从前往后在前半部寻找第一个大于等于中枢的元素
			start ++
		}
		swap(array, start, end)                     // 将这个元素交换到后半部
	}
	return start
}
```

### 两种结合--最易理解版本
原理: 中枢位置不变，中枢之后的元素进行最前和最后的交换，最后再把中枢放到指定位置，相当于上面两种的结合

```go
func QuickSort_v3_1(array []int, start, end int) {
	if start < end {
		pivot, i, j := array[start], start, end // 选定第一个元素为中枢
		for i < j {
			for i < j && array[j] >= pivot {  // 从后往前，找一个比中枢小的元素位置j
				j --
			}
			for i < j && array[i] <= pivot {  // 从前往后，找一个比中枢大的元素位置i
				i ++
			}
			if i < j {
				swap(array, i, j)	// 中枢不动，交互 [i j]
			}
		}
		swap(array, i, start)	// 中枢动，交互 [i 中枢]
		QuickSort_v3_1(array, start, i - 1)
		QuickSort_v3_1(array, i + 1, end)
	}
}
```

### 两种结合
```go
func QuickSort_v3_2(array []int, start, end int)  {
	if start < end {
		pivot := QuickSort_v3_2_partition(array, start, end)
		QuickSort_v3_2(array, start, pivot - 1)
		QuickSort_v3_2(array, pivot + 1, end)
	}
}
func QuickSort_v3_2_partition(array []int, start, end int) int {
	pivot := start
	for start != end {
		for start < end && array[start] < array[pivot] {
			start ++
		}
		for start < end && array[end] >= array[pivot] {
			end --
		}
		swap(array, start, end)
	}
	return start
}
```
## 测试

涉及文件: sotr_test.go

```go
func Test_QuickSort(t *testing.T) {
	var array = []int{2, 9, -9, 7, -6, 8, 0}

	t.Logf("before QuickSort_v1_1:%v", array)
	QuickSort_v1_1(array, 0, len(array)-1)
	t.Logf("after  QuickSort_v1_1:%v", array)

	array = []int{2, 9, -9, 7, -6, 8, 0}
	t.Logf("before QuickSort_v1_2:%v", array)
	QuickSort_v1_2(array, 0, len(array)-1)
	t.Logf("after  QuickSort_v1_2:%v", array)

	array = []int{2, 9, -9, 7, -6, 8, 0}
	t.Logf("before QuickSort_v2_1:%v", array)
	QuickSort_v2_1(array, 0, len(array)-1)
	t.Logf("after  QuickSort_v2_1:%v", array)

	array = []int{2, 9, -9, 7, -6, 8, 0}
	t.Logf("before QuickSort_v2_2:%v", array)
	QuickSort_v2_2(array, 0, len(array)-1)
	t.Logf("after  QuickSort_v2_2:%v", array)

	array = []int{2, 9, -9, 7, -6, 8, 0}
	t.Logf("before QuickSort_v3_1:%v", array)
	QuickSort_v3_1(array, 0, len(array)-1)
	t.Logf("after  QuickSort_v3_1:%v", array)

	array = []int{2, 9, -9, 7, -6, 8, 0}
	t.Logf("before QuickSort_v3_2:%v", array)
	QuickSort_v3_2(array, 0, len(array)-1)
	t.Logf("after  QuickSort_v3_2:%v", array)
}
```

## 测试输出
```shell
=== RUN   Test_QuickSort
--- PASS: Test_QuickSort (0.00s)
    sotr_test.go:72: before QuickSort_v1_1:[2 9 -9 7 -6 8 0]
    sotr_test.go:74: after  QuickSort_v1_1:[-9 -6 0 2 7 8 9]
    sotr_test.go:77: before QuickSort_v1_2:[2 9 -9 7 -6 8 0]
    sotr_test.go:79: after  QuickSort_v1_2:[-9 -6 0 2 7 8 9]
    sotr_test.go:82: before QuickSort_v2_1:[2 9 -9 7 -6 8 0]
    sotr_test.go:84: after  QuickSort_v2_1:[-9 -6 0 2 7 8 9]
    sotr_test.go:87: before QuickSort_v2_2:[2 9 -9 7 -6 8 0]
    sotr_test.go:89: after  QuickSort_v2_2:[-9 -6 0 2 7 8 9]
    sotr_test.go:92: before QuickSort_v3_1:[2 9 -9 7 -6 8 0]
    sotr_test.go:94: after  QuickSort_v3_1:[-9 -6 0 2 7 8 9]
    sotr_test.go:97: before QuickSort_v3_2:[2 9 -9 7 -6 8 0]
    sotr_test.go:99: after  QuickSort_v3_2:[-9 -6 0 2 7 8 9]
PASS
```