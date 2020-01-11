# 机器学习A-Z～简单线性回归

所谓简单线性回归，其实就是自变量只有一个条件情况下的线性回归问题，是线性回归问题中最简单的一种了，这种问题在生活中也经常能简单，本文就用一个简单的例子来讲解简单线性回归。

以下有一组数据集，关于工作年限和薪水之间的联系（篇幅问题只给一部分）：

```python
YearsExperience,Salary
1.1,39343.00
1.3,46205.00
1.5,37731.00
2.0,43525.00
2.2,39891.00
2.9,56642.00
3.0,60150.00
3.2,54445.00
3.2,64445.00
3.7,57189.00
3.9,63218.00
```

正常情况下薪水都是会随着工作年限的增长而增长，因此这两者之间是有着很明显的线性关系的，且这里只有工作年限一个自变量，因此是个简单线性回归问题。我们现在要做的，就是用数学公式来表示这两者之间的关系，并能预测不同工作年限下可能的薪水值。

首先要对数据进行预处理，这里只需要导入数据集，切分成训练集和测试集两步即可。对于特征缩放这个点特别提一下，由于很多数据科学的包中其实已经包含了特征缩放的相关功能，因此某些情况下是不需要进行特征缩放的，但对于没有特征缩放的要注意下自己手动进行特征缩放，本文是不需要的。因此代码可以直接使用之前使用过的代码，如下所示（后面需要的包这里先导入了）：

```python

import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
import matplotlib.pyplot as plt

data_path = '../simple_regression/data/Salary_Data.csv'

dataset = pd.read_csv(data_path)
X = dataset.iloc[:, :-1].values
y = dataset.iloc[:, 1].values

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=1/3, random_state=0)
```

拆分完测试集和训练集后就要进行线性回归处理了，这里同样适用的是sklearn包，首先构造回归器：

```python
# Fitting Simple Linear Regression to the Training Set
regressor = LinearRegression()
regressor.fit(X_train, y_train)
```

这里使用的是LinearRegression这个类，然后传入训练集让其训练的出一个线性回归器。然后再用得出的回归器来预测测试集的结果：

```python
# Predicting the Test set results
y_pred = regressor.predict(X_test)
```

得出预测结果后我们来看看这次训练得到的结果是否足够准确，这里通过matplotlib包来将测试集和训练集的点以及得出的线性回归方程画到图像上来观察：

```python
#Visualising the Training set results
plt.scatter(X_train, y_train, c='red')
plt.plot(X_train, regressor.predict(X_train), c='blue')
plt.title('Salary VS Experience (training set)')
plt.xlabel('Years of Experience')
plt.ylabel('Salary')
plt.show()

#Visualising the test set results
plt.scatter(X_test, y_test, c='red')
plt.plot(X_train, regressor.predict(X_train), c='blue')
plt.title('Salary VS Experience (test set)')
plt.xlabel('Years of Experience')
plt.ylabel('Salary')
plt.show()
```

这里解释一下其中的一行代码：

`plt.plot(X_train, regressor.predict(X_train), c='blue')`

虽然一个是画训练集一个是画测试集，但由于我们实际上用的是通过训练集训练出的线性回归器，所以这里画出线性回归线的时候实际上不需要再将其中的参数换成X_test。然后此时得到的图像如下：

![训练集](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/xxhg1.png)

![测试集](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/xxhg2.png)

很明显能看出此次的训练结果在测试集上的表现还是比较令人满意的，基本和数据吻合，这样就得出了一个简单的线性回归的模型。后面的文章会继续讲解多变量的线性回归，此文只是最简单的回归，作为基础入门。