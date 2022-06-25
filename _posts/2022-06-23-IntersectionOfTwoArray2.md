---
layout: post
title: "Intersection of Two Arrays II"
subtitle: "DSA Solving and practicing"
date: 2022-06-23
author: "Hitesh Kumar"
header-img: "img/leet1.jpg"
tags: [dsa, leetcode, golang]
---


```go


func intersect(nums1 []int, nums2 []int) []int {
    intersection1 := []int{}
	mapforone := make(map[int]int)

	for i := 0; i < len(nums1); i++ {
		mapforone[nums1[i]]++
	}

	for i := 0; i < len(nums2); i++ {
		if mapforone[nums2[i]] > 0 {
			intersection1 = append(intersection1, nums2[i])
			mapforone[nums2[i]]--
		}
	}
    
    return intersection1
    
}


```

[![Screenshot-2022-06-24-at-22-31-56-Intersection-of-Two-Arrays-II-Leet-Code.png](https://i.postimg.cc/Dw2BLh43/Screenshot-2022-06-24-at-22-31-56-Intersection-of-Two-Arrays-II-Leet-Code.png)](https://postimg.cc/ykbmCwTL)





