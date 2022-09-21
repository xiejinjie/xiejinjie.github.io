---
layout: post
title:  "lecode-无重复字符的最长子串"
date:   2019-08-13 20:35:54 +0800
categories: tech
---
[leetcode003](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)
<!-- more -->

## 题目
给定一个字符串，请你找出其中不含有重复字符的 最长子串 的长度。
示例

```
输入: "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
```

## 分析
这道题目可以使用滑动窗口解题。滑动窗口是指两个索引分别指向区间开始和结尾，然后动态移动两个索引确定最终区间。

- 创建两个索引（开始索引和结束索引）
- 使用Set容器存放字符，判断是否重复
- 获取结束索引字符，判断Set中是否存在
	- 存在，容器移除开始索引对应字符，开始索引加一
	- 不存在，容器添加结束索引对应字符，结束索引加一，同时判断区间大小

## 代码
```
class Solution {
    public int lengthOfLongestSubstring(String s) {
        int max = 0;
        int start = 0;
        int end = 0;
        Set set = new HashSet();
        while (start < s.length() && end < s.length()) {
            if (!set.contains(s.charAt(end))){
                set.add(s.charAt(end++));
                max = Math.max(max, set.size());
            }else {
                set.remove(s.charAt(start++));
            }

        }
        return max;
    }
}
```