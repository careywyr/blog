# 机器学习A-Z～Logistic Regression

之前的课程谈论的都是线性回归问题，现在开始看看分类问题。首先讲的是逻辑回归，英文叫做Logistic Regression。看一下下面的图像，因变量不再如同线性回归那样相对来说比较连续，这里的数据点是离散的。

比如我们现在是一家媒体公司，有一些广告投放，为了让客户购买产品。现在收集了客户的年龄信息和客户是否购买的产品，那么就得到了这些数据点。

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-02-12%20%E4%B8%8A%E5%8D%8810.31.05.png)

那么现在的问题就是如何去拟合这组数据，如果直接使用之前的线性回归的方法显然是不合适的。但可以先保留之前的线性回归模型，我们建立模型是为了根据自变量去预测因变量的结果，这里要么等于0要么等于1.但与其预测用户是否会购买产品，不如去预测客户有多少概率去购买产品。对于图像中，那些超过了0和1的范围的，可以将其截去，用水平的直线进行代替。那么这样的曲线就可以预测不同年龄段购买的概率。但这个模型有几个缺点，前后的部分都是常数而且在某一点处存在一个折点。那么接下来看看Logistic Regression到底如何运作的。

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-02-12%20%E4%B8%8A%E5%8D%8810.44.31.png)

首先这条黑色的直线可以当作方程$y = b_0 + b_1 * x$，这里要运用一个机器学习领域非常常见的方程：Sigmoid Function(S函数)。
$$
 p = \frac{1}{1+e^{-y}}
$$
将这个方程带入到上述公式中，会得到：
$$
ln(\frac{p}{1-p}) = b_0 + b_1 * x
$$
图像如下：

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-02-12%20%E4%B8%8A%E5%8D%8810.42.18.png)

那么看看这里发生了什么事情。这里的绿色的曲线是我们表达概率$\hat{p}$的曲线。假设当前有几个数据分别是年龄20、30、40、50。那么其在纵坐标上对应的结果也可以标记出来，得到不同的概率。

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-02-12%20%E4%B8%8A%E5%8D%8810.50.33.png)

这边画出一条概率等于0.5的直线，在这条直线以下的我们可以预测该用户不会购买产品，再其以上的就可以预测用户会购买。

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-02-12%20%E4%B8%8A%E5%8D%8810.52.39.png)

接下来我们看看如何在python中实现Logistic Regression。

现在有一组数据集，反映的是社交网络的用户信息及用户看了投放广告后是否购买产品。（这里也只给出部分数据）

| User ID  | Gender | Age  | EstimatedSalary | Purchased |
| -------- | ------ | ---- | --------------- | --------- |
| 15624510 | Male   | 19   | 19000           | 0         |
| 15810944 | Male   | 35   | 20000           | 0         |
| 15668575 | Female | 26   | 43000           | 0         |

这里的步骤很简单，首先进行数据预处理，然后使用Logistic Regression来创建一个分类器，通过分类器来预测测试集的结果，然后使用混淆矩阵来判断这个分类器的效果。

```python
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import confusion_matrix

data_path = "./Social_Network_Ads.csv"

dataset = pd.read_csv(data_path)
X = dataset.iloc[:, [2, 3]].values
y = dataset.iloc[:, 4].values

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.25, random_state=0)


# feature scaling
sc_X = StandardScaler()
X_train = sc_X.fit_transform(X_train)
X_test = sc_X.transform(X_test)

# Fitting Logistic Regression to the Training set
classifier = LogisticRegression(random_state=0)
classifier.fit(X_train, y_train)

# Predicting the Test set results
y_pred = classifier.predict(X_test)

# Making the Confusion Matrix(混淆矩阵)
cm = confusion_matrix(y_test, y_pred)
```

其中cm的结果是：

```python
array([[65,  3],
       [ 8, 24]])
```

其中8和3分别代表预测错误的数据个数，65和24代表预测正确的个数。得到模型后将图像画出来：

```python
X_set, y_set = X_train, y_train
X1, X2 = np.meshgrid(np.arange(start=X_set[:, 0].min() - 1, stop=X_set[:, 0].max() + 1, step=0.01),
                     np.arange(start=X_set[:, 1].min() - 1, stop=X_set[:, 1].max() + 1, step=0.01))
plt.contourf(X1, X2, classifier.predict(np.array([X1.ravel(), X2.ravel()]).T).reshape(X1.shape),
             alpha=0.75, cmap=ListedColormap(('red', 'green')))
plt.xlim(X1.min(), X1.max())
plt.ylim(X2.min(), X2.max())
for i, j in enumerate(np.unique(y_set)):
    plt.scatter(X_set[y_set == j, 0], X_set[y_set == j, 1],
                c=ListedColormap(('orange', 'blue'))(i), label=j)
plt.title('Classifier (Training set)')
plt.xlabel('Age')
plt.ylabel('Estimated Salary')
plt.legend()
plt.show()
```

得到的图像如下：

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/scatter.png)

其中横纵坐标分别对应的是年龄和薪水，图中的橘黄色的点表示用户看到广告后没有购买，蓝色的表示购买了的。图中有一条很明显的直线区分了两种类别的数据，这条直线就叫做**预测边界**。逻辑回归分类器是个广义的分类器，对于线性分类器的预测边界都是线性的，二维情况下就是一条直线，三维就是一个平面。对于非线性的分类器，预测边界就不是直线了，后面会再讨论。

接下来在将测试集的数据画出来，得到新的图像，代码和训练集基本一样只是换了数据集：

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/classifier2.png)

如何画出图像的代码这里不做解释，这里可以看出图像上的结果和cm得到的值是一样的。

综上，可以观察代码中发现实际上使用到跟分类器相关的代码实际上只有一处：

```python
classifier = LogisticRegression(random_state=0)
classifier.fit(X_train, y_train)

# Predicting the Test set results
y_pred = classifier.predict(X_test)
```

那么之后如果需要创建其他分类器的时候实际上只需要修改这一处的代码即可。