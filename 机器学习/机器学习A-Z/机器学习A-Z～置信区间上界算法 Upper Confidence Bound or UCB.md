#机器学习A-Z～置信区间上界算法 Upper Confidence Bound or UCB

本文将要开始介绍机器学习中的强化学习， 这里首先应用一个多臂老虎机(The Multi-Armed Bandit Problem)问题来给大家解释什么是强化学习。

## 多臂老虎机问题

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-04-15%20%E4%B8%8A%E5%8D%8810.21.47.png)

如图所示，我们有几个单臂老虎机，组成一起我们就称作多臂老虎机，那么我们需要制定什么样的策略才能最大化得到的奖励。这里假设每个老虎机奖励的随机分布是不一样的。

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-04-15%20%E4%B8%8A%E5%8D%8810.22.01.png)

比如第一个分布，D1这个老虎机的分布大概率落在中间这部分，很小概率在两头的地方。假设用户是知道这些分布的，那么用户应当怎么选择？答案很简单，我们应当选择D5这个老虎机，因为它的平均值最高，而且有很大概率在靠右也就是正数范围内。但现在的问题是，用户实际上是不知道这些老虎机的概率分布的。那么我们需要一次次的尝试，尽可能快速的找到这五个老虎机的分布，并利用最佳的分布最大化收益。这个在机器学习上叫做，探索利用。 

探索和利用这两步实际上并不是那么的兼容，为了解释这个概念，这里引入一个定义，叫做遗憾。也就是我们知道D5的分布是最高的，那么最佳策略是应对不停的选择D5这个机器，但当我们不知道的时候，可能会选择其他的机器，当没有选择最佳机器时，都会得到一个得到的奖励和最佳奖励的差，这个差就是遗憾。

那么现在可以解释为什么探索和利用为什么并不是那么兼容，比如我们探索很多，每个机器玩了一百次，对于每个机器都能得到很好的分布估计，但同时也会带来很多的遗憾。如果探索比较少，可能会找不到最佳的分布，探索得到的结论可能是错的。所以说，最佳的策略应该是，通过探索找到最佳分布的老虎机，并且以一种快速而高效的方法找到这个老虎机，且不断的利用它来获得最大化的奖励。

在实际上生活或商业案例上也有很多相似的情形。比如可口可乐公司设计出了5个不同的广告，那么现在的问题就是不知道哪个广告是最好的，能带来最佳的用户转换率，即不知道哪一个投放后会吸引用户点击或购买产品。非常传统的方式就是把每个广告都投放到市场上几天，然后看看得到的结果。但这是很消耗人力和财力的，而且这本质上只进行了探索却没进行利用。因此我们需要对此进行利用，比如其中某个广告投放第一天就显然没有任何的效果，那么后面几天的投放的意义就不是很大了，可以考虑不再投放此广告来节省人力财力。即应用强化学习可以优化成本和支出，又可以得到最高的收益。

## 置信区间上界算法

现在来介绍强化学习中的一个算法，叫做置信区间上界算法(UCB)。这里依然用上述的多臂老虎机问题引出的广告案例。

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-04-15%20%E4%B8%8A%E5%8D%8810.44.18.png)

下面来描述下这个算法,现在假设我们知道五台老虎机的分布，那么我们会不断的选择D5这个老虎机。假如说不知道这几个的概率分布，那么就要开始我们的算法。

第一步，在我们开始之前，假设每个老虎机的概率分布是相同的，即平均值或期望是相同的。如图红色虚线表示平均值或期望，纵轴表示老虎机可能带来的收益。这些彩色的横线代表每个老虎机实际的平均或期望，这些期望我们是不知道的，这个问题的核心就是我们要进行不断的尝试去估算出每个老虎机的平均期望。

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-04-15%20%E4%B8%8A%E5%8D%8810.52.12.png)

置信区间，Confidence Bound，之前有讲过Confidence Interval，这两个词的意义是类似的。  这个Confidence Interval指的是当我们有一定的概率分布的时候，置信区间是和每个概率分布的累积分布曲线有关系。对于每个老虎机，我们讲置信区间，用灰色的方框表示。对于每个老虎机，我们按的概率有很大的概率是在这个区间当中的。我们每一轮将要选择的就是拥有最大的区间上界的老虎机，也就是顶上的这条线最大的老虎机，我们要在每一轮中选择它并按下它。在第一轮中，他们的区间上界是一样的。

比如选择3号老虎机，按下去，首先会发现区间所代表的方框下降了。因为按下去后我们会发现其给予的奖励，3号老虎机的实际期望比较低，那么观察得到的奖励也是比较低，那么重新计算观察到的所有平均值时，那么这个平均值就降低了。第二点，我们会发现这个置信区间变得更小了，因为比起上一轮的游戏，我们总共的观察次数变多了，也就是信心升高了，那么这个置信区间的长度就会变小。

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-04-15%20%E4%B8%8A%E5%8D%8811.02.23.png)

