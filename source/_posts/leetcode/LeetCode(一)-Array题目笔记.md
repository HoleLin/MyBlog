---
title: LeetCode-Array题目笔记
date: 2022-01-14 12:28:45
index_img: /img/cover/LeetCode.jpeg
cover: /img/cover/LeetCode.jpeg
tags:
- Array
categories:
- LeetCode
updated:
type:
comments:
description:
keywords:
top_img:
mathjax:
katex:
aside:
aplayer:
highlight_shrink:
---

### 参考文献

#### 1.Two Sum

```java
// Input: nums = [2,7,11,15], target = 9
// Output: [0,1]
// Explanation: Because nums[0] + nums[1] == 9, we return [0, 1].
class Solution {
    public int[] twoSum(int[] nums, int target) {
        int[] result = new int[2];
        // 采用nums[j]=target-nums[i]来判断两数之和
        HashMap<Integer, Integer> map = new HashMap<>(nums.length);
        for (int i = 0; i < nums.length; i++) {
            final int surplusNumber = target - nums[i];
            if (map.containsKey(surplusNumber)) {
                final Integer index = map.get(surplusNumber);
                result[0] = index;
                result[1] = i;
                break;
            } else {
                map.put(nums[i], i);
            }
        }
        return result;
    }
}
```

#### 26.Remove Duplicates from Sorted Array

```java
// Input: nums = [1,1,2]
// Output: 2, nums = [1,2,_]
class Solution {
    public int removeDuplicates(int[] nums) {
        if (nums == null || nums.length == 0) {
            return 0;
        }
        int slowIndex = 0;
        for (int fastIndex = 1; fastIndex < nums.length; fastIndex++) {
            if (nums[slowIndex] != nums[fastIndex]) {
                nums[++slowIndex] = nums[fastIndex];
            }
        }
        return ++slowIndex;
    }
}
```

#### 27.Remove Element

```java
// Input: nums = [3,2,2,3], val = 3
// Output: 2, nums = [2,2,_,_]
class Solution {
    public static int removeElement(int[] nums, int val) {
        // 采用快慢指针的方法
        int slowIndex = 0;
        // 快慢指针在目标值val没出现之前,快慢指针的值相同;
        // 当目标值val出现时,快慢指针的值就会不一样,当执行 nums[slowIndex++] = nums[fastIndex];
        // 后面的值就覆盖目标值,从而达到删除目标值的作用
        for (int fastIndex = 0; fastIndex < nums.length; fastIndex++) {
            if (val != nums[fastIndex]) {
                nums[slowIndex++] = nums[fastIndex];
            }
        }
        return slowIndex;
    }
}
```

#### 35.Search Insert Position

```java
// Input: nums = [1,3,5,6], target = 5
// Output: 2
class Solution {
    public int searchInsert(int[] nums, int target) {
        int index;
        for (index = 0; index < nums.length; index++) {
            if (target <= nums[index]) {
                break;
            }
        }
        return index;
    }
}
```

#### 283.Move Zeroes

```java
class Solution {
    public void moveZeroes(int[] nums) {
        // 快慢指针
        int slowIndex = 0;
        int count = 0;
        for (int fastIndex = 0; fastIndex < nums.length; fastIndex++) {
            // 采用27.Remove Element策略先想0移除掉,并通过count计算0的数量在数组尾部补充0
            if (0 != nums[fastIndex]) {
                nums[slowIndex++] = nums[fastIndex];
            } else {
                count++;
            }
        }
        for (int i = 1; i <= count; i++) {
            nums[nums.length - i] = 0;
        }
    }
}
```

#### 209.Minimum Size Subarray Sum

* 滑动窗口

```java
class Solution {
    public int minSubArrayLen(int target, int[] nums) {
        int left = 0;
        int sum = 0;
        int result = Integer.MAX_VALUE;
        for (int right = 0; right < nums.length; right++) {
            sum += nums[right];
            while (sum >= target) {
                result = Math.min(result, right - left + 1);
                // 若sum>=target,则需要收缩窗口
                sum -= nums[left++];
            }
        }
        return result == Integer.MAX_VALUE ? 0 : result;
    }
}
```



