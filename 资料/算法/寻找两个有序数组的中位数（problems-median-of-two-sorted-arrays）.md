title: 寻找两个有序数组的中位数（problems/median-of-two-sorted-arrays）
author: WangWangZhou
tags: []
categories: []
date: 2019-12-11 14:57:00
---

# 寻找两个有序数组的中位数（problems/median-of-two-sorted-arrays）

## 题目
给定两个大小为 m 和 n 的有序数组 nums1 和 nums2。
请你找出这两个有序数组的中位数，并且要求算法的时间复杂度为 O(log(m + n))。
你可以假设 nums1 和 nums2 不会同时为空。

示例 1:
```
nums1 = [1, 3]
nums2 = [2]
则中位数是 2.0
```

示例 2:
```
nums1 = [1, 2]
nums2 = [3, 4]
则中位数是 (2 + 3)/2 = 2.5
```


## 思路1
先合并两个有序数组，然后求中位数。
```java
package median_of_two_sorted_arrays;

//寻找两个有序数组的中位数
class Solution {
	
	/**寻找两个有序数组的中位数*/
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
    	//合并两个有序数组
    	int[] c = merge(nums1, nums2);
    	//寻找中位数
    	if(c.length%2==1) {
    		return c[(c.length-1)/2];
    	}else {
    		int d = (c[(c.length-1)/2]+c[(c.length-1)/2+1])/2;
    		int e = (c[(c.length-1)/2]+c[(c.length-1)/2+1])%2;
    		if(e==1) {
    			double f = d+0.5d;
    			return f;
    		}
    		return d;
    	}
    }
    
    /**合并两个有序数组*/
    private int[] merge(int[] nums1,int[] nums2) {
    	int i = 0;
    	int j = 0;
    	int[] c = new int[nums1.length+nums2.length];
    	while(i<nums1.length&&j<nums2.length) {
    		if(nums1[i] < nums2[j]) {
    			c[i+j] = nums1[i];
    			i++;
    			continue;
    		}
    		c[i+j]=nums2[j];
    		j++;
    	}
    	while(i<nums1.length) {
    		c[i+j] = nums1[i];
    		i++;
    	}
    	while(j<nums2.length) {
    		c[i+j] = nums2[j];
    		j++;
    	}
    	return c;
    }
    
    public static void main(String[] args) {
//    	int[] nums1 = {1,3,5}; 
//    	int[] nums2 = {2,4,6};
    	int[] nums1 = {1,3}; 
    	int[] nums2 = {2,4};
//    	int[] nums1 = {0,0}; 
//    	int[] nums2 = {0,0};
    	Solution solu = new Solution();
    	System.out.println(solu.findMedianSortedArrays(nums1, nums2));
	}
    
}
```
