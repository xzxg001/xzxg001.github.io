---
title: Metrics
description: record the ML metrics
slug: Metrics
date: 2024-04-04 00:00:00+0000
image: image.png
categories:
    - machine learning
tags:
    - model evalution
    - metrics
    - machine learning
weight: 1
math: true
---

# 机器学习常用性能指标总结metrics

​	在机器学习中，[性能指标](Metrics)是衡量一个模型好坏的关键，通过衡量模型输出y_predict 和 y_true之间的某种"距离"得出的。

​	性能指标往往是我们做模型时的最终目标，如准确率，召回率，敏感度等等，但是性能指标常常因为不可微分，无法作为优化的loss函数，因此采用如cross-entropy, rmse等“距离”可微函数作为优化目标，以期待在loss函数降低的时候，能够**提高性能指标**。而最终目标的性能指标则作为模型训练过程中，作为验证集做决定(early stoping或model selection)的主要依据，与训练结束后评估本次训练出的模型好坏的重要标准。

## 性能指标的分类

​	根据问题类型的不同，机器学习中的性能指标主要分为：回归性能指标和分类性能指标。


### 回归性能指标

#### 平均绝对误差

​	平均绝对误差（Mean Absolute Error，MAE），也称为 L1 损失，是最简单的损失函数之一，也是一种易于理解的评估指标。它是通过取预测值和实际值之间的绝对差值并在整个数据集中取平均值来计算的。从数学上讲，它是绝对误差的算术平均值。MAE 仅测量误差的大小，不关心它们的方向。MAE越低，模型的准确性就越高。

优点：

