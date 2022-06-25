---
layout: post
title: "Maximum Subarray"
subtitle: "DSA Solving and practicing"
date: 2022-06-22
author: "Hitesh Kumar"
header-img: "img/leet1.jpg"
tags: [dsa, leetcode, golang]
---


```go

func maxSubArray(nums []int) int {

    currSum := 0
    maxSum := -99999
    
    for idx, _ := range nums {
        currSum += nums[idx]
        if currSum >= maxSum {
            maxSum = currSum
        } 
        if currSum < 0 {
            currSum = 0
        }
    }
    
    return maxSum
    
}
```

[![Screenshot-2022-06-23-at-20-25-10-Maximum-Subarray-Leet-Code.png](https://i.postimg.cc/nhwghNsY/Screenshot-2022-06-23-at-20-25-10-Maximum-Subarray-Leet-Code.png)](https://postimg.cc/w1L0WfQ7)








