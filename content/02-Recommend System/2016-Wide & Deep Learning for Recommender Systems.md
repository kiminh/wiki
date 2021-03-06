---
title: "2016-Wide & Deep Learning for Recommender Systems"
layout: page
date: 2020-06-17
---

## 总结

- 针对大规模稀疏特征场景，本文提出了Deep和wide结合的模型结构。
- 本质上就是想同时利用先验信息和潜在信息。

## 主要内容

- 提出了Deep&Wide的模型结构，结合deep部分的泛化能力（Generalization）和wide部分的记忆能力（Memorization）。记忆能力原文中解释为"学习特征/item的共现信息，并利用历史数据中可用的相关性"，本质上是直接利用了特征工程的先验知识。泛化能力原文中解释为"基于相关性的可传递性，学习潜在的共现信息"，本质上是通过dnn来挖掘特征中的相关性从而组合成对结果更有意义的信息。
<div style="text-align: center"><img src="/wiki/attach/images/deepWide-01.png" style="max-width:800px"></div>
- 我理解，deep结构其实可以看作许多wide结构（lr）的组合，所以本质上wide就是对数据的直接加工，deep就只是把上述加工的方法进行重复堆叠而已；wide部分的意义在于特征中体现的先验知识，deep部分的意义则是基于先验知识进行探索。只用先验知识往往是不够的，因为肯定有一些潜在的信息；如果只进行探索，虽然能获取到潜在的信息，但往往会损失掉一部分先验知识，所以两者结合是最好的。
- 模型训练时，deep部分用AdaGrad，wide部分用FTRL+L1（详见另一篇笔记），原因是为了让wide部分变得更稀疏，稀疏的目的是为了serving的性能。
- 本文介绍的实验场景是app推荐，wide部分用了用户下载app和被推荐的app的特征cross product，由于这个结果明显是十分稀疏的，导致deep不容易学习，所以放在了wide部分。
<div style="text-align: center"><img src="/wiki/attach/images/deepWide-02.png" style="max-width:500px"></div>
- 实验结果如下，不再赘述。
<div style="text-align: center"><img src="/wiki/attach/images/deepWide-03.png" style="max-width:500px"></div>