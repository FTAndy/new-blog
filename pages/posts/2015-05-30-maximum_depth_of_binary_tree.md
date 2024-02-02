---
title: 'leetcode 第二题'
date: 2015/05/30
tag: leetcode
author: Andy
---

### Maximum Depth of Binary Tree
题目地址：https://leetcode.com/problems/maximum-depth-of-binary-tree/  

<!--more-->

> Given a binary tree, find its maximum depth.
> The maximum depth is the number of nodes along the longest path from the root node down to the farthest leaf node.
题目描述：找出二叉树中最深节点的深度

利用递归即可。

Ruby 版：  

```ruby
# Definition for a binary tree node.
# class TreeNode
#     attr_accessor :val, :left, :right
#     def initialize(val)
#         @val = val
#         @left, @right = nil, nil
#     end
# end

# @param {TreeNode} root
# @return {Integer}
def max_depth(root)
    root.nil? ? 0 : [max_depth(root.left) + 1, max_depth(root.right) + 1].max
end

```
