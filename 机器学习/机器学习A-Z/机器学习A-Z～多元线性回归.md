# 机器学习A-Z～多元线性回归

之前的文章已经讲述了简单线性回归的概念和代码实现，现在来继续看看多元线性回归。所谓多元线性回归其实就是自变量的个数变多了，之前的简单线性回归方程可以表示为：$y=b_0 +bx$,那么在多元中则是$y=b_0+b_1x_1+b_2x_2+...+b_nx_n$。

## 线性回归的几个前置条件

在正式使用多元线性回归之前，我们先谈谈关于线性回归的几个前置条件，首先，在线性回归中有几个重要的假设如下所示：

1. Linearity 线性 （数据呈线性关系）
2. Homoscedasticity 同方差性（数据要有相同的方差）
3. Multivariate normality 多元正态分布 （数据要呈现多元正态分布）
4. Independence of errors 误差独立 （各个维度上的误差相互独立）
5. Lack of multicollinearity 无多重共线性 （没有一个自变量和另外的自变量存在线性关系）

也就是说我们在使用线性回归时，要考虑到实际情况是否吻合上述的几种假设，否则可能会导致拟合的模型不准确。

除此之外，之前数据预处理的时候也提到过一个叫做**虚拟变量**的概念，现在对其进行详细的讲解。来看下面的数据集（同样由于篇幅问题只提供部分）：

| R&D Spend | Administration | Marketing Spend | State      | Profit    |
| --------- | -------------- | --------------- | ---------- | --------- |
| 165349.2  | 136897.8       | 471784.1        | New York   | 192261.83 |
| 162597.7  | 151377.59      | 443898.53       | California | 191792.06 |
| 153441.51 | 101145.55      | 407934.54       | Florida    | 191050.39 |
| 144372.41 | 118671.85      | 383199.62       | New York   | 182901.99 |

这组数据集反映的是公司的研发投入，行政支出，市场支出以及所在地点对于公司的收益的影响。其中所在地点这个变量是个分类变量，没有数值的概念，因此无法对其进行排序或者带入方程。数据预处理中，我们使用了虚拟变量对其进行重新编码，那么对于这组数据我们也可以做同样的操作，也就是说State这列数据可以拆分成三列：

| New York | California | Florida |
| -------- | ---------- | ------- |
| 1        | 0          | 0       |
| 0        | 1          | 0       |
| 0        | 0          | 1       |
| 1        | 0          | 0       |

假设R&D Spend对应的自变量为$x_1$，Administration对应的是$x_2$，Marketing Spend对应的是$x_3$，虚拟编码对应的是$D_1$,$D_2$,$D_3$。但是对于虚拟编码，任意一个编码都能用另外两个来表示，比如是否是Florida，那么如果New York和California如果存在1，则肯定不是Florida，否则就是Florida。那么可以定义其多元线性回归方程为：
$$
y = b_0 + b_1*x_1 + b_2*x_2 + b_3*x_3 + b_4*D_1 + b_5*D_2
$$
这里顺便一提，我们这里给予虚拟变量的值是0和1，那么如果是（100，0）或者（-1，1）可以吗？答案是肯定的，比如说（100，0），那我们在方程中将前面的参数D缩小一百倍，达到的效果其实是一样的。

接下来想想，如果把我们删掉的$b_6*D_3$加上那么这个方程对不对呢？其实这里就是虚拟变量的一个陷阱。回过头看看上面所说的关于回归问题的几个假设，第五条，无多重共线性这个性质我们是否满足了？任意一个变量实际上都可以用其他的变量来表示，比如$D_3=1-D_1-D_2$，那么就不符合无多重共线性这个要求了。因此，在使用虚拟变量的时候一定要注意实际上使用的虚拟变量的数量应该是始终都要忽略掉一个的。

## 如何构建一个多元线性回归模型

接下来我们再谈谈如何一步一步地去构建一个多元线性回归模型。在实际应用中，往往会遇到对于一个因变量y，有很多的自变量x1，x2等等，但这些自变量不是所有的都是对这个y的预测很有帮助因素，我们需要从其中剔除掉无用的元素来得到最合适的模型，那么此时就有一个问题，如何来选择这些自变量呢？这里有五种方法来建立模型：

1. All-in
2. Backward Elimination 反向淘汰
3. Forward Selection 顺向选择
4. Bidirectional Elimination 双向淘汰
5. Score Comparison 信息量比较

