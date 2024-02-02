---
title: 'leetcode 第一题'
date: 2015/05/27
tag: leetcode
author: Andy
---
## 开始
算法很弱啊，怎么办啊，边看[《算法》](http://book.douban.com/subject/19952400/)边做题吧。听说 leetcode 的题很适合初入职场的菜鸟，就来吧。  
我把 Ruby 版的 leetcode 解题过程放在了[github](https://github.com/FTAndy/leetcode)上。

<!--more-->

## Single number
题目地址：https://leetcode.com/problems/single-number/  


>Given an array of integers, every element appears twice except for one. Find that single one.  
Note:  
>Your algorithm should have a linear runtime complexity. Could you implement it without using extra memory?
题目描述：一个数组有很多个整形数，每个数出现两次，但是只有一个数出现一次，找出那个数。  

用 Ruby 标准库的`index`和`rindex`一行搞定。  

```ruby
def single_number(nums)
    nums.each {|num| return num if nums.index(num) == nums.rindex(num) }
end
```

做完之后发现很多人倾向于用xor异或运算，也是一行搞定。  

`^`运算：


按位运算时，同为0，异为1。

法则：
`a ^ b ^ c` 与 `a ^ c ^ b` 相同。

`a ^ a = 0`，`a ^ 0 = a`

所以当所有数做异或运算时，相同的两个数最终为0，而0异或任何数为任何数，得到唯一值。  

异或版：  
```ruby
def single_number(nums)
    nums.inject {|result,num| result ^=num }
end
```
