# 机器学习A-Z～先验算法Apriori

本文将会讲述关联规则学习中的一个基本算法，叫做先验算法。所谓先验算法，就是找出不同事件之间的联系。比如一个人在超市买了产品A，他可能会买货物B。这里我们看一个例子。

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-04-09%20%E4%B8%8B%E5%8D%882.51.18.png)

这里有七笔交易，那么根据这些数据我们可以得出一些猜测，当有货物A时可能有货物B。比如有汉堡的时候可能有薯条，如果有蔬菜可能就有水果等等。当我们的商店越来越大，交易记录越来越多，那么通过人为观察来看出这些商品之间的联系就很难了，这时就需要用到先验算法。

## 概念

先验算法当中有三个核心概念，support(支持度), confidence(信心水准), lift(提升度)。

先来看看支持度，比如交易的例子，对于一个商品I来说，那么就是所有包含商品I的交易数目除以总的交易数目。
$$
support(I) = \frac{transactions\quad containing\quad I}{transactions}
$$
第二个概念，信心水准，这里I1表示商品1，I2表示商品2，那么信心水准就是同时包含商品1和2的交易除以包含商品1的交易记录个数。
$$
confidence(I_1 -> I_2) = \frac{transactions\quad containing\quad I_1\quad and\quad I_2}{transactions \quad containing \quad I_1}
$$
第三个概念，提升度，这个和支持度和信心水准有关,就是configdence/support。当这个提升度大于1时，我们可以认为商品$I_1$对$I_2$是有提升的。
$$
lift(I_1 -> I_2) = \frac{confidence(I_1->I_2)}{support(I_2)}
$$
那么现在做个总结，这个先验算法主要可以分为四步：

1. 设置一个最低的support和confidence
2. 选择所有support比刚刚设置的要大的商品
3. 根据刚刚已经选择的商品，选择所有比刚刚定义的最小confidence要高的所有规则的集合
4. 把刚刚的规则从大到小排序，选出提升度最高的几个。

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-04-10%20%E4%B8%8A%E5%8D%8810.59.09.png)

## 代码实现

这次代码实现我们使用一家商店如何使用先验算法来提高销量的例子。这里有这家商店最近的所有交易，每个交易中分别卖出了不同种类的商品。

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-04-10%20%E4%B8%8A%E5%8D%8811.28.52.png)

这次的代码使用了一个额外的包，因此需要大家自己去看这个包里的代码，这里只贴出如何使用这个包进行先验算法的使用。

```python
from apyori import apriori
import pandas as pd

dataset = pd.read_csv('Market_Basket_Optimisation.csv', header=None)
transactions = []
for i in range(0, 7501):
    transactions.append([str(dataset.values[i, j]) for j in range(0, 20)])

# Training Apriori on the dataset

rules = apriori(transactions, min_support=0.003, min_confidence=0.2, min_lift=3, min_length=2)

# Visualising the results
results = list(rules)
myResults = [list(x) for x in results]
```

这里的apyori包可以去我的[github](https://github.com/careywyr/udemy_ml_a2z)查看这部分代码。以上，就是先验算法的相关基础知识。

