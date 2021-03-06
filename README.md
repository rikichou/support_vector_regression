
# Support Vector Regression

## 1. Kernel Ridge Regression

上一节我们讲了将kernel的方法应用于logistics regression。并且介绍了representer theorem，说的是只要是使用L2 regularization的线性模型都可以使用kernel trick。那我们本小节的任务就是将kernel trick应用于ridge regression。

<img src="6-1.png" width="60%">

因为我们在讲解linear regression的时候说过，我们可以得到w的analytic solution。同样的我们也想得到kernel版本的ridge regression的analytic solution。

### 1.1 解Kernel Ridge Regression

<img src="6-2.png" width="60%">

首先将W用Z的线性组合来表示，再将此表达式代入到目标方程中，将一个W的问题转换为一个解β的问题，同时使用kernel trick将所以的Z空间的内积用kernel function表示出来。为了方便表示，我们写成向量的形式(其中右面的error部分可以写作：||Y-Kβ||^2，然后再打开平方得到上图的式子)

下一步就是取β的梯度 = 0，然后把β表示出来：

<img src="6-3.png" width="60%">

因为K是半正定矩阵，λ>0，所以逆矩阵一定存在。

*注意：我们的K是dense的，那么求逆矩阵的话会花费大量的时间。

所以现在我们有一个更容易的方法来做Non-linear regression了。

### 1.2 linear vs Kernel Ridge Regression

<img src="6-4.png" width="60%">

* 对于求逆部分，linear的矩阵大小是dxd的，而kernel的矩阵大小是NxN的。
* linear的缺点是比较受限制，不能做复杂的曲线(需要重新手动特征转换)。而kernel可以通过修改K来调整特征的转换，自由度更高。
* 当n >> d的话，linear可能是一个比较好的方式。反之，kernel更好

## 2. Support Vector Regression

### 2.1 动机

我们知道ridge regression可以用于做分类，那我们的kernel ridge regression当然也可以用于做分类，而且我们还有一个专业的名词来形容这样的事情叫做--least-squares SVM

<img src="6-5.png" width="60%">

下面我们来比较soft-margin SVM和Least-Squares SVM，由图中可看出：

* 两者形成的边界大概一致
* 但是Least-Squares SVM拥有更多的support vector。因为kernel ridge regression求出来的那个β向量很多都不是0（kernel logistics regression也是这样子）.这就代表我们在分类预测的时候会更慢。我们可能不会选择least-squares SVM来做分类。会选择sparse a的soft-margin来做。

那这就引入了一个问题，我们可不可以做Kernel Ridge Regression同时使向量β是sparse的？

(总结起来就是：想要用Kernel的方法来解决Ridge Regression，但是有希望β是sparse的)


### 2.2 Tube Regression

所以我们会考虑一个叫做tube Regression的方案。

<img src="6-6.png" width="60%">

传统的regression的error measure是计算预测点到实际的点的距离。那我们现在就改变一下，设立一个中立区，也就是我们的tube的宽度是ε，如果我们的预测值和实际值的差小于ε，那么我们认为错误为0，否则错误为差值减去ε。我们把这样的错误衡量叫做ε insensitive error。

那么我们接下来的事情就是要将这个L2 regularied tube regression通过向soft-margin SVM一样的推导来得到一个sparse 的β解。

### 2.3 Tube vs Squared Regression

在计算Tube Regression之前，先来比较一下这两种regression。

<img src="6-7.png" width="60%">

* 如上图，两种regression都是指的红色直线的部分，但是右边计算的是错误的平方。
* 还可以看出，在s和Y差别很大的时候，squared error的值将会非常大，就是产生一个非常大的惩罚，如果这个error是噪声的话，那就意味着受噪声的影响很大。所以tube regression受噪声的影响相对较小。

### 2.4 solve L2 Regularized Tube Regression

我们要解决的是L2 Regularized Tube Regression的问题，但是我们需要模仿soft-margin。

<img src="6-8.png" width="60%">

* 虽然我们的问题是一个非限制的最优化问题，但是表达式中的max确实不可微的。可以模仿soft-margin来加入一个tube violation的量，虽然这样做为因为限制条件，但这确是一个可接的QP问题。
* 我们想引入kernel的技巧，虽然可以使用representer theorem来引入kernel，但是这样做不能够保证β是sparse的。但是我们的初衷就是为了β是sparse的。所以我们可以模仿soft-margin SVM，通过得出原始问题的对偶问题来引入kernel方法。而且还可以利用KKT条件来保证我们的β是sparse的。

然后为了和SVM的形式看起来更匹配，做了一些形式上的改变（注意，这里的regularizer的W并没有b，和我们之前的regularizer不同，老师的解释是可能是历史原因）


#### 2.4.1 standard support vector regression primal(标准的支持向量回归的原始形式)

<img src="6-9.png" width="60%">

所以我们就按照2.4节所说的步骤进行。

1. 首先为了将max去掉，我们采用和soft-margin一样的想法，引入一个松弛变量ξ，这个变量记录了每个点违反tube的数量，这个C就是用于权衡复杂边界和更少的错误。
2. 去掉max后，但是我们条件中会有一个绝对值，有绝对值的话就不是线性的条件，就不符合QP的要求，那当然不行了。所以我们第二部就是要将绝对值去掉。我们会引进两种ξ的错误，一种是当y-wz-b > ε，我们会引入ξ^来表示违反的量，如果y-wz-b < -ε的话，那么违反的量用ξ>（打不出来）来表示。所以此时条件就变成了线性的了。

经过上面那个步骤以后，我们的Tube Regression也就符合了QP的标准。我们这把这个新的形式叫做Support Vector Regression的原始形式

<img src="6-10.png" width="60%">

* 这个C参数可以理解为：上面说了C参数作为一种权衡，权衡复杂边界和更少的错误的。
* 我们有更多的ε参数需要选择。

#### 2.4.2 standard support vector regression dual(标准的支持向量回归的对偶形式)

<img src="6-11.png" width="60%">

如何将原始问题转为对偶问题呢？首先需要的是拉格朗日乘子，如图所示。然后就是同soft-margin SVM一样的过程，对我们的变量做微分，利用KKT condition来替换掉那些变量，然后就得到了一个新的QP问题。在这里不再重复推导的过程直接给出一些结果。

#### 2.4.3 SVM dual和SVR dual的相似性

<img src="6-12.png" width="60%">

上图总结了SVM和SVR的dual的相似性，最后说明可以用相似的QPsolver来解决SVR的问题。

#### 2.4.4 SVR的解的稀疏性

这是我们使用SVR而不用kernel ridge regression的原因。下面就来证明其最优解具有稀疏性。

<img src="6-13.png" width="60%">

证明β的稀疏性也就是找出β在什么情况下等于零，利用的工具当然也是complementary slackness

考虑一种情况：点严格位于tube内--根据complementary slackness，这种情况的βn等于0，而对于回归来说，这种情况的数据点占大多数。


## 3. 核模型的汇总

### 3.1 线性模型的分布

<img src="6-14.png" width="60%">

如上图所示使我们所学习过的所有的线性模型，然后第二排是台湾大学实验室开发的LIBLINEAR里面的核心，如果要解决linear问题，建议使用该软件包。

### 3.2 kernel模型的分布

<img src="6-15.png" width="60%">

第四排就是我们的线性模型的kernel版本，在我们大名鼎鼎的LIBSVM中均有实现。

实用度：
* 第一排的算法在实际上比较少用，因为他的表现相对于其他没那么好，比如对线性不可分数据啊，模型过于简单啊等等
* 第三排因为最优解不稀疏的缺点，所以很少使用
