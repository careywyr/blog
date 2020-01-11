# 机器学习A-Z～多项式回归

之前的文章中已经学习过多元线性回归，现在来讲讲多项式回归。首先说说多项式线性回归，表达式可以表示为：
$$
y = b_0 + b_1x_1 + b_2x_1^2 + ... + b_nx_1^n
$$
这个表达式和多元线性回归非常像，唯一的区别就是多项式线性回归中存在很多次方项，而多元线性回归中是多个变量。实际上这里可以把多元线性回归中的多个变量理解成多项式中的$x_n^2$。所谓线性看的是$b_0$一直到$b_n$这些参数的一个线性组合，跟自变量是否线性其实没什么关系，因此这种情况依然是线性的。当然也存在非线性的多项式回归，比如这里假设公式是$y = b_0/b_2 + b_3x_3$，这个时候就不是关于$b_0$一直到$b_n$这些参数的线性表达式了。那么什么时候使用多项式回归呢，下面有一个例子：

![多项式回归](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-01-11%20%E4%B8%8B%E5%8D%884.25.05.png)

图像中的点表示数据集，如果用简单或者多元线性回归显然拟合效果不会很好，这个时候就可以使用多项式回归来拟合，这里拟合的图像会变成一条曲线，更加符合当前数据集的实际情况。

多项式回归的代码实现和线性回归很接近，前几个步骤还是一样的，这里首先给出数据集：

| Position          | Level | Salary  |
| ----------------- | ----- | ------- |
| Business Analyst  | 1     | 45000   |
| Junior Consultant | 2     | 50000   |
| Senior Consultant | 3     | 60000   |
| Manager           | 4     | 80000   |
| Country Manager   | 5     | 110000  |
| Region Manager    | 6     | 150000  |
| Partner           | 7     | 200000  |
| Senior Partner    | 8     | 300000  |
| C-level           | 9     | 500000  |
| CEO               | 10    | 1000000 |

这组数据集是某公司各个级别的职位对应的薪资，假设这个时候来了一个新人，我们需要根据这份薪资表来决定要给他多少的薪水。第一步依然是导入数据集，由于这里没有分类的变量，所以不需要进行虚拟编码。再者由于我们需要根据这份数据来决定新人的薪资，因此这份数据其实就是训练集，而新人的资料和薪资才是测试集，这里就不需要再将当前数据集拆分成训练集和测试集。

```python
import pandas as pd
from sklearn.linear_model import LinearRegression
from sklearn.preprocessing import PolynomialFeatures


data_path = '../data/Position_Salaries.csv'

dataset = pd.read_csv(data_path)
X = dataset.iloc[:, 1:2].values
y = dataset.iloc[:, 2].values
```

导入数据后我们先进行线性回归的拟合，之后我们会将拟合出的模型和多项式回归的模型进行比对。

```python
# Fitting Linear Regression to the Training set
lin_reg = LinearRegression()
lin_reg.fit(X, y)
```

接下来再进行多项式回归的拟合，这里用的是sklearn.preprocessing中的PolynomialFeatures，这里我们根据数据假设最高此项为2。

```python
# Fitting Ploynomial Regression to the Training set
poly_reg = PolynomialFeatures(degree=2)
X_poly = poly_reg.fit_transform(X)
lin_reg2 = LinearRegression()
lin_reg2.fit(X_poly, y)
```

然后我们看看两种方式拟合的模型的效果：

```python
# Visualising the Linear Regression results
plt.scatter(X, y, c='r')
plt.plot(X, lin_reg.predict(X), c='b')
plt.title('Truth or Bluff (Linear Regression)')
plt.xlabel('Position Level')
plt.ylabel('Salary')
plt.show()
# Visualising the Polynomial Regression results
plt.scatter(X, y, c='r')
plt.plot(X, lin_reg.predict(X), c='b')
plt.plot(X, lin_reg2.predict(poly_reg.fit_transform(X)), c='b')
plt.title('Truth or Bluff (Polynomial Regression)')
plt.xlabel('Position Level')
plt.ylabel('Salary')
plt.show()
```

下图分别是线性回归模型和多项式回归模型结果：

![线性回归](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-01-14%20%E4%B8%8B%E5%8D%888.25.08.png)



![多项式回归](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-01-14%20%E4%B8%8B%E5%8D%888.25.19.png)

我们发现多项式回归的确比线性回归更合适一点，但这里的模型和实际数据依然有着不小的差距，那么这里就需要对拟合的方式进行一些调整，把最高次数调整成4试试看，得到结果后发现这次得到的结果更加贴近与实际数据。由于图像中的线看起来是一段段的折线，而不是一个平滑的曲线，这里我们可以对其进行优化一下从而得到一条平滑的曲线。实际上就是让其又更多的间距小的点来画这个曲线，图像和代码如下：

![最高次数为4的多项式回归](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/dxshg.png)



```python
X_grid = np.arange(min(X), max(X), 0.1)
X_grid = X_grid.reshape(len(X_grid), 1)
plt.scatter(X, y, c='r')
plt.plot(X_grid, lin_reg2.predict(poly_reg.fit_transform(X_grid)), c='b')
plt.title('Truth or Bluff (Polynomial Regression)')
plt.xlabel('Position Level')
plt.ylabel('Salary')
plt.show()
```

接下来，我们要利用拟合好的模型来预测新人应当匹配的薪水，这里也分别用线性回归和多项式回归模型来预测。假设新人在6-7等级之间，设置其等级为6.5.

```python

# Predicting a new result with Linear Regression
lin_reg.predict(6.5)

# Predicting a new result with Polynomial Regression
lin_reg2.predict(poly_reg.fit_transform(6.5))
```

得到的结果分别为：330378.78787879和158862.45265153,很明显后者更符合实际情况，因此这里多项式回归模型时更好的选择。