其中的2，3，4三种方式，也是我们最常用的，叫做Step Regression也就是逐步回归，它们的算法是类似的，只是应用的顺序有点不同。接下来一种种的介绍这几种方法。

### All-in

所谓All-in，就是把所有我们认为的自变量全扔进去，一般有这几种情况使用：

- Prior Knowledge 我们已经提前知道了所有的信息，知道这些所有自变量都会对模型结果有影响
- You have to 上级需要你必须使用这些自变量
- Preparing for Backward Elimination 为反向淘汰做准备

All-in这个方法很简单，但一般都是一些特殊情况下或者一些外力干涉时才会使用，这种方法不作推荐。

### Backward Elimination

反向淘汰这个算法的精髓在于对于每一个这个模型的自变量来说，他对我们的模型的预测结果其实是有影响力的，用统计学上的一个概念叫做P-value来形容它的影响力。在这里我们先定义它的这个影响力是否显著的一个门槛也就是一个significance level（SL），先定义为0.05。接着第二步使用所有的自变量来拟合出一个模型。第三步，对于这个模型当中的每一个自变量都来计算它的P值（P-value）,来显示它对我们模型有多大的影响力，然后我们取这个最高的P值，假设这个P>SL,就继续往第四步，否则就算法结束。那么第四步，最高的P值对应的那个自变量我们就要将它从我们的模型中去除。第五步，去除了一个自变量后，在用剩下的自变量重新对模型进行拟合，因此这里就是一个第三步到第五步的一个循环，直到所有剩下的P值都比SL要小，这样就说明模型已经拟合好了。步骤详情如下：

![Backward Elimination](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-01-09%20%E4%B8%8B%E5%8D%883.58.30.png)



### Forward Selection

顺向选择和反向淘汰的算法思想很接近，只是顺序有了个颠倒。这里第一步还是先定一个显著性的门槛SL=0.05.第二步，我们在这边对每个自变量$x_n$都进行简单线性回归拟合，分别得到它们的P值，然后取得它们中最低的。第三步对于这个最低的P值，我们的结论就是这个自变量它对我们将要拟合的模型的影响是最大的，所以说，我们会保留这个自变量。接下来第四步我们再看剩下的自变量当中加上哪一个会给我们带来最小的P值，假如说新的P值比我们之前定义的SL小，那就重新回到第三步，也就是第三步又加入了一个新的变量然后在接下来剩下的变量当中，重新找最大的P值，然后继续加到模型当中，以此类推，直到剩下来的还没有被加入模型当中的变量它们的P值全部大于SL，也就是说剩下的变量对于模型有可能产生的影响不够显著，那对这些就不予采纳，这个时候模型就拟合好了。步骤详情如下：

![Forward Selection](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-01-09%20%E4%B8%8B%E5%8D%884.38.31.png)

### Bidirectional Elimination

所谓双向淘汰，其实就是对之前的两种算法的结合。在第一步中，我们需要选择两个显著性门槛：一个旧的变量是否应该被剔除和一个新的还没有被采纳的变量是否应当进入我们的模型。在第二步中，我们要进行顺向选择，来决定是否采纳一个新的自变量。第三步要进行反向淘汰，也就是我们可能要剔除旧的变量，然后在第二第三步之间进行循环，由于已经定义了两个门槛，但出现新的出不去，旧的进不来时，就说明模型已经拟合好了。步骤详情如下：

![Bidirectional Elimination](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-01-09%20%E4%B8%8B%E5%8D%884.45.03.png)

### Score Comparison 

最后一种，信息量比较。所谓信息量，就是对于一个多元线性回归模型的一个评价方式。比如说给它进行打分。那么就有很多种打分方式。比如最常见的一种，叫做Akaike criterion.对于所有可能的模型，我们对它们进行逐一的打分，对于多元线性回归，如果有N个自变量，那么就有$2^N-1$个不同的模型，对这些模型打分后选择分数最高的模型。那这里就会有一个问题，如果N很大的时候，模型的数量就会非常庞大。所以说这个方法虽然直觉上很好理解，但自变量数量很大时就不适合使用这种方法。

![Score Comparison](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-01-09%20%E4%B8%8B%E5%8D%8810.04.45.png)

## Coding

