---
title: 跳跃游戏(jump-game)
tags: [leetcode,刷题,贪心]
---

# 题目

给定一个非负整数数组，你最初位于数组的第一个位置。

数组中的每个元素代表你在该位置可以跳跃的最大长度。

判断你是否能够到达最后一个位置。

示例 1:

输入: [2,3,1,1,4]
输出: true
解释: 从位置 0 到 1 跳 1 步, 然后跳 3 步到达最后一个位置。
示例 2:

输入: [3,2,1,0,4]
输出: false
解释: 无论怎样，你总会到达索引为 3 的位置。但该位置的最大跳跃长度是 0 ， 所以你永远不可能到达最后一个位置。

# 思路与代码



## CPP代码

```c++
#include <stdio.h>

#include <vector>
class Solution {
public:
    bool canJump(std::vector<int>& nums) {
        std::vector<int> index;
        for (int i = 0; i < nums.size(); i++){
        	index.push_back(i + nums[i]);
        }
        int jump = 0;
        int max_index = index[0];
        while(jump < index.size() && jump <= max_index){
        	if (max_index < index[jump]){
	        	max_index = index[jump];
	        }
        	jump++;
        }
        if (jump == index.size()){
        	return true;
        }
        return false;
    }
};

int main(){
	std::vector<int> nums;
	nums.push_back(2);
	nums.push_back(3);
	nums.push_back(1);
	nums.push_back(1);
	nums.push_back(4);
	Solution solve;
	printf("%d\n", solve.canJump(nums));
	return 0;
}

```

