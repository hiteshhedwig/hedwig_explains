---
layout: post
title: "Contains Duplicate"
subtitle: "DSA Solving and practicing"
date: 2021-06-21
author: "Hitesh Kumar"
header-img: "img/post-bg-os-metro.jpg"
tags: [dsa, leetcode, golang]
---
 
## My First Try :

```go

import "sort"

func containsDuplicate(nums []int) bool {
    sort.Slice(nums, func(i, j int) bool {
        return nums[i] < nums[j]
    })
    for idx , _ := range nums {
        if idx < (len(nums)-1) {
            if nums[idx] == nums[idx+1] {
                return true
            }   
        }
    }
    
    return false
    
}

```
[![Screenshot-2022-06-22-at-16-47-52-Contains-Duplicate-Submission-Detail-Leet-Code.png](https://i.postimg.cc/SNMWLwVZ/Screenshot-2022-06-22-at-16-47-52-Contains-Duplicate-Submission-Detail-Leet-Code.png)](https://postimg.cc/D87Jnp9s)

## Second try:

```go

func containsDuplicate(nums []int) bool {
    hashmap := make(map[int]int)
    
    for _ , num := range nums {
        hashmap[num] = 1
        
    }
    
    if len(nums)-len(hashmap) > 0 || len(nums)-len(hashmap) < 0 {
        return true
    }
    
    
    return false
    
}
```

[![Screenshot-2022-06-22-at-16-49-37-Contains-Duplicate-Submission-Detail-Leet-Code.png](https://i.postimg.cc/BnNct2ms/Screenshot-2022-06-22-at-16-49-37-Contains-Duplicate-Submission-Detail-Leet-Code.png)](https://postimg.cc/LJYZbgd0)






