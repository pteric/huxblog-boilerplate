---
layout:     post
title:      "50+ 精选数据结构和算法面试问题 【译】"
subtitle:   "附带算法对应解答"
date:       2018-10-13
author:     "PengTuo"
catalog:    true
header-img: "img/post-bg-algo-01.jpg"
categories: [Base Algorithms]
tags:
    - Algorithms
---
本文概览：
* TOC
{:toc}

> 原文链接：[https://hackernoon.com/50-data-structure-and-algorithms-interview-questions-for-programmers-b4b1ac61f5b0](https://hackernoon.com/50-data-structure-and-algorithms-interview-questions-for-programmers-b4b1ac61f5b0)
>

> 本文的描述语言为 Java


有许多计算机科学专业毕业生或者程序猿会申请研发职位，在这个行业里有很多待遇比较好的大公司，例如国内的BAT，头条，美团等，国外也有亚马逊，谷歌以及 Facebook 等大型公司，但他们中的许多人都不知道当你在这些公司申请工作时会**遇到什么样的编程面试问题**。

在本文中，我将与不同经验水平的程序猿分享一些*常见的编程面试问题*，涉及**从刚从大学毕业的人到具有一到两年经验的程序员**。

研发面试主要由 [数据结构和基于算法的问题](http://www.java67.com/2018/06/data-structure-and-algorithm-interview-questions-programmers.html) 组成，也会有一些逻辑性的面试问题，例如 *如何在不使用临时变量的情况下交换两个整数？*

我认为将算法问题划分到不同的领域是非常有帮助的。我在面试中经常遇见的领域有数组、链表、字符串、二叉树，还有算法问题（例如字符串算法问题、排序算法问题，或者其他），并且这些都会在本文中提及。

我不能保证本文中的数据结构问题或者算法问题肯定会被问及，但是这些会让你充分了解实际面试中的遇到的各种问题以及解题思路。

一旦充分学习了这些问题，你应该有足够的信心参加任何电话面试或者线下面试。

顺便说一下，如果你没有认真学习过基本的数据结构和算法，或者你没有多年触及它们，那么还是建议你先去补充一下基础知识，再来研究本文的问题。

## Top 50 算法和编程面试问题
话不多说，这里列出我在研发工作以及面试中一些最常见的算法面试问题列表：

#### 1. 数组类面试问题
数组是最基本的数据结构，它将元素存储在连续的内存位置。这也是面试官很喜欢面试的方向，你会在任何算法面试中听到很多关于数组的问题，例如：反转数组，排序数组或搜索数组上的元素。

数组数据结构最大的优点是当知道索引时能够在 `O(1)` 的时间复杂度内搜索，但是向数组数据结构里添加和移除元素时会非常慢，因为数组一旦创建就不能更改其规格。当需要更改数组的大小时，则需要创建一个新的数组，然后从原先的数组中将元素拷贝到新的数组中。

解决基于数组类的算法问题的关键是要充分了解数组数据结构以及基本的编程技能，例如循环、递归等。

以下列出非常受欢迎的基于数组类的算法面试问题供你练习：

1. **如何在一个给定的，范围是 1 到 100 的乱序整数数组中找到缺失的值？[solution](https://javarevisited.blogspot.com/2014/11/how-to-find-missing-number-on-integer-array-java.html)**
2. **如何在一个给定的整数数组中找到重复的数字？[solution](https://javarevisited.blogspot.com/2014/01/how-to-remove-duplicates-from-array-java-without-collection-API.html)**
3. **如何在一个乱序的整数数组中找到最大值与最小值？[solution](http://www.java67.com/2014/02/how-to-find-largest-and-smallest-number-array-in-java.html)**
4. **如何找到所有的整数对，其总和等于给定的数字？[solution](https://javarevisited.blogspot.com/2014/08/how-to-find-all-pairs-in-array-of-integers-whose-sum-equal-given-number-java.html)**
5. **如何在包含多个重复项的数组中找到重复的数字？[solution](https://javarevisited.blogspot.com/2014/03/3-ways-to-find-first-non-repeated-character-String-programming-problem.html)**
6. **如何用 Java 语言在一个给定的数组中移除重复项？[solution](https://javarevisited.blogspot.com/2014/01/how-to-remove-duplicates-from-array-java-without-collection-API.html)**
7. **请利用快速排序对一个数组进行**原地**排序。[solution](https://javarevisited.blogspot.com/2014/08/quicksort-sorting-algorithm-in-java-in-place-example.html)**
8. **如何用 Java 在原地反转一个数组？[solution](https://javarevisited.blogspot.com/2013/03/how-to-reverse-array-in-java-int-String-array-example.html)**
9. **请了解数组问题中的对撞指针和滑动窗口技巧**

这些问题不仅能够帮助你提高解决问题的能力，还能更好的帮助你理解数组这个数据结构。如果你觉得这几道题不够联系，则可以查看 [30 array question](https://javarevisited.blogspot.com/2015/06/top-20-array-interview-questions-and-answers.html)，也可以在 [Leetcode](https://leetcode.com/)或者[Lintcode](https://www.lintcode.com/)上的数组标签下找题目。

#### 2. 链表类面试题
[链表](http://www.java67.com/2017/06/5-difference-between-array-and-linked.html)是另一种非常常见的数据结构。与数组结构类似，链表也是线性数据结构并且以线性的方式存储。

然而，和数组数据结构不同的是，在内存物理位置上数据元素并不是相邻的，相反，数据是分散存储在内存中，通过节点与其他数据相连。链表就是一系列的节点，其中每个节点存储着数据值和下一个节点的地址。

由于这种结构，在链表中添加和删除元素很容易，因为您只需要更改链接而不是创建数组，但搜索很困难，在单链表中通常需要花费 `O(n)` 时间来查找元素。

链表还有其他的变种，例如双向链表，允许在链表上两个方向（向前或向后）游走；还有循环链表，链表的首尾相连组合成一个圈。

为了解决基于链表的问题，需要对递归很了解，因为链表是[递归数据结构](https://javarevisited.blogspot.com/2017/03/how-to-reverse-linked-list-in-java-using-iteration-and-recursion.html)。如果从链表中获取一个节点，则剩余的数据结构仍然是链表，因此，许多链表问题具有比迭代解决方案更简单的递归解决方案。

以下列出一些常见的受欢迎的链表的面试问题：

1. **如何遍历一次就找到链表的中间节点？[solution](https://javarevisited.blogspot.com/2012/12/how-to-find-middle-element-of-linked-list-one-pass.html)**
2. **如何判断一个链表是否有环？如何找到这个环的起始节点？[solution](https://javarevisited.blogspot.com/2013/05/find-if-linked-list-contains-loops-cycle-cyclic-circular-check.html)**
3. **如何反转一个链表？[solution](http://www.java67.com/2016/07/how-to-reverse-singly-linked-list-in-java-example.html)**
4. **如何不使用递归的情况下反转链表？[solution](https://javarevisited.blogspot.com/2017/03/how-to-reverse-linked-list-in-java-using-iteration-and-recursion.html)**
5. **如何从未排序的链表中移除重复节点？[solution](https://www.geeksforgeeks.org/remove-duplicates-from-an-unsorted-linked-list/)**
6. **请求出一个单链表的长度。[solution](https://javarevisited.blogspot.com/2016/05/how-do-you-find-length-of-singly-linked.html)**
7. **如何找出单链表的倒数第三个节点？[solution](https://javarevisited.blogspot.com/2016/07/how-to-find-3rd-element-from-end-in-linked-list-java.html)**
8. **请对两个链表表示的整数求和，并返回链表。[solution](https://www.geeksforgeeks.org/sum-of-two-linked-lists/)**

这些问题将帮助您提高解决问题的能力，并提高您对链表数据结构的了解。如果对于以上这些问题感觉比较吃力，建议先补充链表的基础知识。你也可以联系[**30 linked list interview questions**](http://javarevisited.blogspot.sg/2017/07/top-10-linked-list-coding-questions-and.html) 

#### 3. 字符串类面试问题
与数组和链表数据结构同样，字符串类型是面试中的另一个热门话题。关于字符串问题的好消息是，如果你懂数组，你可以很容易地解决基于字符串的问题，因为字符串只是一个字符数组。

所以所有你的基于数组类问题的解题思路与技巧都可以用来解决字符串类型的问题。

以下是我列出的在工作面试中会被频繁问及的字符串类型问题：

1. **如何将字符串中重复的字符打印出来？[solution](http://java67.blogspot.sg/2014/03/how-to-find-duplicate-characters-in-String-Java-program.html)**
2. **如何判断两个乱序的字符串中的字母都相同？[solution](https://javarevisited.blogspot.com/2013/03/Anagram-how-to-check-if-two-string-are-anagrams-example-tutorial.html)**
3. **请将字符串中第一个未重复的字符打印出来。[solution](https://javarevisited.blogspot.com/2014/03/3-ways-to-find-first-non-repeated-character-String-programming-problem.html)**
4. **如何利用递归翻转一个字符串？[solution](https://javarevisited.blogspot.com/2012/01/how-to-reverse-string-in-java-using.html)**
5. **如何判断一个字符串中只包含数字字符？[solution](https://javarevisited.blogspot.com/2012/10/regular-expression-example-in-java-to-check-String-number.html)**
6. **请统计一个字符串中的元音和辅音的个数。[solution](http://www.java67.com/2013/11/how-to-count-vowels-and-consonants-in-Java-String-word.html)**
7. **如何计算字符串中给定字符的出现次数？[solution](https://javarevisited.blogspot.com/2012/12/how-to-count-occurrence-of-character-in-String.html)**
8. **请输出给定字符串的所有排列组合。[solution](https://javarevisited.blogspot.com/2015/08/how-to-find-all-permutations-of-string-java-example.html)**
9. **如何翻转给定句子中的单词？[solution](http://www.java67.com/2015/06/how-to-reverse-words-in-string-java.html)**
10. **如何判断两个字符串互为旋转？[solution](http://www.java67.com/2017/07/string-rotation-in-java-write-program.html)**
11. **如何判断一个字符是回文字符？[solution](http://www.java67.com/2015/06/how-to-check-is-string-is-palindrome-in.html)**

这些问题有助于提高您对字符串作为数据结构的了解。 如果你可以在没有任何帮助的情况下解决所有这些`String`问题，那么你很棒棒。

如果你需要更多的练习，可以看[这里](http://javarevisited.blogspot.sg/2015/01/top-20-string-coding-interview-question-programming-interview.html)

#### 4. 二叉树面试问题
到目前为止，我们只研究了线性数据结构，但现实世界中的所有信息都无法以线性方式表示，而这正是树数据结构所帮助的地方。

树数据结构是一种数据结构，允许以分层方式存储数据。 根据存储数据的方式，有不同类型的树，例如[二叉树](https://javarevisited.blogspot.com/2016/07/binary-tree-preorder-traversal-in-java-using-recursion-iteration-example.html)，其每个节点最多有两个子节点。以及其近亲，[二叉搜索树](https://javarevisited.blogspot.com/2017/04/recursive-binary-search-algorithm-in-java-example.html)，这个是最受欢迎的树形数据结构了。因此你会发现有很多的面试问题都是基于二叉树和二叉搜索树，例如 如何遍历，计算节点个数，找到树深度或者判断其是否为平衡树等。

解决二叉树问题的一个关键点是需要很扎实的理论基础，例如： 什么是二叉树的大小或深度，什么是叶子，什么是节点，以及对流行的遍历算法的理解，例如前中后序遍历。

以下是流行的基于二叉树的面试问题列表：
1. **如何实现二叉搜索树？[solution](https://javarevisited.blogspot.com/2015/10/how-to-implement-binary-search-tree-in-java-example.html#axzz4wnEtnNB3)**
2. **请前序遍历一颗二叉树。[solution](https://javarevisited.blogspot.com/2016/07/binary-tree-preorder-traversal-in-java-using-recursion-iteration-example.html#axzz5ArdIFI7y)**
3. **请不使用递归的情况下前序遍历一颗二叉树。[solution](http://www.java67.com/2016/07/binary-tree-preorder-traversal-in-java-without-recursion.html)**
4. **请中序遍历一颗二叉树。[solution](http://www.java67.com/2016/08/binary-tree-inorder-traversal-in-java.html)**
5. **请在不使用递归的情况下中序遍历一颗二叉树并输出。[solution](http://www.java67.com/2016/08/binary-tree-inorder-traversal-in-java.html)**
6. **如何实现后序遍历算法？[solution](http://www.java67.com/2016/10/binary-tree-post-order-traversal-in.html)**
7. **请在不使用递归的情况下后序遍历一颗二叉树。[solution](http://www.java67.com/2017/05/binary-tree-post-order-traversal-in-java-without-recursion.html)**
8. **如何输出一颗二叉搜索树的所有叶子结点？[solution](http://www.java67.com/2016/09/how-to-print-all-leaf-nodes-of-binary-tree-in-java.html)**
9. **如何统计一颗二叉树的所有叶子节点的个数。[solution](https://javarevisited.blogspot.com/2016/12/how-to-count-number-of-leaf-nodes-in-java-recursive-iterative-algorithm.html)**
10. **如何在一个数组中执行二分搜索。[solution](https://javarevisited.blogspot.com/2015/10/how-to-implement-binary-search-tree-in-java-example.html#axzz4wnEtnNB3)**


#### 5. 各式面试问题
除了基于数据结构的问题之外，大多数面试中还会询问算法，设计，位操作和基于逻辑的一般问题，我将在本节中对其进行描述。

1. **如何实现冒泡排序？[solution](https://javarevisited.blogspot.com/2014/08/bubble-sort-algorithm-in-java-with.html#axzz5ArdIFI7y)**
2. **如何实现迭代快速排序算法？[solution](https://javarevisited.blogspot.com/2016/09/iterative-quicksort-example-in-java-without-recursion.html#axzz5ArdIFI7y)**
3. **如何实现插入排序算法？[solution](http://www.java67.com/2014/09/insertion-sort-in-java-with-example.html)**
4. **如何实现归并排序算法？[solution](http://www.java67.com/2018/03/mergesort-in-java-algorithm-example-and.html)**
5. **如何实现桶排序算法？[solution](https://javarevisited.blogspot.com/2017/01/bucket-sort-in-java-with-example.html)**
6. **如何实现计数排序算法？[solution](http://www.java67.com/2017/06/counting-sort-in-java-example.html)**
7. **如何实现基数排序算法？[solution](http://www.java67.com/2018/03/how-to-implement-radix-sort-in-java.html)**
8. **如何在不使用第三个变量的情况下交换两个数字？[solution](http://www.java67.com/2015/08/how-to-swap-two-integers-without-using.html)**
9. **如何判断两个矩形是否互相重叠？[solution](https://javarevisited.blogspot.com/2016/10/how-to-check-if-two-rectangle-overlap-in-java-algorithm.html)**
10. **如何设计自动售货机？[solution](https://javarevisited.blogspot.com/2016/06/design-vending-machine-in-java.html)**

顺便说一下，你在实践中解决的问题越多，你的准备就越好。 因此，如果您认为 50 还不够而且您需要更多，那么请查看这些额外的[50个编程问题](https://javarevisited.blogspot.com/2015/02/50-programmer-phone-interview-questions-answers.html)，以便进行更全面的准备。

## 现在，你应该准备好了
这些是数据结构和算法之外的一些最常见的问题，可以帮助您在面试中做得很好。这些常见的数据结构和算法问题是您在任何级别的技术面试都需要了解的问题。

如果您正在寻找开发工作，您可以使用此这些问题列表开始准备。此列表提供了准备的好主题，也有助于评估您的准备工作，以找出您的优势和劣势领域。熟悉数据结构和算法对于面试成功非常重要。

