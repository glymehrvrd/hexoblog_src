---
title: Leetcode No.35 Sort Colors
date: 2016-03-09 15:50:20
categories: technology
tags:
- Leetcode
- Algorithm
---

[Sort Colors](https://leetcode.com/problems/sort-colors/)是一个排序问题，但是元素类型只有三种（RWB），如果类型只有两种的话，很容易可以想到使用 quick sort 中的一次 partition 来实现。三种元素的排序可以使用两次 partition 来实现，但是时间复杂度就超过了 N，而题目要求 One-Pass 运算完毕。

<!-- more -->

这里添加使用两个指针 red, blue，red对应 partition 中的 small 指针，red 指针总是指向最后一个 red 元素，blue 指针总指向最后一个 blue 元素。初始化时， red 指向 left-1， blue 指向 right+1。当 red 和 blue 都被聚在头和尾时， white 也被聚集在中间，完成三种元素的排序。

代码如下，注意当每次交换元素后，如果交换过来的元素不是 white，则需要对该点再次进行分类。可以看到每个点都只被判断一次就交换到正确的位置，因此实现了 One-pass 的要求。
```c++
class Solution {
public:
    void sortColors(vector<int>& nums) {
        int red=-1;
        int blue=nums.size();
        for(int i=0;i<nums.size();++i){
            if(i>=blue)
                break;
            while(nums[i]!=1) // not white
            {
                if(nums[i]==0){
                    red++;
                    if(red==i){
                        break;
                    }
                    else{ // do exchange
                        nums[i]=nums[red];
                        nums[red]=0;
                    }
                }
                else{
                    blue--;
                    if(blue==i){
                        break;
                    }
                    else{ // do exchange
                        nums[i]=nums[blue];
                        nums[blue]=2;
                    }
                }
            }
        }
    }
};
```
