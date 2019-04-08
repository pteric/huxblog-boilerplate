---
layout:     post
title:      "Adaboost Algorithm"
subtitle:   ""
date:       2019-04-02
author:     "PengTuo"
catalog:    true
header-img: "img/post-bg-algo-01.jpg"
categories: [Towards Data Science]
tags:
    - Algorithms
---

## 数学原理

#### 1. 初始化数据集权重
给定一个数据集：
<center> $$D=\{(x^{(1)}, y^{(1)}), (x^{(2)}, y^{(2)}) … (x^{(n)}, y^{(n)})\}$$ </center>

并为数据集初始化一个对应的权重分布 $W_{k,i}$，得：
<center>$$W_{k,i} = (w_{k,1},  w_{k,2} …   w_{k,n})$$</center>

初始时设置 $k=1$，且初始时权重向量中的每一项的值为 $$w_{1i}=\frac{1}{n}, i=1,2…n$$

#### 2. 学习基本分类器
对具有权重分布的数据集 $D$ 进行学习，得到基本分类器，记为 $G_k(x):X \to \{-1,1\}$

并且标记此次基本分类器的预测结果为 $G_k(x^{(i)}), i\in (1,2…n)$，仍然初始时 $k=1$

#### 3. 计算误差与基本分类器权重
计算基本分类器对应的误差 $e_k$ ：
 <center>$$e_k = \sum_{i=1}^nw_{k,i}I(G_k(x^{(i)})\neq y^{(i)})$$</center>

然后根据误差计算基本分类器的 $G_k(x)$ 的权重值 $\alpha_k$ :
<center>$$\alpha_k = \frac{1}{2}ln(\frac{1-e_k}{e_k})$$</center>

#### 4. 更新 k+1 轮数据权重
根据第 $k$ 轮的基本分类器 $G_k(x)$ 的权重值更新第 $k+1$ 轮的数据集权重 $W_{k+1,i}$：
<center>$$w_{k+1,i}=\frac{w_k,i}{Z_k}exp(-\alpha_k*y^{(i)}*G_k(x^{(i)}))$$</center>

其中 $Z_k$ 为规范化因子，为的是将权重值映射到区间$(0,1)$
<center>$$Z_k = \sum_{i=1}^nw_{k,i}*exp(-\alpha_k*y^{(i)}*G_k(x^{(i)}))$$</center>
   
#### 5. 得到最终分类器
进行了 $K$ 轮的更新后，得到基本分类器的线性组合
<center>$$f(x) = \sum_{k=1}^K\alpha_k*G_k(x)$$</center>

最终的分类器即为： $F(x) = sign(f(x)) = \sum_{k=1}^K\alpha_k*G_k(x)$


## Python3 实现
```python3

```