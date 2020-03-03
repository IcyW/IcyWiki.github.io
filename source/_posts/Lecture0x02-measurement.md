---
title: Lecture0x02_measurement
toc: true
date: 2020-03-02 23:52:03
tags: [Machine Learning]
categories: Machine Learning
---

# 1. 回归模型中的指标选择

## 评判指标一览
- 在预测任务中，给定样例集$D = {(x_1, y_1), (x_2, y_2),...,(x_n,y_n)}$，其中$y_i$是样本$x_i$的真实的观测值，$n$表示样本数量，$f(x_i)$代表对样本$x_i$的预测值。

| 名称                                       | 中文               | 特点                                                         | 公式                                                         | 优劣                                                         |
| ------------------------------------------ | ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Mean Absolute Error（MAE）                 | 平均绝对误差       | $$[0, +\infty)$$误差越小越好。也即L1损失。                   | $$MAE =\frac{1}{n}\sum_{i=0}^{n}{|y_i - f(x_i)|}$$           | 容易受到极端值影响                                           |
| Mean Absolute Pencentage Error (MAPE）     | 平均绝对百分比误差 | $$[0, +\infty)$$误差越小越好。加权版的MAE                    | $$MAPE = \frac{1}{n}\sum_{i=0}^{n}|\frac{y_i - f(x_i)}{y_i}|$$ | 不易受个别离群点影响，鲁棒性更强；$y_i$有0则不能用。         |
| Mean Squared Error/Deviation（MSE/D）      | 均方误差           | $$[0, +\infty)$$误差越小越好。也即L2损失。                   | $$MSE = \frac{1}{n}\sum_{i=0}^{n}(f(x_i)-y_i)^2$$            | 更易受到极端值影响                                           |
| Root Mean Square Error/Deviation（RMSE/D） | 均方根误差         | $$[0, +\infty)$$误差越小越好。MSE开根号                      | $$RMSE = \sqrt{\frac{1}{n}\sum_{i=0}^{n}(f(x_i)-y_i)^2}$$    | 容易受到极端值影响                                           |
| Normalized Root Mean Square Error（NRMSE） | 归一化的均方根误差 | $$[0, 1]$$误差越小越好。归一化的RMSE                         | $$NRMSE = \frac{RMSE}{y_{max}-y_{min}}$$                     | 容易受到极端值影响                                           |
| Coefficient of determination （R squared） | 决定系数           | $$[0, 1]$$反映因变量的全部变异能通过回归关系被自变量解释的比例，越大越好。 | $$R^2=1-1 - \frac{\sum\limits_i(y_i - f(x_i))^2 }{\sum\limits_i(y_i - \bar{y})^2}$$ | 考虑了预测值与真值之间的差异&问题本身真值之间的差异；归一化的度量标准 |

## 关于$R^2$的深入了解
### $R^2$的公式推导
- 搞清楚R2_score计算之前，我们还需要了解几个统计学概念。若用$y_i$表示真实的观测值，用$\bar{y}$表示真实观测值的平均值，用$\hat{y_i}$表示预测值,则：

**1、回归平方和：SSR**

$$SSR = \sum_{i=1}^{n}(\hat{y_i} - \bar{y})^2$$

即估计值与平均值的误差，反映**自变量与因变量之间的相关程度**的偏差平方和。

**2、残差平方和：SSE**

$$SSE = \sum_{i=1}^{n}(y_i-\hat{y_i} )^2$$

即估计值与真实值的误差，反映**模型拟合程度**。

**3、总离差平方和：SST**
$$SST =SSR + SSE= \sum_{i=1}^{n}(y_i - \bar{y})^2$$

即**平均值与真实值的误差**，反映与数学期望的偏离程度。

$$R^2=1-\frac{SSE}{SST} = 1 - \frac{\sum\limits_i(y_i - \hat{y_i})^2 / n}{\sum\limits_i(y_i - \bar{y})^2 / n} = 1 - \frac{MSE}{Var}$$
分子就变成了常用的评价指标均方误差MSE，分母就变成了方差。

### $R^2$的理解
- 对于$R^2$可以通俗地理解为**使用均值作为误差基准，看预测误差是否大于或者小于均值基准误差。**

  >  R2_score = 1，样本中预测值和真实值完全相等，没有任何误差，表示回归分析中自变量对因变量的解释越好。
  >  R2_score = 0。此时分子等于分母，样本的每项预测值都等于均值。
  >  R2_score不是R的平方，也可能为负数(分子>分母)，模型等于盲猜，还不如直接计算目标变量的平均值。

### $R^2$的运用：
  1、$R^2$ 一般用在线性模型中（非线性模型也可以用）
  2、$R^2$不能完全反映模型预测能力的高低，假如某个实际观测的<u>自变量取值范围很窄</u>，此时所建模型的$R^2$ 会较大，但这并不代表模型在外推应用时的效果肯定会很好。
  3、数据集的样本越大，$R^2$越大，因此，不同数据集的模型结果比较会有一定的误差，此时可以使用Adjusted R-Square (校正决定系数），能对添加的非显著变量给出惩罚:

  $R^2_{\text{Adj}}=1-(1-R^2)\frac{n-p-1}{n-1}$ ，其中n是样本的个数，p是变量的个数

## 其他注意事项

- 在选用评价指标时，需要考虑：
  1. 真实观测值数据中是否有0 ，如果有0值就不能用MPE、MAPE之类的指标；
  2. 数据的分布如何 ，如果是**长尾分布可以选择带对数变换的指标**，中位数指标比平均数指标更好；
  3. 是否存在极端值 ，诸如MAE、MSE、RMSE之类容易受到极端值影响的指标就不要选用；
  5. 对异常值而言，中位数比均值更加鲁棒，MAE比MSE更不易受到离群值影响；但MAE存在一个严重问题，更新梯度始终相同，这不利于模型的学习。
  5. 得到的指标是否依赖于量纲 (即绝对度量，而不是相对度量)，如果指标依赖量纲那么不同模型之间可能因为量纲不同而无法比较；（需要归一化）

#2. 分类模型中的指标选择

## 混淆矩阵（confusion_matrix）

|              | 预测值_反例0         | 预测值_正例1        |
| :----------- | :------------------- | :------------------ |
| 真实值_反例0 | True Negative（TN）  | FP                  |
| 真实值_正例1 | False Negative（FN） | True Positive（TP） |

## 指标

###  accuracy、precision、recall、F1

| 名称      | 中文          | 公式                                                         | 定义/用法                              | 特点                                                         |
| --------- | ------------- | ------------------------------------------------------------ | -------------------------------------- | ------------------------------------------------------------ |
| accuracy  | 准确度        | $$Accuracy = \frac{TP+TN}{TN+FP+FN+TP}$$                     | 所有样本中预测对的比率                 | 对于原本就有偏（正反例不平衡）的样本集，准确度无法说明问题。 |
| precision | 查准率        | $$Precision = \frac{TP}{TP+FP}$$                             | 所有预测为正例的样本中真的为正例的比率 | 倾向于只挑选最有把握的样例时，查准率较高。                   |
| recall    | 查全率/召回率 | $$ Recall = \frac{TP}{TP+FN}$$                               | 所有真实正例中被预测对的比率           | 倾向于预测为1时，查全率较高。                                |
| F1-score  | /             | $$\frac{1}{F1}=\frac {1} {Precision}+\frac{1}{Recall}$$<br/>$$F1=\frac {2PR} {P+R}$$ | 基于precision与recall的调和平均        | Precision与Recall相互矛盾；而F1越高，模型越稳健。            |

### P-R曲线

- 在P-R曲线中，Recall为横坐标，Precision为纵坐标。在P-R曲线中，曲线越凸向右上角越好。

> 配图 is on the way

### ROC曲线

- ROC曲线称为受试者工作特征曲线 （Receiver Operating Characteristic Curve），又称为感受性曲线（Sensitivity Curve），此名称的由来有二战相关的典故；而AUC（Area Under Curve）是ROC曲线下的面积。在ROC曲线中曲线越凸向左上角越好。

> 配图 is on the way



# 附录
Ref. 1 木东居士/[机器学习的敲门砖：kNN算法（中）](https://mp.weixin.qq.com/s/vvCM0vWH5kmRfrRWxqXT8Q)
Ref. 2 饼干 / [评价分类结果（上）](https://mp.weixin.qq.com/s/Fi13jaEkM5EGjmS7Mm_Bjw)
Ref. 3 饼干 / [模型之母：线性回归的评价指标](https://mp.weixin.qq.com/s/BEmMdQd2y1hMu9wT8QYCPg)
Ref. 4 云社区 / [回归模型评估指标（机器学习基础）](https://cloud.tencent.com/developer/article/1462976)
Ref. 5 [深度研究：回归模型评价指标R2_score](https://www.cnblogs.com/jpld/p/12022123.html)
Ref. 6 kuaizi_sophia / [分类和回归模型常用的性能评价指标](https://blog.csdn.net/kuaizi_sophia/article/details/84942307)
Ref. 7 大数据文摘 / [机器学习大牛最常用的5个回归损失函数，你知道几个？](https://baijiahao.baidu.com/s?id=1603857666277651546&wfr=spider&for=pc)
Ref8 [全面了解ROC曲线](https://www.plob.org/article/12476.html)