接下来就要进行代码编写了，首先还是老样子，先进行数据预处理，将数据导入并对分类数据进行虚拟编码，然后将数据集划分成训练集和测试集。上面有提及过虚拟编码的陷阱，实际上我们使用的包中已经避开了这个陷阱，但为了强调这个问题，这里也对其进行处理一下：

```python
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import LabelEncoder, OneHotEncoder
from sklearn.linear_model import LinearRegression
import statsmodels.formula.api as sm
import numpy as np

data_path = '../data/50_Startups.csv'

dataset = pd.read_csv(data_path)
X = dataset.iloc[:, :-1].values
y = dataset.iloc[:, 4].values

labelencoder_X = LabelEncoder()
X[:, 3] = labelencoder_X.fit_transform(X[:, 3])
onehotencoder = OneHotEncoder(categorical_features=[3])
X = onehotencoder.fit_transform(X).toarray()

# Avoiding the Dummy Variable Trap
X = X[:, 1:]

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=0)
```

预处理结束后，就来进行拟合线性回归模型，这里第一步拟合线性回归模型的方式和之前的简单线性回归是一样的。

```python
# Fitting Multiple Linear Regression to the Training set
regressor = LinearRegression()
regressor.fit(X_train, y_train)

# Predicting the Test set results
y_pred = regressor.predict(X_test)
```

目前我们拟合的模型使用的方法是上述提到的All-in方法，但这个方法并不是最好的，这里我们使用Backward Elimination来对回归器进行优化，使用的工具是statsmodels.formula.api。再然后我们还要做一件事情，在多元线性回归时，很多时候模型里面都会有一个常数$b_0$，相当于$b_0*1$。但在用到的标准库的函数里面是不包含这个常数项的，因此我们需要在包含自变量的矩阵中加上一列，这一列全部都是1，$b_0$就是它的系数。代码如下：

```python
X_train = np.append(arr=np.ones((40, 1)), values=X_train, axis=1)
```

接下来要真正开始反向淘汰了，首先创建一个矩阵X_opt，包含了最佳的自变量选择，第一步实际上是所有的自变量，之后的不断循环后会渐渐淘汰其中的自变量。

```python
X_opt = X_train[:, [0, 1, 2, 3, 4, 5]]
```

这里为什么用X_train[:, [0, 1, 2, 3, 4, 5]]而不是直接X_train呢？这是因为后面会进行不断的淘汰，会不断删除其中的列，后续的代码就能看出来了。

创建完X_opt后，接下来就要用它来拟合新的回归器regressor_OLS。这里用的sm.OLS方法的参数解释一下：endog对应的是因变量，这里就是y_train，exog指的是自变量，因此这里就是X_opt。

```python
regressor_OLS = sm.OLS(endog=y_train, exog=X_opt).fit()
regressor_OLS.summary()
```

然后使用summary()方法，这个方法能给我们提供很多回归器的信息，执行后得到结果如下：

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-01-10%20%E4%B8%8B%E5%8D%883.19.07.png)

这里能看到所有自变量对用的P值，其中最大的是$x_2$，就是这个公司是否在加利福利亚洲，然后我们剔除$x_2$，再继续拟合，得到summary，根据结果不断剔除自变量直到最终的P值都小于我们定义的SL=0.05.代码如下：

```python
X_opt = X_train[:, [0, 1, 3, 4, 5]]
regressor_OLS = sm.OLS(endog=y_train, exog=X_opt).fit()
regressor_OLS.summary()

X_opt = X_train[:, [0, 3, 4, 5]]
regressor_OLS = sm.OLS(endog=y_train, exog=X_opt).fit()
regressor_OLS.summary()

X_opt = X_train[:, [0, 3, 5]]
regressor_OLS = sm.OLS(endog=y_train, exog=X_opt).fit()
regressor_OLS.summary()

X_opt = X_train[:, [0, 3]]
regressor_OLS = sm.OLS(endog=y_train, exog=X_opt).fit()
regressor_OLS.summary()
```

最终得到的结果是：

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-01-10%20%E4%B8%8B%E5%8D%883.19.52.png)

那么根据这个结果可以得到的结论就是一家公司的收益主要跟公司的研发投入有关。当然其实其中也有其他的自变量对应了很低的P值，如果自己一步步运行这段代码会发现行政的投入对应的P值也只有0.07不算很高，如何更好的判断线性回归模型的优劣后面还会有其他的方法来判断。