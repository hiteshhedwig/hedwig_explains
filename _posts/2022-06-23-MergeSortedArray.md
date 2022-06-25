---
layout: post
title: "Merge Sorted Array"
subtitle: "DSA Solving and practicing"
date: 2022-06-23
author: "Hitesh Kumar"
header-img: "img/leet1.jpg"
tags: [dsa, leetcode, golang]
---


```go
import "sort"

func merge(nums1 []int, m int, nums2 []int, n int)  {
    
    if m == 0 && n == 1{
        nums1[0] = nums2[0]
    } 

    
    if m >= 0 && n >= 0 {
        num2Counter := 0
        for i:= m ; i < (m+n) ; i++ {
            nums1[i] = nums2[num2Counter]
            num2Counter++
        }
        sort.Ints(nums1)
    }

}
```

[![Screenshot-2022-06-23-at-21-19-59-Merge-Sorted-Array-Leet-Code.png](https://i.postimg.cc/zDKbNRt0/Screenshot-2022-06-23-at-21-19-59-Merge-Sorted-Array-Leet-Code.png)](https://postimg.cc/SjQN7jpM)





