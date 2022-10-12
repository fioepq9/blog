# 二分查找


二分查找是一种在有序序列中查找特定元素的搜索算法。优点是时间复杂度为 O(log n)，缺点是只能在有序序列中使用。

## 在升序数组 nums 中查找 target 的索引

### 搜索过程

![二分查找](https://raw.githubusercontent.com/ltlin9/note-img/master/blog-img/二分查找.svg)

### 实现代码（go）

```go
func Search(nums []int, target int) int {
	lo, hi := 0, len(nums)-1
	for lo <= hi {
		mid := (lo + hi) >> 1
		if target < nums[mid] {
			hi = mid - 1
		} else if target > nums[mid] {
			lo = mid + 1
		} else {
			return mid
		}
	}
	return -1
}
```

## 在升序数组 nums 中查找 target 的最小索引

### 搜索过程

![二分查找最小索引](https://raw.githubusercontent.com/ltlin9/note-img/master/blog-img/二分查找最小索引.svg)

### 实现代码（go）

```go
func SearchLeft(nums []int, target int) int {
	lo, hi := 0, len(nums)-1
	for lo <= hi {
		if nums[lo] == target {
			return lo
		}
		mid := (lo + hi) >> 1
		if target < nums[mid] {
			hi = mid - 1
		} else if target > nums[mid] {
			lo = mid + 1
		} else {
			hi = mid
		}
	}
	return -1
}
```

## 在升序数组 nums 中查找 target 的最大索引

### 搜索过程

![二分查找最大索引](https://raw.githubusercontent.com/ltlin9/note-img/master/blog-img/二分查找最大索引.svg)

### 实现代码（go）

```go
func SearchRight(nums []int, target int) int {
	lo, hi := 0, len(nums)-1
	for lo <= hi {
		if nums[hi] == target {
			return hi
		}
		mid := (lo+hi)>>1 + (lo+hi)&1
		if target < nums[mid] {
			hi = mid - 1
		} else if target > nums[mid] {
			lo = mid + 1
		} else {
			lo = mid
		}
	}
	return -1
}
```

