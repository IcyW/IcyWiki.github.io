---
title: Lecture0x01_knn
toc: true
date: 2020-02-24 12:10:42
tags: [Machine Learning, Algorithm]
categories: Machine Learning
---

> ​        前言：回想起来，距离第一次接触Machine Learning、做project、在组会上做分享，已经过了整整三年，前沿技术一直在发展，甚至YOLO之父 Joseph Redmon都宣布要退出CV界，而我的知识储备却并未增加。为了避免关键时刻生疏，今年初跟随饼干的学习小组，重温机器学习的轶事。


# 1. KNN若只如初见

​        鉴于已经不是第一次与knn约会了，不多寒暄直入主题，给它发一个身份卡（持续更新）。

| 名称  |  中文  |  分类模型  |  回归模型  |  关注参数  |  适用场景| 必备技巧 |
|  ----  |  ----  |  ----  |  ----  |  ----  |  ----  |  ----  |
| K-Nearest Neighbor  | k最近邻算法 | Yes | Yes | k, 相似度距离算法 |特征维数低，特征以连续值为主，样本量小 |归一化|


# 2. 建模过程


## 2-1. 数据集
​        恰逢疫情时段，我便尝试拿一份公开的疫情人数[数据]( https://lab.isaaclin.cn/nCoV/)来分析一下，做一个简单的、可能并不太合适的建模。
<img src="http://myimg.icyhh.xyz/ml/01.png?imageView2/format/webp" >

​        数据和特征决定了机器学习的上限，而模型和算法只是逼近这个上限。对原始数据的预处理对模型结果的优劣会有关键影响，这里我先对数据集做一个简单分析：

> provinceName (string): 中文省名
> provinceEnglishName (string): 英文省名
> province_zipCode (string): 省邮政编码
> cityName (string): 中文城市名
> cityEnglishName (string): 英文城市名
> city_zipCode (string): 城市邮政编码
> province_confirmedCount (int): 省确诊人数
> province_suspectedCount (int): 省疑似人数
> province_curedCount  (int): 省治愈人数
> province_deadCount (int): 省死亡人数
> city_confirmedCount (int): 市确诊人数
> city_suspectedCount (int): 市疑似人数
> city_curedCount (int): 市治愈人数
> city_deadCount (int): 市死亡人数
> updateTime: 更新时间

​        假设要预测的对象是“市死亡人数”，其他特征都为他打辅助。人数是连续值，这里需要用一个回归模型。
​        显而易见，省名、英文省、邮政编码互相之间交叉熵为零，相似度100%，三个特征一起用对结果预测没有增强作用，反而冗余，所以三选一即可。先以最简单地方式处理，留下邮政编码，去除名字特征。

​       特征X：

> province_zipCode (string): 省邮政编码
> city_zipCode (string): 城市邮政编码
> province_confirmedCount (int): 省确诊人数
> province_suspectedCount (int): 省疑似人数
> province_curedCount  (int): 省治愈人数
> province_deadCount (int): 省死亡人数
> city_confirmedCount (int): 市确诊人数
> city_suspectedCount (int): 市疑似人数
> city_curedCount (int): 市治愈人数

​       预测对象Y：
> city_deadCount (int): 市死亡人数

​        邮政编码在这份表里有缺失，数据预处理需要先对它做个简单的**数据填充**（其实为避免特征噪声，数据存在的情况下对城市名进行重编码会更准确；迫不得已才做这样的数据填充）。

<img src="http://myimg.icyhh.xyz/ml/02.png?imageView2/format/webp" >
​        todo. 可以改进的点有：改变非连续值特征的编码 / 对不同量纲的特征数据做归一化。

## 2-2. 训练模型
​        将数据集切分为训练集测试集，以便验证是否有**过拟合**的问题。这里可以思考的点有：k的选择[4-2](## 4-2. \k的选择) / knn算法的距离选择[4-1](##4-1.相似度评判的方式) / 是否对投票加权 / 数据如果自带顺序，需要shuffle一下。

<img src="http://myimg.icyhh.xyz/ml/03.png?imageView2/format/webp" >

## 2-3. 测试模型
​        验证模型的预测结果，这里可以思考的点：有评判指标的选择[Lecture02](https://mywiki.icyhh.xyz/2020/03/02/Lecture0x02-measurement/) ，如何正确的验证模型的效果。
<img src="http://myimg.icyhh.xyz/ml/04.png?imageView2/format/webp" >


# 3. 手写knn
## 3-1. 核心原理
​        一言以蔽之预测/分类思路是：找到已知数据集里与自己最相似（e.g. 欧氏距离）的k个样本，借鉴他们的结果（e.g. 回归模型则取平均/加权取平均，分类模型则投票/加权投票）。
<img src="http://myimg.icyhh.xyz/ml/05.png?imageView2/format/webp" >

## 3-2. 效果对比
​       首先展示盲选k=5时，在全数据集（训练样本数：51924，测试样本数：5770）中，sklearn封装好的knn表现：

<img src="http://myimg.icyhh.xyz/ml/06-1.png?imageView2/format/webp" >

​       然后缩小一下数据的范围，取1%的样本数据集（训练样本数：577，测试样本数：519）中，对比一下调包与自己手动实现的效果：

<img src="http://myimg.icyhh.xyz/ml/06-2.png?imageView2/format/webp" >

<img src="http://myimg.icyhh.xyz/ml/07.png?imageView2/format/webp" >

​       自己手写的knn比sklearn封装的相比，暴力遍历的方式运算耗时过分长[ 时间复杂度O(n) ]，所以不得不放弃在全数据集中做实验——计算最近邻居可以采用KDTree、BallTree等算法，有效加速；在1%的样本数据集中明显过拟合了，$R^2$ 的值在训练集上接近1，在测试集上却为0。


# 4. 更进一步
## 4-1.相似度评判的方式



| 名称  |  中文  |  定义/用法  |  参考公式  |  适用场景  |
|  ----  |  ----  |  ----  |  ----  |  ----  |
| Euclidean metric | 欧几里得度量 / 欧氏距离 | m维空间中两个点之间的真实距离 | $$ \sqrt{(x_i-x_j)^2+(y_i-y_j)^2} $$ | 适用于空间问题 |
| Manhattan Distance | 曼哈顿距离 / 出租车几何 | 两个点的绝对轴距总和 |            $$ |x_i-x_j|+|y_i-y_j| $$           | 适用于路径问题 |
| Chebyshev distance | 切比雪夫距离 | 两个点坐标数值差(绝对值)的最大值 | $$ max( |x_i-x_j|, |y_i-y_j| ) $$ | 网格中的距离计算（棋盘、仓储物流） |
| Mahalanobis Distance | 马氏距离 | 测量数据的协方差距离 | $$ \sqrt{(X_i-X_j)^TS^{-1}(X_i-X_j)} $$ | 量纲无关，可以排除变量之间的相关性的干扰 |
| Bhattacharyya Distance | 巴氏距离 | 用于测量两个离散或连续概率分布的相似性 | 离散概率分布：$$-ln(\sum_{x \in X}\sqrt{p(x)q(x）)})$$<br/>连续概率分布：$$\int{\sqrt{p(x)q(x)}dx} $$ | 常在分类中测量类之间的可分离性 |
| Hamming distance | 汉明距离 | 两个等长字符串，将其中一个变为另外一个所需要作的最小替换次数 | （e.g.字符串“1111”与“1001”之间的汉明距离为2。） | 信息编码（使编码间的最小汉明距离尽可能大，增强容错性） |
| Cosine | 夹角余弦 | 衡量两个向量方向的差异 | $$ cos\theta = \frac{x_ix_j+y_iy_j}{\sqrt{x_i^2+y_i^2}\sqrt{x_j^2+y_j^2}} $$ | 数据挖掘中衡量样本向量之间的差异 |
| Pearson Correlation Coefficient | 皮尔森相关系数 | 反映两个变量线性相关程度的统计量 | $$ \rho_{XY} = \frac{E[X-E(X)(Y-E(Y))]} {\sqrt{(D(X)}\sqrt{D(Y)}} $$<br/> 期望：$$E(X) = \frac{\sum{_{i=1}^n X_i}}{n}  $$<br/> 方差：$$D(X) = E{[X-E(X)]^2}  $$ <br/>相关距离： $$1 - \rho_{XY}$$ | 线性相关系数 |
| Jaccard similarity coefficient | 杰卡德相似系数 | 用两个集合中不同元素占所有元素的比例来衡量两个集合的区分度 | $$J(A,B)= \frac{|A\cap B|}{|A\cup B|}{}$$ | 衡量样本的相似度 |

​       此外提一下，闵可夫斯基距离（Minkowski Distance）是一组距离的定义，公式中有一个变参p：

$$ \sqrt[p]{\sum_{k=1}^n (x_{ik}-x_{jk})^p} $$

1. 当p=1时，是曼哈顿距离；
2. 当p=2时，是欧氏距离；
3. 当p→∞时，就是切比雪夫距离。



​       其他需要注意的 / 经验记录：

- 值域越大的变量常常会在距离计算中占据主导作用，因此应先对变量进行**归一化**。

- 当变量数越多，欧式距离的区分能力就越差。



## 4-2.k的选择
1. 如果k太小，分类结果易受噪声点影响，误差会增大；

2. 如果k太大，近邻中又可能包含太多的其它类别的点（对距离加权，可以降低k值设定的影响）；

3. 在实际应用中，K值一般取一个比较小的数值，例如采用**交叉验证法（cross-validation）**来选择最优的K值。

- 经验规则：k一般低于**训练样本数的平方根**。


# 附录
Ref. 0 https://scikit-learn.org/stable/modules/generated/sklearn.neighbors.KNeighborsRegressor.html
Ref. 1 木东居士 / [机器学习的敲门砖：初探kNN算法](https://mp.weixin.qq.com/s/VslgD9CHyu8w6KQtf3WQYQ)<br/>
Ref. 2 数月亮 / [机器学习-KNN算法](https://www.cnblogs.com/gemine/p/11130032.html)<br/>
Ref. 3 JokerChange /  [KNN浅析（距离、k值等）](https://blog.csdn.net/m0_37075274/article/details/902736920)<br/>
Ref. 4 [常见距离公式](https://www.zybuluo.com/spiritnotes/note/288072)<br/>
Ref. 5 lijfrank /  [几种常见的距离计算公式](https://blog.csdn.net/Frank_LJiang/article/details/102646649)<br/>