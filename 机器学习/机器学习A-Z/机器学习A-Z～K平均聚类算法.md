# 机器学习A-Z～K平均聚类算法

本文来讲讲K平均聚类算法(K-Means Clustering),K Means算法是所有聚类算法中最经典的一种，因为它不断在直觉上容易理解，而且它的计算效率也是非常的高。

## 原理

在讲K-Means算法前我们先看看，这个算法能做什么。下面有一组数据，我们想要把数据分成若干个类，在某一类当中，这些数据的彼此之间的距离比较近。对于这个大问题，我们有两个小问题。第一个是，我们如何确定分的类的个数；第二个问题是，如何在确定类的个数的情况下，如何确定每个类中包含的元素。那么K-Means算法就可以自动帮助我们找到最佳的聚类的方式。如图所示，K-Means算法讲这些数据分成了红蓝绿三组。

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-03-15%20%E4%B8%8B%E5%8D%881.53.00.png)

那么我们就来看看K-Means算法的工作流程。

1. 选择我们想要的类的个数K；
2. 在平面上随机选择K个点，作为初始化类的中心点，不一定在原先数据当中；
3. 对于数据集中的每个点，要判断它属于我们之前K个中心点的哪一类。依据数据中的每个点对这K个点的距离的大小，找到最短的距离，那么就是每个数据点对应的类别，这一步可以称作是分配；
4. 重新计算一些新的中心点，就是应用之前分配的结果重新计算分配好的每个类当中的中心点；
5. 重新分配，如果重新分配的结果和之前分配的结果相同，则说明找到最佳的K-Means算法的结果，如果不同，那继续去第四步进行分配计算，直到找到最佳算法结果

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-03-15%20%E4%B8%8B%E5%8D%882.14.59.png)

下面从具体的例子来讲述这个步骤。假设有一组数据，我们要分配成两类，即K=2；然后随机选择两个点，分别计算每个点距离这两个点的距离。这里可以有个比较简单的计算方式，我们作出这两个点的垂直平分线，那么这个绿线上方的点都是离蓝色点比较近，下面的离红色比较近。

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-03-15%20%E4%B8%8B%E5%8D%882.18.58.png)

那么我们就把上面的点分作蓝组，下面的分为红组。目前的步骤相当于已经进行到第三步。接下来第四步，更新每组数据的中心点，那么我们就找到了新的中心点可以进行第五步，依据新的中心点，重新进行分配。

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-03-15%20%E4%B8%8B%E5%8D%882.22.26.png)

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-03-15%20%E4%B8%8B%E5%8D%882.23.08.png)

不断重复45步骤，直到分配的结果和前一步分配的结果是一致的。

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-03-15%20%E4%B8%8B%E5%8D%882.24.33.png)

K-Means算法可以以一种算法的方式告诉我们最佳的聚类的方式，这里就得到了左下方红组，右上方蓝组的这样一个结论。

## 随机初始化陷阱

现在看看初始点的选择对最终K-Meas聚类结果的影响。下面有一个例子，我们需要用K-Means算法对这组数据进行聚类，选择K=3。这里很明显有三类，我们这里就直接选择最佳的中心点并标记出这三类数据。

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-03-15%20%E4%B8%8B%E5%8D%882.29.47.png)

那么这里是我们肉眼看出来的三个中心点，但如果我们选择的不是这最佳的中心点，则需要重复上述的45步，比如选的是下面这三个中心点。

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-03-15%20%E4%B8%8B%E5%8D%882.32.19.png)

那么这时就需要对中心点进行位移，但由于这个位移是非常小的，所以新的分类结果和之前并不会有什么改变，所以算法就这样结束了。

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-03-15%20%E4%B8%8B%E5%8D%882.46.18.png)

这样得到的分类结果和之前那个显然是不同的。但这样就发生了同一组数据，却产生了两个不同的分类结果。区别就在于选择了不同的初始中心点。我们不好直接说哪一个分类算法更好，需要有一个方法来判断如何选择初始中心点。也就是说初始中心点不能随机进行选择了。现在有一个K-Means算法的更新版本，叫做K-Means++，它完美的解决了初始化中心点的陷阱，数学上来讲叫做局部最小值的一个陷阱。无论在R还是Python中，这个K-Means++都已经加入了算法当中，因此不用担心之后的代码实现会不会掉入这个陷阱。

## 选择类的个数

上文讲到的是选择中心点的陷阱，那么现在在谈谈如何选择类的个数。从直观上，上文中的图像大部分人应该很容易想到分为3组，也有的人可能想分为2组，但怎样选择才是最佳的分组方式是个需要好好研究的问题。首先来定义一个数学的量，组内平方和(WCSS)。

![WCSS](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-03-15%20%E4%B8%8B%E5%8D%883.06.39.png)