那么此时第三个机器的置信区间上界比其他的要低，所以下一轮要选择其他四个机器，比如选择第四个，那么它的置信区间上界会变高，区间大小会变小。

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-04-15%20%E4%B8%8A%E5%8D%8811.09.41.png)

在下一步，125三个机器的上界时一样的，因此可以在他们三个中间选一个，比如第一个，它的实际平均值比较低，但当我们按下老虎机的时候，它实际是个随机事件，因此可能高也可能低。虽然它的平均值，但假设此时运气比较好，投下去的钱翻倍了，那么他的区间上限变高，长度变小。

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-04-15%20%E4%B8%8A%E5%8D%8811.12.20.png)

再然后在25中间选择一个，比如第二个，此时他的置信区间上界变低，宽度变小。

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-04-15%20%E4%B8%8A%E5%8D%8811.13.28.png)

那么最后按下第五个机器，这时得到一个非常高的观察值，那么上限变高，宽度变小。但由于D5实际上是最佳的机器，那么就算置信区间变小了，但它的上界依然比其他的都要高。

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-04-15%20%E4%B8%8A%E5%8D%8811.16.54.png)

那么我接下来选择的老虎机依然是它，当我们选择了一个最佳的解时，我们可能在这个选择上停留很多轮，但由于对他的信心会越来越高，因此这个区间会越来越小，此时的区间上界可能就会变成不是最高。

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-04-15%20%E4%B8%8A%E5%8D%8811.24.04.png)

UCB算法有一个特点，当我们选择了一个机器很多次后，他的置信区间会变得很小，要给其他的机器一些机会，看看其他机器显示的观察结果所对应的新的置信区间上界是否会更高。这样经过很多轮后，最终D5的选择次数依然会很多很多，他的置信区间会越来越扁，一直到最终轮。

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-04-15%20%E4%B8%8A%E5%8D%8811.27.18.png)

以上就是置信区间算法的一个基本原理。

## 代码实现

这里用一个商业案例来做例子。假设有一个汽车公司为了一个车做了十个广告ad1-ad10，这个公司需要知道哪个广告投放后，点击率最高。这组数据集是在模拟环境得到的，若值为1则指的是这个用户点击了这个广告。总结一下问题就是，我们有十个广告，要决定哪个广告有最高的点击率，并对逐步得到的信息来决定，第n个用户投放哪个广告得到最高的点击数。我们先看看如果对于每一个用户，我们随机抽选广告，会得到怎样的点击数。这里提供一个随机选择的算法，不是此次算法的重点。

```python
# Importing the libraries
import matplotlib.pyplot as plt
import pandas as pd

# Importing the dataset
dataset = pd.read_csv('Ads_CTR_Optimisation.csv')

# Implementing Random Selection
import random
N = 10000
d = 10
ads_selected = []
total_reward = 0
for n in range(0, N):
    ad = random.randrange(d)
    ads_selected.append(ad)
    reward = dataset.values[n, ad]
    total_reward = total_reward + reward

# Visualising the results
plt.hist(ads_selected)
plt.title('Histogram of ads selections')
plt.xlabel('Ads')
plt.ylabel('Number of times each ad was selected')
plt.show()
```

运行后得到的ads_selected代表对于每一个用户，哪一个广告被选择，total_reward指的是所有的点击数。得到的图像表示每个广告被点击的次数。每个广告的到点击次数是接近的，大致在1000左右。

对于UCB算法，我们没有直接的工具包来使用，因此需要自己实现，下面是算法逻辑：

![](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-04-15%20%E4%B8%8A%E5%8D%8810.47.02.png)

这里就不解释太多，只贴代码，最终运行后得到的总的点击数为2178，明显高于上面用随机得到的结果。且查看ads_selected会发现编号4出现的次数最多，即第五个广告ad5。

```python


import matplotlib.pyplot as plt
import pandas as pd
import math

# import the dataset
dataset = pd.read_csv('Ads_CTR_Optimisation.csv')

# Implementing UCB
N = 10000
d = 10
ads_selected = []
numbers_of_selections = [0] * d
sums_of_rewards = [0] * d
total_reward = 0
for n in range(0, N):
    ad = 0
    max_upper_bound = 0
    for i in range(0, d):
        if numbers_of_selections[i] > 0:
            average_reward = sums_of_rewards[i] / numbers_of_selections[i]
            delta_i = math.sqrt(3/2 * math.log(n + 1) / numbers_of_selections[i])
            upper_bound = average_reward + delta_i
        else:
            upper_bound = 1e400
        if upper_bound > max_upper_bound:
            max_upper_bound = upper_bound
            ad = i
    ads_selected.append(ad)
    reward = dataset.values[n, ad]
    numbers_of_selections[ad] = numbers_of_selections[ad] + 1
    sums_of_rewards[ad] = sums_of_rewards[ad] + reward
    total_reward = total_reward + reward

# Visualising the results
plt.hist(ads_selected)
plt.title('Histogram of ads selections')
plt.xlabel('Ads')
plt.ylabel('Number of times each ad was selected')
plt.show()
```

以上，就是强化学习置信区间算法的相关基础知识。