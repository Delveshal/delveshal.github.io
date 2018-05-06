---
title: leetcode 56. Merge Intervals
date: 2018-05-06 20:27:15
tags:
categories: leetcode
---

# Description

Given a collection of intervals, merge all overlapping intervals.

# Example1

```
Input: [[1,3],[2,6],[8,10],[15,18]]
Output: [[1,6],[8,10],[15,18]]
Explanation: Since intervals [1,3] and [2,6] overlaps, merge them into [1,6].
```
<!-- more -->
# Example 2

```
Input: [[1,4],[4,5]]
Output: [[1,5]]
Explanation: Intervals [1,4] and [4,5] are considerred overlapping.
```
# 题解

```
/**
 * Definition for an interval.
 * type Interval struct {
 *	   Start int
 *	   End   int
 * }
 */
func merge(intervals []Interval) []Interval {
	if len(intervals) <= 0{
		return intervals
	}
	sort.Slice(intervals, func(i, j int) bool {
		return intervals[i].Start < intervals[j].Start
	})
	result := make([]Interval,0,len(intervals))
	result = append(result,intervals[0])
    tail := len(result)-1
	for i:=1;i<len(intervals);i++{
        tail = len(result)-1
		if result[tail].End >= intervals[i].Start{
			if result[tail].End < intervals[i].End{
				result[tail].End = intervals[i].End
			}
		}else{
				result = append(result, intervals[i])
        }
	}
    return result
}
```
