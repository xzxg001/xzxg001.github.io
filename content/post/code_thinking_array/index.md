---
title: Code_with_thinking:Array 
date: 2025-4-25
description: a coding experiment for array
image: image.png
tags: 
    - array
    - coding
categories:
    - array
    - coding
weight: 1
math: true
---
# 代码随想录_数组

## 1.数组：二分查找

1>**前提是数组为有序数组**，同时题目还强调**数组中无重复元素**，因为一旦有重复元素，使用二分查找法返回的元素下标可能不是唯一的，

2>判断两个**边界条件**：$$while(left< or <=right)$$以及$$if(num[middle]>target)right=middle-1/middle$$

**循环不变量**：[left,right] or [left,right)

```c++
//假设left=0 right=num.size-1(包括右边界)
//[]
while(left<=right){middle=(left+right)/2 

if(num[middle]>target)right=middle-1;

else if(num[middle]<target)left=middle+1;

else return middle;

}

return -1;
```

```c++
//假设left=0 right=num.size(不包括右边界)
//[)
while(left<right){middle=(left+right)/2 

if(num[middle]>target)right=middle;

else if(num[middle]<target)left=middle+1;

else return middle;

}

return -1;
```

3>$$middle = left + ((right - left) >> 1);$$

4>同学对于二分法都是**一看就会，一写就废**？

其实主要就是对**区间的定义没有理解清楚**，在循环中没有始终坚持根据查找区间的定义来做边界处理。

区间的定义就是不变量，那么在循环中坚持根据查找区间的定义来做边界处理，就是循环不变量规则。

## 2.数组：移除元素（双指针）

等同于`vector`容器中erase的操作

1>暴力实现：两重for循环

2>双指针解决：时间复杂度：O(n),使用快慢指针。

快指针指向需要删除的元素，慢指针指向新数组需要的元素。主要思路是覆盖

```c++
// 时间复杂度：O(n)
// 空间复杂度：O(1)
class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        int slowIndex = 0;//慢指针指向新数组需要的元素
        for (int fastIndex = 0; fastIndex < nums.size(); fastIndex++) {
            if (val != nums[fastIndex]) {
                nums[slowIndex++] = nums[fastIndex];//快指针指向需要删除的元素
            }
        }
        return slowIndex;
    }
};

```

## 3.数组：有序数组的平方（双指针）

1>暴力排序 平方+排序 O(nlogn)

2>双指针法 从两边开始不断比较两端平方的大小，放入result数组



## 4.数组：长度最小的子数组（滑动窗口）

要求$$sum>=s$$
找到长度最小的子数组

1>暴力：两重for循环，确定起始和终止位置

2>滑动窗口（双指针）使用j来遍历终止位置，然后移动起始位置确定最小长度数组

窗口就是 满足其和 ≥ s 的长度最小的 连续 子数组。

窗口的起始位置如何移动：如果当前窗口的值**大于等于s**了，窗口就要向前移动了（也就是该缩小了）。

窗口的结束位置如何移动：窗口的结束位置就是遍历数组的指针，也就是for循环里的索引。

**滑动窗口的精妙之处在于根据当前子序列和大小的情况，不断调节子序列的起始位置。从而将O(n^2)暴力解法降为O(n)。**

```C++
class Solution {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
        int sum=0;
        int i=0;
        int ans=INT_MAX;
        for(int j=0;j<nums.size();j++){
            sum+=nums[j];
            while(sum>=target){
                ans=min(ans,j-i+1);
                sum-=nums[i++];
            }
        }
        return (ans==INT_MAX)?0:ans;
    }
};
```

当然，评论区也有人说用前缀和来做，然后用二分查找来遍历寻找，最终时间复杂度$$O(nlogn)$$

## 5.数组：螺旋矩阵

1>循环不变量:( ]  [ )  首选[ )

2>模拟顺时针画矩阵的过程:

- 填充上行从左到右
- 填充右列从上到下
- 填充下行从右到左
- 填充左列从下到上

由外向内一圈一圈这么画下去。

其实**真正解决题目的代码都是简洁的，或者有原则性的**

3>掌握好边界条件进行**模拟**即可，重点是需要理解变量的含义及定义，绕圈法

```c++
class Solution {
public:
    vector<vector<int>> generateMatrix(int n) {
        vector<vector<int>> a(n,vector<int>(n,0));
        int offset=1;
        int count=1;
        int loop=n/2;
        int startx=0,starty=0;
        while(loop--){
            int i=startx,j=starty;
            for(j;j<n-offset;j++)a[i][j]=count++;
            for(i;i<n-offset;i++)a[i][j]=count++;
            for(;j>starty;j--)a[i][j]=count++;
            for(;i>startx;i--)a[i][j]=count++;

            startx++;
            starty++;
            offset++;
        }
        if(n%2==1)a[n/2][n/2]=count;
        return a;
    }
};
```

4>可以通过严格控制边界，逐渐缩圈来解决

```c++
class Solution {
public:
    vector<int> spiralOrder(vector<vector<int>>& matrix) {//这种方法是设置了边界条件，移动完之后判断边界情况
    //直到超过循环位置停止
        int w=0,s=matrix.size()-1,a=0,d=matrix[0].size()-1;
        vector<int> ans;
        while(1){
            for(int i=a;i<=d;i++)ans.push_back(matrix[w][i]);
            if(++w>s)break;
            for(int j=w;j<=s;j++)ans.push_back(matrix[j][d]);
            if(--d<a)break;
            for(int i=d;i>=a;i--)ans.push_back(matrix[s][i]);
            if(--s<w)break;
            for(int j=s;j>=w;j--)ans.push_back(matrix[j][a]);
            if(++a>d)break;
        }
        return ans;
    }
};
```





























