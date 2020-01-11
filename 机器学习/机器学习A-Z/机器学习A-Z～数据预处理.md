# 机器学习A-Z～数据预处理

数据预处理是机器学习中非常重要的步骤，为了能正确的分析数据，得到最佳的机器学习算法，拿到数据后我们一般来说都需要对数据进行预处理。数据预处理包括以下几个步骤：

1. 导入数据集
2. 处理缺失数据
3. 分类数据
4. 数据分成训练集和测试集
5. 特征缩放

## 导入数据集

我们当前有一组数据集如下：

```python
Country,Age,Salary,Purchased
France,44,72000,No
Spain,27,48000,Yes
Germany,30,54000,No
Spain,38,61000,No
Germany,40,,Yes
France,35,58000,Yes
Spain,,52000,No
France,48,79000,Yes
Germany,50,83000,No
France,37,67000,Yes
```

这组数据反映的是用户的国籍、年龄、薪水对是否购买该商品的影响。导入数据集我们一般要用到pandas包，对于这组数据而言，前三列都是自变量，最后一列是因变量，即我们要进行预测的结果。那么导入数据的代码如下（这里我们先把后面要用到的包导入）：

```python
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
data_path = '../data/Data.csv'

#import the dataset
dataset = pd.read_csv(data_path)
X = dataset.iloc[:,:-1].values
y = dataset.iloc[:,3].values
```

X为数据中所有行的除了最后一列的所有数据，y则是最后一列的结果。这样，数据集就被导入进来了。

## 处理缺失数据

仔细观察这组数据集，我们会发现有几行的数据出现了数据缺失的情况，比如第五行数据中就缺少了salary的信息。那么对于这种缺失的数据应该怎么处理呢？以下有两种方法：

- 删除缺失的数据（操作简单但风险很大，容易删除重要的数据）
- 取该列的平均值来代替缺失数据

那么如何用python来处理呢，我们要用到的就是强大的sklearn包，其中Imputer类可以用来处理缺失数据，代码如下所示：

```python
# Taking care of missing data
from sklearn.preprocessing import Imputer
imputer = Imputer(missing_values = 'NaN', strategy = 'mean', axis = 0)
imputer = imputer.fit(X[:, 1:3])
X[:, 1:3] = imputer.transform(X[:, 1:3])
```

处理之后我们再查看X的数据，会发现缺失数据已经被该列的平均值所填充。

## 分类数据

仔细观察这组数据，对于年龄和薪水都是数值，而国家却是各个国家的类别，是否购买这边只有购买和未购买两个类别。在机器学习中我们本质上是用方程对数据进行不同的处理，那么针对这种不同的类别，需要将其转换成不同的数值，来带入我们的方程里面。在python中，依然是使用sklearn包，要用到的工具是LabelEncoder。代码如下：

```python
# Encoding categorical data
# Encoding the Independent Variable
from sklearn.preprocessing import LabelEncoder, OneHotEncoder
labelencoder_X = LabelEncoder()
X[:, 0] = labelencoder_X.fit_transform(X[:, 0])
onehotencoder = OneHotEncoder(categorical_features = [0])
X = onehotencoder.fit_transform(X).toarray()
# Encoding the Dependent Variable
labelencoder_y = LabelEncoder()
y = labelencoder_y.fit_transform(y)
```

在我们使用LabelEncoder进行转换后，第一列的数据会将国家变成0，1，2这些数值，但这样会带来一个问题，原本这些国家仅仅只是表示了不同的类别，但转换成数字后会无意的将其进行了排序，而这些排序是无意义的，那么这里要使用**虚拟编码**来解决这种问题。

所谓虚拟编码，如下图所示，原本数据中的国籍有三个类别法国、西班牙和德国，那么我们可以将其分为三列，每一列代表用户是否是这一列的分组，比如如果该用户是法国人，那么法国这一列就为1，而另外两列为0。

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-01-03%20%E4%B8%8B%E5%8D%882.56.50.png)

使用虚拟编码的方式，可以将原来的一列变量变为了三列变量，但数值之间没有了顺序的区别。使用python的话这里用到的工具是OneHotEncoder，代码上面也已经给出，categorical_features指的是要处理的是哪一列。当然别忘了最后一列，由于最后一列是因变量，python的函数可以自动将这一列识别为分类数据，因此不需要再使用OneHotEncoder，直接用LabelEncoder处理即可。

## 数据分成训练集和测试集

再切分数据前，我们需要谈谈什么是训练集和测试集，那么首先，说一下机器学习这个名词代表的意义。机器学习，顾名思义，就是让机器学习数据之间的关系，并可以用学习到的结果对新的数据进行预测。那么学习的过程就是不断通过数据来修改自己的公式。机器从训练集中的数据学习数据的相关性，学习完之后，机器需要在新的数据即测试集来测试自己训练的成果怎样。在python中，有非常简单的方法来切分数据集，如下所示：

```python
#spliting the dataset to trainset and testset
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size = 0.2, random_state = 0)
```

其中test_size指的就是测试集所占的比例，最后的random_state指的是切分方式，一样的random_state切出来的结果是一样的。关于训练集、测试集的概念在我的另一篇博客： [吴恩达机器学习笔记-应用机器学习的建议](http://www.leafw.cn/2018/09/26/%E5%90%B4%E6%81%A9%E8%BE%BE%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0-%E5%BA%94%E7%94%A8%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E7%9A%84%E5%BB%BA%E8%AE%AE/) 中有提及，是吴恩达老师的课程的笔记，可以额外参考一下。

## 特征缩放

回过头来看下这组数据，这里重点关注下年龄和薪水，年龄的值基本在30-50浮动，而薪水在50000-80000之间浮动。机器学习中有个术语叫做**欧式距离**，所谓欧式距离可以看作是平面中两个点之间的距离，显然是两者横纵坐标差的平方和相加后开方。那么问题来了，如果这里的薪水和年龄分别为横纵坐标，那么薪水的坐标差的平方相对于年龄的来说差距非常大，那么年龄对这个结果的影响会变得很小。因此，我们需要将年龄和薪水缩放到同一个数量级上面，有些算法可能没用到欧式距离，但进行特征缩放后，算法的收敛速度会变快很多（比如决策树）。

接下来看看如何对数据进行特征缩放，以下有两种算法：Standardisation和Normalisation。

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-01-03%20%E4%B8%8B%E5%8D%883.34.23.png)

其中标准化中，mean(x)表示x的平均值，StandardDeviation表示标准方差，即衡量一列数据中浮动性大概有多少。那么这里得到的$x_{stand}$ 表示的就是平均值为0，方差为1的分布。

在python中的代码如下所示：

```python
#feature scaling
sc_X = StandardScaler()
X_train = sc_X.fit_transform(X_train)
X_test = sc_X.transform(X_test)
```

记得上面的分类数据中，对第一列和最后一列都进行了虚拟编码，这些虚拟变量是否需要进行特征缩放呢，这个需要针对不同的场景进行分析，这里的变量值只有0和1，看起来已经进行了特征缩放，因此可以不做特征缩放，但进行特征缩放后可能会对算法的性能有所提升，这里就都进行特征缩放。对自变量进行特征缩放后，在看看因变量，这里的因变量代表的是不同的类别，那么这里就不需要了，若是回归问题，那么因变量就可能需要进行特征缩放。

## 总结

以上就是数据预处理中的各个步骤，针对实际情况，不是所有的步骤都一定需要做。针对不同的场景选择需要的步骤即可。