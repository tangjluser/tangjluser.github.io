---
layout: blog_post
title: "LeetCode Hot 100 刷题反思"
date: 2026-07-20
tags:
  - LeetCode
  - 算法
  - 刷题笔记
---
# hot100 解题思路
## [两数之和](https://leetcode.cn/problems/two-sum/)

先在哈希表中找 target - x存不存在 如果不存在就把x插入哈希表

>为什么用哈希表？

因为哈希表的 查询是 o（1）的 但是哈希表里的数据是无序的 需要和 map区分开

>那为什么是插入x呢？ 

因为我是从左到右去遍历的，遍历到x的时候，表中没有 target - x 那我往后面遍历的时候 遍历到 target - x的时候 就可以查到表中有 x 这样就可以 o（n）的解决了

## [字母异位词分组](https://leetcode.cn/problems/group-anagrams/)

遇到每个单词，先按从小到大排序，然后存入哈希表，下次来就是o（1）的查询

## [最长连续序列](https://leetcode.cn/problems/longest-consecutive-sequence/)

先将每个数存入哈希表，为了去重
然后遍历哈希表，遍历一个数，就先看这个数-1在不在哈希表，不在就以它为起点，一直看这个数+1有没有，有就重复，重复的长度就是连续序列的长度

>为什么要先看 这个数-1 在不在？

为了防止重复判断，因为我每次都找的是这个连续序列的起点

## [移动零](https://leetcode.cn/problems/move-zeroes/)

设置两个指针L，R  
L用于找第一个0，找到停
R用于找第一个非0，找到停
并且 L <= R
两个都找到则交换两个位置，然后继续

## [盛最多水的容器](https://leetcode.cn/problems/container-with-most-water/)

设置两个指针，一个在最左，一个在最右
先更新答案，两个指针对应的值的最小值 * 区间长度
如何哪个小 哪个移动 （相等 随便移动哪一边）

## [三数之和](https://leetcode.cn/problems/3sum/)


因为不要求顺序，而且只是输出3元组是哪三个数，不是输出下标
所以先从小到大排序
然后从1-n遍历 第一个数x
然后从 x 之后 到 n 遍历第二个数y
然后这个时候是关键  假设现在有个 k
x + y + k = 0
然后我接下来遍历 y‘ 
但是 y’ > y 
所以 x + y‘ + k > 0 所以 k 就要变小
那一开始 k 不就可以从右往左遍历？ 就是对于 x固定的时候
所以就变成
```C++
for (int i=0;i<len;i++)
	for (int j=i+1;j<k;j++)
		while(a[i]+a[j]+a[k]>0 && k>j+1)
```
这样总体的复杂度就是 o($n^2$)

## [接雨水](https://leetcode.cn/problems/trapping-rain-water/)

本质是从动态规划的思想优化到双指针
动态规划就是 
从左到右求一遍 到i位置的 lmax 然后存到i位置
从右到左求一遍 到i位置的 rmax 然后存到i位置
然后对于每个i 那就是 lmaxi 和 rmaxi 取小 然后 - height[i]
那优化成双指针就是
L在左，R在右
然后维护 Lmax，Rmax
然后左边小更新左边
右边小更新右边 （等号依旧放左 放右 都没区别 可能争对不同的数据 有快慢区别 但是对答案没影响）
```c++
while(l<r){
            lmax = max(lmax,h[l]);
            rmax = max(rmax,h[r]);
            if(lmax < rmax){
                ans += (lmax - h[l]);
                l++;
            }else {
                ans += (rmax - h[r]);
                r--;
            }
        }
```


## [无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/)

核心在于  假设我现在找到一个 字串了
左端点是 L 右端点是R
如果右边的下一个 和里面不重复 那肯定加进来
那如果右边的重复了，我只需要 L 右移就可以了
[L , R] 不重复  - > [L+1 , R] 也肯定是不重复的，然后就可以继续判断了 整体的复杂度就是 o(n)了
为了高校的实现判断 新的这个字母在不在 这个区间内，我们用到哈希集合
```C++
unordered_set<char> mp;
if(i!=0){
	mp.erase(s[i-1]);
}
while(r+1<len && mp.count(s[r+1])==0){
	mp.insert(s[r+1]);
	r++;
}
ans = max(ans,r-i+1);
```