来看这个表达式，一共有3项，每一项代表对于每一组的平方和。比如第一项，就是对所有数据点对这一组中心点距离的平方。很显然，如果每一组的数据蜷缩的越紧，那么这个平方和就越小。

那么如果将这组数据分为1组，那么这个组内平方和只有一项，那么这个结果很显然会很大。如果分为2组，那么结果比1组的肯定要小，当分为3组时，得到的结果会更小。也就是说，随着分组的个数增加，这个组内平方和会逐渐变小。那么现在的问题来了，如何选择最合适的分组的个数？

这里要介绍一个法则，叫做手肘法则(The Elbow Method)。我们把随着分组个数的增加，WCSS的结果的图像画出来。

![手肘法则](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-03-15%20%E4%B8%8B%E5%8D%883.14.26.png)

找到最像手肘的这个点，这里就是3，那么这个点，就是最佳的分组的个数。这个曲线上可以看到，从1到2，和2到3时，下降的速率都是比较快的，但从3往后，下降的速率都是非常小的，那么我们要找的就是这样一个点，在到达这个点之前和从这个点开始的下降，速率的变化时最大的。

###  代码实现

我们这次要用到的数据集部分如下，反映的是一个购物商场的购物信息。最后一列Spending Score是购物商场根据客户的信息打出的客户的评分，分数越低意味着客户花的钱越少，越高以为着客户花的越多。商场希望通过对客户的年收入和购物指数来进行分群。

| CustomerID | Genre  | Age  | Annual Income (k$) | Spending Score (1-100) |
| ---------- | ------ | ---- | ------------------ | ---------------------- |
| 0001       | Male   | 19   | 15                 | 39                     |
| 0002       | Male   | 21   | 15                 | 81                     |
| 0003       | Female | 20   | 16                 | 6                      |
| 0004       | Female | 23   | 16                 | 77                     |

那么这个问题的自变量就是第三四列，年收入和购物指数。但它是个无监督学习，因此没有因变量。这里我们要用到的工具是sklearn.cluster中的KMeans类。

首先要计算各个分组的WCSS。这里我们计算组数从1到10的情况。

```python
wcss = []
for i in range(1, 11):
    kmeans = KMeans(n_clusters=i, max_iter=300, n_init=10, init='k-means++', random_state=0)
    kmeans.fit(X)
    wcss.append(kmeans.inertia_)
plt.plot(range(1, 11), wcss)
plt.title('The Elbow Method')
plt.xlabel('Number of Clusters')
plt.ylabel('WCSS')
plt.show()
```

这里的KMeans中的参数也解释下，n_clusters指的是分组数，max_iter指的是每一次计算时最大的循环个数，这里使用默认值300，n_init代表每一个做K平均算法时，会对多少组不同的中心值进行计算。init这个参数非常重要，指的是我们如何选择初始值，最简单的是random，即随机，但为了避免掉入随机初始值陷阱，这里使用k-means++。

拟合好后得到组间距离就是kmeans.inertia_。这样我们就可以画出对于不同的分组数，wcss的图像。

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/kmeans1.png)

那么通过手肘法则，可以得到最佳的分组个数是5组，则可以开始拟合数据。

```python
# Applying the k-means to the mall dataset
kmeans = KMeans(n_clusters=5, max_iter=300, n_init=10, init='k-means++', random_state=0)
y_kmeans = kmeans.fit_predict(X)
```

拟合好数据后，得到的y_means实际上就是0-4五个分组。我们来将分组后的图像画出来看看。

```python
# Visualizing the clusters
plt.scatter(X[y_kmeans == 0, 0], X[y_kmeans == 0, 1], s=100, c='red', label='Careful')
plt.scatter(X[y_kmeans == 1, 0], X[y_kmeans == 1, 1], s=100, c='blue', label='Standard')
plt.scatter(X[y_kmeans == 2, 0], X[y_kmeans == 2, 1], s=100, c='green', label='Target')
plt.scatter(X[y_kmeans == 3, 0], X[y_kmeans == 3, 1], s=100, c='cyan', label='Careless')
plt.scatter(X[y_kmeans == 4, 0], X[y_kmeans == 4, 1], s=100, c='magenta', label='Sensible')
plt.scatter(kmeans.cluster_centers_[:, 0], kmeans.cluster_centers_[:, 1], s=300, c='yellow', label='Centroids')
plt.title('Clusters of clients')
plt.xlabel('Annual Income (k$)')
plt.ylabel('Spending Score (1-100)')
plt.legend()
plt.show()
```

得到的图像如下，我们就可以根据图像来进行分析，给予不同的标签。

![分类结果](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/kmeans2.png)

以上，就是K-Means聚类算法的相关基础知识点。