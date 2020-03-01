---
title: Lecture0x01_knn
toc: true
date: 2020-02-24 12:10:42
tags: [Machine Learning, Algorithm]
categories: Machine Learning
---

> ​        前言：回想起来，距离第一次接触Machine Learning、做project、在组会上做分享，已经过了整整三年，前沿技术一直在发展，甚至YOLO之父 Joseph Redmon都宣布要退出CV界，而我的知识储备却并未增加。为了避免关键时刻生疏，今年初跟随饼干的学习小组，重温机器学习的轶事。


# KNN若只如初见

​        鉴于已经不是第一次与knn约会了，不多寒暄直入主题，给它发一个身份卡（持续更新）。



| 名称  |  中文  |  分类模型  |  回归模型  |  关注参数  |  适用场景|必备技巧 |
|  ----  |  ----  |  ----  |  ----  |  ----  |  ----  |  ----  |
| K-Nearest Neighbor  | k最近邻算法 | Yes | Yes | k, 相似度距离算法 |特征维数低，特征以连续值为主，样本量小 |归一化|



# 建模过程


## 数据集
​        恰逢疫情时段，我便拿一份公开的疫情人数[数据]( https://lab.isaaclin.cn/nCoV/)来做一个简单的、可能并不太合适的建模。

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

## 训练模型
​        将数据集切分为训练集测试集，以便验证是否有**过拟合**的问题。这里可以思考的点有：k的选择（tips: 交叉验证）/ knn算法的距离相似度选择。
<img src="http://myimg.icyhh.xyz/ml/03.png?imageView2/format/webp" >

## 测试模型
​        验证模型的预测结果，这里可以思考的点：有评判指标的选择，如何正确的验证模型的效果。
<img src="http://myimg.icyhh.xyz/ml/04.png?imageView2/format/webp" >


# 手写knn
​        todo


# 附录

Ref. 1 木东居士/[机器学习的敲门砖：初探kNN算法](https://mp.weixin.qq.com/s/VslgD9CHyu8w6KQtf3WQYQ)
Ref. 2 数月亮/[机器学习-KNN算法](https://www.cnblogs.com/gemine/p/11130032.html)


