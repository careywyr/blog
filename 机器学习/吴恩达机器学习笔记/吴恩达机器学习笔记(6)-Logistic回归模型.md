## 回归函数

在逻辑回归模型中我们不能再像之前的线性回归一样使用相同的代价函数，否则会使得输出的结果图像呈现波浪状，也就是说不再是个凸函数。代价函数的表达式之前有表示过,这里我们把 1/2 放到求和里面来。

$$
J(\theta) = \frac{1}{m}\sum_{i=1}^{m}\frac{1}{2}(h_\theta(x^(i))-y^(i))^2
$$

这里的求和部分我们可以表示为：

$$
Cost(h_\theta(x(i)),y)
$$

很显然，如果我们把在之前说过的分类问题的假设函数带进去，即$h_\theta(x) = \frac{1}{1+e^{-z}}$，得到的结果可能就是上述所说的不断起伏的状况。如果这里使用梯度下降法，不能保证能得到全局收敛的值，这个函数就是所谓的非凸函数。因此我们需要找一个不同的代价函数，并且是个凸函数，使得我们可以使用好的算法并找到全局最小值。这个代价函数如下所示：

![  ][1]

我们可以画出$J(\theta)$的图像如下所示，分别为当 y=1 和 y=0 时的图像：

![][2]

![][3]

那根据图像我们可以得出结论：

$$
Cost(h_\theta(x),y) = 0 \quad if \quad  h_\theta(x) = y \\ Cost(h_\theta(x),y) \rightarrow \infty \quad if \quad  y = 0 \quad and \quad h_\theta(x) \rightarrow 1 \\ Cost(h_\theta(x),y) \rightarrow \infty \quad if \quad  y = 1 \quad and \quad h_\theta(x) \rightarrow 0
$$

如果 y 的值为 0，则当假设函数输出 0 时代价函数也为 0，如果假设函数趋向于 1，则代价函数趋向于无穷大；如果 y 的值为 1，则当假设函数输出 1 时代价函数为 0，如果假设函数趋向于 0，则代价函数趋向于无穷大。

上述的代价函数可以简化成如下所示：

$$
Cost(h_\theta(x),y) = -ylog(h_\theta(x)) - (1-y)log(1-h_\theta(x))
$$

若 y=1 则第二项为 0，若 y=0 则第一项为 0，因此此等式和上述的分段描述的函数是等价的。
那么可以写出完整的代价函数：

![][4]

用向量表示则是：

![][5]

## 梯度下降

我们之前有讲过梯度下降算法的概念：

![][6]

将公式带入进行导数计算后得到：

![][7]

用向量法表示：

![][8]

当然要注意这个算法和我们在线性回归中的方法一样，我们需要同步的更新所有的$\theta$的值。

## 高级优化

换个角度来看梯度下降的话，实际上就是我们有一个代价函数$J(\theta)$，我们需要使它最小化。我们要做的就是编写代码来计算输入$\theta$时，得到$J(\theta)$和$J(\theta)$对$\theta_j$的偏导。梯度下降本质上就是不断重复这个过程来更新参数$\theta$。也就是说梯度下降就是编写代码计算出$J(\theta)$和$J(\theta)$的偏导带入梯度下降公式中，然后它就可以为我们最小化这个函数。实际上对于梯度下降而言，不是一定要计算出$J(\theta)$的值，但如果为了能更好的监控到$J(\theta)$的收敛性，需要自己编写代码来计算代价函数和偏导性。
除了梯度下降之外还有一些其他算法来优化代价函数。包括共轭梯度法 BFGS 和 L-BFGS 还有 Conjugate gradient。这些算法相对梯度下降算法更高级但同时也是更加的复杂。这些算法都暂时不在当前学习的范畴内。不过这里可以简单说一下他们的特性：

- 不需要手动选择学习速率$\alpha$
- 往往比梯度下降算法要快一点
- 但相对来说更加的复杂

我们可以通过写一个方法来计算$J(\theta)$和$\frac{\partial}{\partial\theta_j}J(\theta)$的值：

    function [jVal, gradient] = costFunction(theta)
        jVal = [...code to compute J(theta)...];
        gradient = [...code to compute derivative of J(theta)...];
    end

其中 jVal 就是我们代价函数的值，第二个就是梯度。下图就是一个简单的例子以及对应的 octave 的代码：

![][9]

## 多类别分类问题

接下来我们来介绍多类别的分类问题，也就是说我们的类别可能不止两种。比如天气可能是晴天、阴天、雨天，或者邮件分类可能包括工作、朋友、垃圾邮件等等。那这个时候的 y={0,1}就不再适用了，而是 y={0,1,2...,n}。下图中，左边代表的是之前说的二元分类问题的数据集，有两种不同的符号表示；右边的是多类别分类问题，用三种不同的符号表示三个不同的类别中的样本。

![][10]

我们要做的就是找到合适的算法来进行分类，在二元分类问题中我们知道了通过回归来解决分类问题，利用直线将数据分成正类和负类。利用一对多的思想我们同样可以将其应用在多分类问题上。下图为一个简单的有三种类别的分类问题。

![][11]

我们可以创建一个新的数据集将类别 1 当做正类，类别 2 和 3 设定为负类，拟合出一个逻辑回归分类器$h^(1)_\theta(x)$。同样的对类别 2 和类别 3 也可以拟合出其他逻辑回归分类器$h^(2)_\theta(x)$和$h^(3)_\theta(x)$。总的来说就是$h^(i)_\theta(x) = P(y=i|x;\theta) \quad (i=1,2,3))$。即给定 x 和$\theta$时 y=i 的概率。当输入一个新的 x 时，我们要做的就是在这三个分类器中输入 x，然后选择 h 最大的类别，即选出三个分类器中可信度最高的一个

以上，为吴恩达机器学习 logistic regression 章节的内容。

[1]: https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/logi1.png
[2]: https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/logi2.png
[3]: https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/logi3.png
[4]: https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/logi4.png
[5]: https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/logi5.png
[6]: https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/logi6.png
[7]: https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/logi7.png
[8]: https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/logi8.png
[9]: https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/logi9.png
[10]: https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/logi10.png
[11]: https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/logi11.png