- 由于采用了绝对值，因此所有误差都以相同的比例加权。
- 如果训练数据有异常值，MAE 不会惩罚由[异常值](https://so.csdn.net/so/search?q=异常值&spm=1001.2101.3001.7020)引起的高错误。
- 它提供了模型执行情况的平均度量。

缺点：

- 有时来自异常值的大错误最终被视为与低错误相同。
- 在零处不可微分。许多优化算法倾向于使用微分来找到评估指标中参数的最佳值。在 MAE 中计算梯度可能具有挑战性。

```python
def mean_absolute_error(true, pred):
    abs_error = np.abs(true - pred)
    sum_abs_error = np.sum(abs_error)
    mae_loss = sum_abs_error / true.size
    return mae_loss
```

#### 平均偏差误差

平均偏差误差(Mean Bias Error ,MBE)是测量过程高估或低估参数值的趋势。偏差只有一个方向，可以是正的，也可以是负的。正偏差意味着数据的误差被高估，负偏差意味着误差被低估。平均偏差误差 是预测值与实际值之差的平均值。该评估指标量化了总体偏差并捕获了预测中的平均偏差。**它几乎与 MAE 相似，唯一的区别是这里没有取绝对值。这个评估指标应该小心处理，因为正负误差可以相互抵消。**

优点：

- 想检查模型的方向（即是否存在正偏差或负偏差）并纠正模型偏差，MBE 是一个很好的衡量标准。

缺点：

- 就幅度而言，这不是一个好的衡量标准，因为误差往往会相互补偿。
- 它的可靠性不高，因为有时高个体错误会产生低MBE。
- 作为一种评估指标，它在一个方向上可能始终是错误的。

```python
def mean_bias_error(true, pred):
    bias_error = true - pred
    mbe_loss = np.mean(np.sum(diff) / true.size)
    return mbe_loss
```

#### 相对绝对误差

相对绝对误差（Relative Absolute Error ，RAE）是通过将总绝对误差除以平均值和实际值之间的绝对差来计算的。RAE并以比率表示。RAE的值从0到1。一个好的模型将具有接近于零的值，其中零是最佳值。

优点：

- RAE 可用于比较以不同单位测量误差的模型。
- RAE 是可靠的，因为它可以防止异常值。

```python
def relative_absolute_error(true, pred):
    true_mean = np.mean(true)
    squared_error_num = np.sum(np.abs(true - pred))
    squared_error_den = np.sum(np.abs(true - true_mean))
    rae_loss = squared_error_num / squared_error_den
    return rae_loss
```

#### 平均绝对百分比误差

平均绝对百分比误差(Mean Absolute Percentage Error ,MAPE)是通过将实际值与预测值之间的差值除以实际值来计算的。MAPE 也称为平均绝对百分比偏差，随着误差的增加而线性增加。MAPE 越小，模型性能越好。

优点：

- MAPE与变量的规模无关，因为它的误差估计是以百分比为单位的。
- 所有错误都在一个共同的尺度上标准化，很容易理解。
- MAPE避免了正值和负值相互抵消的问题。

缺点：

- 分母值为零时，面临着“除以零”的问题。
- MAPE对数值较小的误差比对数值大的误差错误的惩罚更多。
- 因为使用除法运算，所欲对于相同的误差，实际值的变化将导致损失的差异。

```python
def mean_absolute_percentage_error(true, pred):
    abs_error = (np.abs(true - pred)) / true
    sum_abs_error = np.sum(abs_error)
    mape_loss = (sum_abs_error / true.size) * 100
    return mape_loss
```

#### 均方误差

均方误差（Mean Squared Error ，MSE）也称为 L2 损失，MSE通过将预测值和实际值之间的差平方并在整个数据集中对其进行平均来计算误差。MSE 也称为二次损失，因为惩罚与误差不成正比，而是与误差的平方成正比。平方误差为异常值赋予更高的权重，从而为小误差产生平滑的梯度。

MSE 永远不会是负数，因为误差是平方的。误差值范围从零到无穷大。MSE 随着误差的增加呈指数增长。一个好的模型的 MSE 值接近于零。

优点：

- MSE会得到一个只有一个全局最小值的梯度下降。
- 对于小的误差，它可以有效地收敛到最小值。没有局部最小值。
- MSE 通过对模型进行平方来惩罚具有巨大错误的模型。

缺点：

- 对异常值的敏感性通过对它们进行平方来放大高误差。
- MSE会受到异常值的影响，会寻找在整体水平上表现足够好的模型。

```python
def mean_squared_error(true, pred):
    squared_error = np.square(true - pred) 
    sum_squared_error = np.sum(squared_error)
    mse_loss = sum_squared_error / true.size
    return mse_loss
```

#### 均方根偏差

均方根偏差（Root Mean Squared Error ，RMSE）是通过取 MSE 的平方根来计算的。它测量误差的平均幅度，并关注与实际值的偏差。RMSE 值为零表示模型具有完美拟合。RMSE 越低，模型及其预测就越好。

优点：

- 易于理解，计算方便

缺点：

- 建议去除异常值才能使其正常运行。
- 会受到数据样本大小的影响。

```python
def root_mean_squared_error(true, pred):
    squared_error = np.square(true - pred) 
    sum_squared_error = np.sum(squared_error)
    rmse_loss = np.sqrt(sum_squared_error / true.size)
    return rmse_loss
```

#### 相对平方误差

相对平方误差(Relative Squared Error ,RSE)需要使用均方误差并将其除以实际数据与数据平均值之间的差异的平方。

优点

- 对预测的平均值和规模不敏感。

```python
def relative_squared_error(true, pred):
    true_mean = np.mean(true)
    squared_error_num = np.sum(np.square(true - pred))
    squared_error_den = np.sum(np.square(true - true_mean))
    rse_loss = squared_error_num / squared_error_den
    return rse_loss
```

#### 归一化 RMSE

归一化 RMSE 通常通过除以一个标量值来计算，它可以有不同的方式。有时选择四分位数范围可能是最好的选择，因为其他方法容易出现异常值。当您想要比较不同因变量的模型或修改因变量时，NRMSE 是一个很好的度量。它克服了尺度依赖性，简化了不同尺度模型甚至数据集之间的比较。

```python
def normalized_root_mean_squared_error(true, pred):
    squared_error = np.square((true - pred))
    sum_squared_error = np.sum(squared_error)
    rmse = np.sqrt(sum_squared_error / true.size)
    nrmse_loss = rmse/np.std(pred)
    return nrmse_loss
```

### 分类性能指标

#### 混淆矩阵

​	也叫可能性矩阵或错误矩阵。混淆矩阵是可视化工具，特别用于监督学习，在无监督学习一般叫做匹配矩阵。在图像精度评价中，主要用于比较分类结果和实际测得值，可以把分类结果的精度显示在一个混淆矩阵里面。

| None     | 预测1 | 预测0 | 合计        |
| -------- | ----- | ----- | ----------- |
| 实际1(P) | TP    | FN    | TP+FN (P)   |
| 实际0(N) | FP    | TN    | FP+TN(N)    |
| 合计     | TP+FP | FN+TN | TP+FN+FP+TN |

其中**T/F表示预测是否正确**，**P/N表示预测的label**而不是实际的label

 True Positive（TP）：真正类。样本的真实类别是正类，并且模型识别的结果也是正类。

 False Negative（FN）：假负类。样本的真实类别是正类，但是模型将其识别为负类。

 False Positive（FP）：假正类。样本的真实类别是负类，但是模型将其识别为正类。

 True Negative（TN）：真负类。样本的真实类别是负类，并且模型将其识别为负类。 

#### 准确率

$$Acc=(TP+TN)/(TP+FN+FP+TN)$$

​	即预测正确的样本比例，表示了一个分类器的区分能力，注意，这里的区分能力**没有偏向于是正例还是负例**，这也是Accuracy作为性能指标最大的问题所在。

- **优点**：简单易懂。
- **缺点**：在类别不平衡的情况下，容易产生误导。例如，当正负样本比例极度不平衡时，模型即使预测所有样本为多数类，也可能得到较高的准确率。

#### 精确率/查准率

$$Pre=TP/(TP+FP)$$

​	表示在所有被预测为正例的样本中，真正是正例的比例。对于数据不平衡或是当某一方数据漏掉(通常是把这样的例子作为正例)时会产生很大的代价的时候，我们需要更有效的指标作为补充。

- **优点**：在关注预测结果为正类时的正确性（如垃圾邮件检测）时，精确率是一个重要指标。
- **缺点**：忽略了实际正类样本中有多少被正确预测。

#### 召回率/查全率/敏感度

$$Recall=TP/(TP+FN)$$

​	表示在所有实际为正例的样本中，被预测为正例的比例。在医学领域中，常把患病这样的高风险类别作为正类，**当漏掉正类的代价非常高**，像是漏诊可能导致病人的延迟治疗，召回率就很重要。

在医学中，必须极力**降低漏诊率**，而误诊(把负例判为正例)相对于漏诊的重要性就低了很多。

- **优点**：在关注所有正类样本都被正确识别时（如疾病检测），召回率是一个重要指标。
- **缺点**：忽略了预测为正类的样本中有多少是错误的。

#### 特异性

$$Specificity=TN/(FP+TN)$$

​	表示在所有实际为负的样本中，被预测为负例的比例，即有多大概率被预测出来。可以简单地将特异性理解为负例查全率。特异性在医疗中也被认为是一个重要指标，因为特异性低也就是“误诊率高”，举一个极端例子，一个分类器把所有的样本都判定成患病，此时敏感度为1，但是有特异性却很低。因此，在医学领域，特异性和敏感度是需要同时考量的。

- **优点**：关注负类性能，在**负类重要性高**的任务中（如疾病筛查中的健康人判断），特异性比准确率（Accuracy）更能反映模型对负类的识别能力。
- **缺点**：忽略正类性能，依赖明确的负类定义，无法单独作为评估标准

#### **Fβ_Score**

​	Fβ的物理意义就是将精确率和召回率的一种加权平均，在合并的过程中，召回率的权重是精确率的β倍。常用的是F1分数（F1 Score），是统计学中用来衡量二分类模型精确度的一种指标。Accuracy和F1-Score是判断分类模型总体的标准。

![F1 score计算](https://i-blog.csdnimg.cn/blog_migrate/3e4f2a0cd742ceae4650569c634cfaf2.png)



![Fβ score计算](https://i-blog.csdnimg.cn/blog_migrate/6c57b65304c7a309b71f5795d6d8ddfb.png)

- Macro-F1和Micro-F1是相对于多标签分类而言的。

- Micro-F1，计算出**所有类别总的**Precision和Recall，然后计算F1。

- Macro-F1，计算出**每一个类的**Precison和Recall后计算F1，最后将F1平均。

  ​		

- **优点**：在类别不平衡的情况下，比单独使用精确率或召回率更能全面反映模型性能。

- **缺点**：无法同时优化精确率和召回率的具体值。

#### ROC曲线和AUC

​	ROC曲线的全称叫做Receiver Operating Characteristic，常常被用来判别一个分类器的好坏程度

![ROC曲线](https://i-blog.csdnimg.cn/blog_migrate/c030c4ec2fbed37d0093ac0428d796fb.png)

x轴坐标是False positive rate，即假正率，定义为$$FPR=FP/(FP+TN)=1-Specificity$$，即误诊率，代表所有实际为负例的样本中，预测错误的比例。

y轴坐标是True positive rate，即真正率，定义为$$TPR=TP/(TP+FN)$$,即召回率/查全率/敏感度

现在可知，ROC的x轴是误诊率，y轴是漏诊率。

​	AUC，即曲线下面积（Area Under Curve），是 ROC 曲线下面积的一个数值表示。它提供了一个定量的指标，用来衡量分类模型的整体表现。AUC 值范围从 0 到 1，值越大表示模型性能越好。

<img src="https://i-blog.csdnimg.cn/blog_migrate/e4931f4fe68fc59f133ab07528b12498.png" alt="在这里插入图片描述" style="zoom: 33%;" />

下方是具体绘制的方法

画出ROC的曲线的方法不只是计算一次误诊率和漏诊率，按照以下方式进行：

1. 将分类器预测为正例的概率从小到大排序
2. 把每两个样本间的概率作为阈值，小于该阈值的分为负例，大于的分为正例
3. 分别计算TPR和FPR
4. 转2
5. 当所有阈值都被枚举完之后，获得一组(TPR, FPR)的坐标点，将它们画出来。结束

下方是具体的python代码

```python
from sklearn.metrics import roc_curve
    import matplotlib.pyplot as plt
    
    p_rate = model.get_prob(X) #计算分类器把样本分为正例的概率
    fpr, tpr, thresh = roc_curve(y_true, p_rate)
   
    plt.figure(figsize=(5, 5))
    plt.title('ROC Curve')
    plt.xlabel('False Positive Rate')
    plt.ylabel('True Positive Rate')
    plt.grid(True)
    plt.plot(fpr, tpr)
    plt.savefig('roc.png')
```

​	模型的ROC曲线**离对角线越近，模型的准确率越低**。如果模型真的很好，随着有序列表（第一张由元组标号的图）向下移动，开始就可能遇到真正例元组，图开始就陡峭地从 0 开始上升，后来遇到的真正例越来越少，假正例越来越多，曲线平缓变得更加水平。

​	为了评估模型的准确率，可以测量曲线下方的面积（AUC）。面积越接近 0.5 ，模型的准确率越低。完全正确的模型面积为 1.0

- 优点：
  1. 兼顾正例和负例的权衡。因为TPR聚焦于正例，FPR聚焦于与负例，使其成为一个比较均衡的评估方法。
  2. ROC曲线选用的两个指标，不受类别不平衡影响，都不依赖于具体的类别分布，全面反映模型在不同阈值下的性能。
- **缺点**：计算复杂度较高，解释起来可能不直观。













