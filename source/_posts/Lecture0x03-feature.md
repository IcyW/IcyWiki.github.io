---
title: Lecture0x03_feature engineering
toc: true
date: 2020-03-11 19:25:33
tags: [Machine Learning]
categories: Machine Learning
---

# 数据预处理与特征工程
## Data PreProcessing（数据预处理）

### 1. 无量纲化
#### 标准化(Standardization)

**适用场景：**
- 某些算法要求样本具有**零均值和单位方差**；
- 需要消除样本不同属性具有**不同量级**时的影响；
- 在分类、聚类算法中，需要使用距离来度量相似性的时候（如SVM、KNN），或者使用PCA技术进行降维的时候，Z-score表现更好；。

**做法：**

- 基于原始数据的均值（mean）和标准差（standard deviation）进行数据的标准化。
- z-score标准化：

$$x' =  \frac{x-\mu}{\sigma} $$

其中 $\mu = \frac{1}{N} \sum_{i=1}^{N}x_i$， $\sigma = \sqrt{\frac{1}{N}\sum_{i=1}^{N}}$

- 对于属性A，将其原始值x使用z-score标准化到x’。适用于属性的最大值和最小值未知、或有超出取值范围的离群数据的情况。

**优点：**

- z-score最大的优点就是简单，容易计算，Z-Score能够应用于数值型的数据，并且不受数据量级的影响，因为它本身的作用就是消除量级给分析带来的不便。

**缺点：**

- Z-Score对于数据的分布有一定的要求，正态分布是最有利于Z-Score计算的；
- Z-Score消除了数据具有的实际意义，因此Z-Score的结果只能用于比较数据间的结果，关注数据的真实意义还需要还原其原值；
- 如果存在异常值，则无法保证平衡的特征尺度。

#### 归一化(Normalization)

**适用场景：**

- 归一化有可能提高精度——数量级的差异将导致量级较大的属性占据主导地位，但实际情况有可能是值域范围小的特征更重要；
- 数量级的差异将导致迭代收敛速度减慢——当使用梯度下降法寻求最优解时，很有可能走“之字型”路线（垂直等高线走），从而导致需要迭代很多次才能收敛；
- 依赖于**样本距离**的算法对于数据的数量级非常敏感，务必归一化。

**做法：**

1. MinMax归一化

   - 区间缩放法利用了边界值信息，将属性缩放到[0,1]。

     $$x' =  \frac{x-Min}{Max-Min} $$

   **缺陷：**

   - 当有新数据加入时，可能导致max和min的变化，需要重新定义；
   - MinMaxScaler对异常值的存在非常敏感。

3. MaxAbs归一化

  - 单独地缩放和转换每个特征，使得训练集中的每个特征的最大绝对值将为1.0，将属性缩放到[-1,1]。它不会移动/居中数据，因此不会破坏任何稀疏性。

     $$x' =  \frac{x}{|Max|} $$
  
   **缺陷：**
  
   - 当有新数据加入时，可能导致max和min的变化，需要重新定义；
   - 在仅有正数据时，该缩放器的行为MinMaxScaler与此类似。
   - MaxAbsScaler也对异常值的存在非常敏感。

#### 正态分布化
——也是Normalization中的一种。

**适用场景：**

- 正则化的过程是将每个样本缩放到单位范数(每个样本的范数为1)，如果要使用如二次型(点积)或者其它核方法计算两个样本之间的相似性，这个方法会很有用。

- 该方法是文本分类和聚类分析中经常使用的——向量空间模型（Vector Space Model)的——基础。

**做法：**

- 主要思想是对每个样本计算其$L_{p}$范数，然后对该样本中每个元素**除以该范数**，这样处理的结果是使得每个处理后样本的$L_{p}$ 范数(L1-norm, L2-norm)等于1。比如：

  $$x' =  \frac{x}{L_{2}-norm } $$

**$L_{p}$范数：**

- 在线性代数，函数分析等数学分支中，范数（Norm）是一个函数，其赋予某个向量空间（或矩阵）中的每个向量以长度或大小。

1. $L_{0}-norm$ 

   $||w||_{0} = num(i)$   $with$    $ x_{i}  \neq 0 $ （表示向量中所有非零元素的个数）

2. $L_{1}-norm$ 

   $$||w||_{1} = \sum_{1}^{d}|x_i| $$ （表示每个元素的绝对值之和）

3. $L_{2}-norm$ 

   $$||w||_{2} = \sqrt{\sum_{i=1}^{d}(x_j)^2} $$  （欧氏距离）

4. $L_{p}-norm$ 

   $$||w||_{p} = ({\sum_{i=1}^{d}x_j^p})^{\frac{1}{p}} $$  

5. $L_{max}-norm$ （对象属性之间的最大距离，）

   $$||w||_{\infty} = max（│x_1│，│x_2│，…，│x_n│）$$

可以看到sklearn中的实现细节：

```
if norm == 'l1':
    norms = np.abs(X).sum(axis=1)
elif norm == 'l2':
    norms = row_norms(X)
elif norm == 'max':
    norms = np.max(X, axis=1)
norms = _handle_zeros_in_scale(norms, copy=False)
X /= norms[:, np.newaxis]
```
#### 对比
1、归一化是为了消除纲量压缩到[0,1]区间，标准化只是调整特征整体的分布；

2、归一化输出在[0,1]或[-1,1]之间，标准化无限制。

3、如果对输出结果范围有要求，用归一化；如果数据较为稳定，不存在极端的最大最小值，用归一化；

4、如果数据存在异常值和较多噪音，用标准化，可以间接通过中心化避免异常值和极端值的影响。

5、基于树的方法不需要进行特征的归一化。

- 一般来说，建议优先使用标准化。对于输出有要求时再尝试别的方法，很多方法都可以将输出范围调整到[0, 1]；如果我们对于数据的分布有假设的话，更加有效的方法是使用相对应的概率密度函数来转换。

- 除了上面介绍的方法外，还有一些相对没这么常用的处理方法：RobustScaler、PowerTransformer、QuantileTransformer和QuantileTransformer等。

### 2. 特征分桶



### 3. 统计变换

### 4. 特征编码

## Feature Extraction（特征提取）

## Feature Selection（特征选择）


## Feature construction（特征构造）



# 附录

Ref. 1 木东居士 / [​特征工程系列：特征预处理（上）](https://mp.weixin.qq.com/s/qWO9zgKyntvyWfftpGqrHQ)

Ref. 2 木东居士 / [机器学习的敲门砖：归一化与KD树](https://mp.weixin.qq.com/s?__biz=MzI4MjkzNTUxMw==&mid=2247483857&idx=3&sn=5a4573e5fe074241a45f6affb969448f)

Ref. 3 是安酱和菜菜呀 / [sklearn中的数据预处理和特征工程](https://www.cnblogs.com/juanjiang/archive/2019/05/30/10948849.html)

Ref.4 Dave / [ML 入门：归一化、标准化和正则化](https://zhuanlan.zhihu.com/p/29957294)

