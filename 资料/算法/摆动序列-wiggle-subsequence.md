---
title: 摆动序列(wiggle-subsequence)
tags: [leetcode,刷题,贪心,状态机]
---

# 题目

如果连续数字之间的差严格地在正数和负数之间交替，则数字序列称为摆动序列。第一个差（如果存在的话）可能是正数或负数。少于两个元素的序列也是摆动序列。

例如， [1,7,4,9,2,5] 是一个摆动序列，因为差值 (6,-3,5,-7,3) 是正负交替出现的。相反, [1,4,7,2,5] 和 [1,7,4,5,5] 不是摆动序列，第一个序列是因为它的前两个差值都是正数，第二个序列是因为它的最后一个差值为零。

给定一个整数序列，返回作为摆动序列的最长子序列的长度。 通过从原始序列中删除一些（也可以不删除）元素来获得子序列，剩下的元素保持其原始顺序。

示例 1:

输入: [1,7,4,9,2,5]
输出: 6 
解释: 整个序列均为摆动序列。
示例 2:

输入: [1,17,5,10,13,15,10,5,16,8]
输出: 7
解释: 这个序列包含几个长度为 7 摆动序列，其中一个可为[1,17,10,13,10,16,8]。
示例 3:

输入: [1,2,3,4,5,6,7,8,9]
输出: 2
进阶:
你能否用 O(n) 时间复杂度完成此题?

# 代码

## CPP代码

```c++
#include <stdio.h>

#include <vector>
class Solution {
public:
    int wiggleMaxLength(std::vector<int>& nums) {
    	if (nums.size() < 2){
	    	return nums.size();
	    }
	    static const int BEGIN = 0;
	    static const int UP = 1;
	    static const int DOWN = 2;
	    int STATE = BEGIN;
	    int max_length = 1;
    	for (int i = 1; i < nums.size(); i++){
    		switch(STATE){
	    	case BEGIN:
	    		if (nums[i-1] < nums[i]){
	    			STATE = UP;
		    		max_length++;
		    	}
		    	else if (nums[i-1] > nums[i]){
	    			STATE = DOWN;
	    			max_length++;
	    		}
	    		break;
	    	case UP:
	    		if (nums[i-1] > nums[i]){
	    			STATE = DOWN;
		    		max_length++;
		    	}
		    	break;
	    	case DOWN:
	    		if (nums[i-1] < nums[i]){
	    			STATE = UP;
		    		max_length++;
		    	}
		    	break;
		    }
	    }
    	return max_length;
    }
};

int main(){
	std::vector<int> nums;
	nums.push_back(1);
	nums.push_back(17);
	nums.push_back(5);
	nums.push_back(10);
	nums.push_back(13);
	nums.push_back(15);
	nums.push_back(10);
	nums.push_back(5);
	nums.push_back(16);
	nums.push_back(8);
	Solution solve;
	printf("%d\n", solve.wiggleMaxLength(nums));
	return 0;
}
```



## java代码



```java
class Solution {
    public int wiggleMaxLength(int[] nums) {
        if(nums.length<2) {
        	return nums.length;
        }
        final int BEGIN = 0;
        final int UP = 1;
        final int DOWN = 2;
        int STATE = BEGIN;
        int max_length = 1;
        for (int i = 1; i < nums.length; i++) {//从第二个开始循环
			switch (STATE) {
			case BEGIN:
				if(nums[i-1]<nums[i]) {//前面的小于后面的,状态改为上升
					STATE = UP;
					max_length++;
				}else if(nums[i-1]>nums[i]) {
					STATE = DOWN;
					max_length++;
				}
				break;
			case UP:
				if(nums[i-1]>nums[i]) {
					STATE = DOWN;
					max_length++;
				}
				break;
			case DOWN:
				if(nums[i-1]<nums[i]) {
					STATE = UP;
					max_length++;
				}
				break;
			}
		}
    	return max_length;        
    }
}
```

