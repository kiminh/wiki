---
title: "2017-LightGBM: A Highly Efficient Gradient Boosting Decision Tree"
layout: page
date: 2020-06-21
---


## 总结

- GBDT能够在许多任务中取得较好的效果，但他在大数据场景面临着准确率和效率的权衡，主要的计算来自于建树时分裂节点的搜索。
- 本文通过采样梯度较大的样本（GOSS）和捆绑互斥特征来减少特征数量（EFB）两种方法来加速计算。

## 主要内容

- Gradient-based One-Side Sampling，目的是为了采样样本加速运算。具体做法是保留梯度为Top a的样本，在剩余样本中再采样b个（计算信息增益时乘以权值（1-a)/b以作修正）。
- Exclusive Feature Bundling，目的是为了减少特征数量。在许多场景下会存在大量的稀疏特征（少量的非零值），这些稀疏特征之间是互斥的（几乎不会同时为非零值）。利用这种特性对这些特征重新编码，捆绑为新的特征。
    - 特征捆绑的选择：将特征按照非零值个数排序，遍历每个特征，将它分配给现有特征包，或者新建一个特征包，使得总体冲突最小。
    - 特征捆绑的方法：例如有2个数据为{data1[f1=0,f2=1],data2[f1=1,f2=0]}，捆绑后为{data1[f3=0],data2[f3=1]}。
    - 其本质就是将稀疏特征进行组合得到新的特征而不损失信息，当然100个数据中可能出现f1,f2同时为0或者1的，但是可以忽略，某种程度上还引入了正则。

## LightGBM相比于XgBoost的其他优化

- 直方图做差加速。为了加速分裂点搜索，将每个特征都离散化到直方图中，而在计算叶子的直方图中可以先计算其中一个，另一个直接利用直方图做差得到。上述操作存在一个问题是叶子和父节点的直方图是一样的，这就和XgBoost中的全局直方图类似，效果不如局部的直方图近似，但分裂完成之后，应该可以再对直方图做调整。
- 子树采用Leaf-wise的生长策略，对选取叶子结点中分裂增益最大的，减少了Level-wise中一些不必要的计算，同时加入了最大深度的限制防止过拟合。
- 对于类别特征，转换成onehot编码之后的也是互斥的（肯定只有1个非零值），所以本文直接将一个类别特征与一个bin关联，无需预先进行onehot编码。在建树时，将直方图按对应类别label均值进行排序，然后依次搜索，为防止过拟合还加入了一些约束和正则化。
- 对并行计算进行了一系列优化。