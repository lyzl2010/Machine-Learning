# 1. CART分类树算法的最优特征选择方法

我们知道，在ID3算法中我们使用了信息增益来选择特征，信息增益大的优先选择。在C4.5算法中，采用了信息增益比来选择特征，以减少信息增益容易选择特征值多的特征的问题。但是无论是ID3还是C4.5,都是基于信息论的熵模型的，这里面会涉及大量的对数运算。能不能简化模型同时也不至于完全丢失熵模型的优点呢？有！CART分类树算法使用基尼系数来代替信息增益比，基尼系数代表了模型的不纯度，基尼系数越小，则不纯度越低，特征越好。这和信息增益\(比\)是相反的。

具体的，在分类问题中，假设有K个类别，第k个类别的概率为$$p_k$$, 则基尼系数的表达式为：

$$Gini(p) = \sum\limits_{k=1}^{K}p_k(1-p_k) = 1- \sum\limits_{k=1}^{K}p_k^2$$

如果是二类分类问题，计算就更加简单了，如果属于第一个样本输出的概率是p，则基尼系数的表达式为：

Gini\(p\)=2p\(1−p\)

对于个给定的样本D,假设有K个类别, 第k个类别的数量为$$C_k$$,则样本D的基尼系数表达式为：

$$Gini(D) = 1-\sum\limits_{k=1}^{K}(\frac{|C_k|}{|D|})^2$$

特别的，对于样本D,如果根据特征A的某个值a,把D分成D1和D2两部分，则在特征A的条件下，D的基尼系数表达式为：

$$Gini(D,A) = \frac{|D_1|}{|D|}Gini(D_1) + \frac{|D_2|}{|D|}Gini(D_2)$$

大家可以比较下基尼系数表达式和熵模型的表达式，二次运算是不是比对数简单很多？尤其是二类分类的计算，更加简单。但是简单归简单，和熵模型的度量方式比，基尼系数对应的误差有多大呢？对于二类分类，基尼系数和熵之半的曲线如下：

![](http://images2015.cnblogs.com/blog/1042406/201611/1042406-20161111105202170-1563882835.jpg)

从上图可以看出，基尼系数和熵之半的曲线非常接近，仅仅在45度角附近误差稍大。因此，基尼系数可以做为熵模型的一个近似替代。而CART分类树算法就是使用的基尼系数来选择决策树的特征。同时，为了进一步简化，CART分类树算法每次仅仅对某个特征的值进行二分，而不是多分，这样CART分类树算法建立起来的是二叉树，而不是多叉树。这样一可以进一步简化基尼系数的计算，二可以建立一个更加优雅的二叉树模型。

# 2. CART分类树算法对于连续特征和离散特征处理的改进

对于CART分类树连续值的处理问题，其思想和C4.5是相同的，都是将连续的特征离散化。唯一的区别在于在选择划分点时的度量方式不同，C4.5使用的是信息增益，则CART分类树使用的是基尼系数。

具体的思路如下，比如m个样本的连续特征A有m个，从小到大排列为a1,a2,...,am,则CART算法取相邻两样本值的中位数，一共取得m-1个划分点，其中第i个划分点Ti表示为：$$T_i = \frac{a_i+a_{i+1}}{2}$$。对于这m-1个点，分别计算以该点作为二元分类点时的基尼系数。选择基尼系数最小的点作为该连续特征的二元离散分类点。比如取到的基尼系数最小的点为at,则小于at的值为类别1，大于at的值为类别2，这样我们就做到了连续特征的离散化。要注意的是，与离散属性不同的是，如果当前节点为连续属性，则该属性后面还可以参与子节点的产生选择过程。

对于CART分类树离散值的处理问题，采用的思路是不停的二分离散特征。

回忆下ID3或者C4.5，如果某个特征A被选取建立决策树节点，如果它有A1,A2,A3三种类别，我们会在决策树上一下建立一个三叉的节点。这样导致决策树是多叉树。但是CART分类树使用的方法不同，他采用的是不停的二分，还是这个例子，CART分类树会考虑把A分成{A1}和{A2,A3},{A2}和{A1,A3},{A3}和{A1,A2}三种情况，找到基尼系数最小的组合，比如{A2}和{A1,A3},然后建立二叉树节点，一个节点是A2对应的样本，另一个节点是{A1,A3}对应的节点。从描述可以看出，如果离散特征A有n个取值，则可能的组合有n\(n-1\)/2种。同时，由于这次没有把特征A的取值完全分开，后面我们还有机会在子节点继续选择到特征A来划分A1和A3。这和ID3或者C4.5不同，在ID3或者C4.5的一棵子树中，离散特征只会参与一次节点的建立。

# 3. CART分类树建立算法的具体流程

上面介绍了CART算法的一些和C4.5不同之处，下面我们看看CART分类树建立算法的具体流程，之所以加上了建立，是因为CART树算法还有独立的剪枝算法这一块，这块我们在第5节讲。

算法输入是训练集D，基尼系数的阈值，样本个数阈值。

输出是决策树T。

我们的算法从根节点开始，用训练集递归的建立CART树。

1\) 对于当前节点的数据集为D，如果样本个数小于阈值或者没有特征，则返回决策子树，当前节点停止递归。

2\) 计算样本集D的基尼系数，如果基尼系数小于阈值，则返回决策树子树，当前节点停止递归。

3\) 计算当前节点现有的各个特征的各个特征值对数据集D的基尼系数，对于离散值和连续值的处理方法和基尼系数的计算见第二节。缺失值的处理方法和上篇的C4.5算法里描述的相同。

4\) 在计算出来的各个特征的各个特征值对数据集D的基尼系数中，选择基尼系数最小的特征A和对应的特征值a。根据这个最优特征和最优特征值，把数据集划分成两部分D1和D2，同时建立当前节点的左右节点，做节点的数据集D为D1，右节点的数据集D为D2.

5\) 对左右的子节点递归的调用1-4步，生成决策树。

对于生成的决策树做预测的时候，假如测试集里的样本A落到了某个叶子节点，而节点里有多个训练样本。则对于A的类别预测采用的是这个叶子节点里概率最大的类别。

# 4. CART回归树建立算法

CART回归树和CART分类树的建立算法大部分是类似的，所以这里我们只讨论CART回归树和CART分类树的建立算法不同的地方。

首先，我们要明白，什么是回归树，什么是分类树。两者的区别在于样本输出，如果样本输出是离散值，那么这是一颗分类树。如果果样本输出是连续值，那么那么这是一颗回归树。

除了概念的不同，CART回归树和CART分类树的建立和预测的区别主要有下面两点：

1\)连续值的处理方法不同

2\)决策树建立后做预测的方式不同。

对于连续值的处理，我们知道CART分类树采用的是用基尼系数的大小来度量特征的各个划分点的优劣情况。这比较适合分类模型，但是对于回归模型，我们使用了常见的均方差的度量方式，CART回归树的度量目标是，对于任意划分特征A，对应的任意划分点s两边划分成的数据集D1和D2，求出使D1和D2各自集合的均方差最小，同时D1和D2的均方差之和最小所对应的特征和特征值划分点。表达式为：

$$\underbrace{min}_{A,s}\Bigg[\underbrace{min}_{c_1}\sum\limits_{x_i \in D_1(A,s)}(y_i - c_1)^2 + \underbrace{min}_{c_2}\sum\limits_{x_i \in D_2(A,s)}(y_i - c_2)^2\Bigg]$$

其中，c1为D1数据集的样本输出均值，c2为D2数据集的样本输出均值。

对于决策树建立后做预测的方式，上面讲到了CART分类树采用叶子节点里概率最大的类别作为当前节点的预测类别。而回归树输出不是类别，它采用的是用最终叶子的均值或者中位数来预测输出结果。

除了上面提到了以外，CART回归树和CART分类树的建立算法和预测没有什么区别。

# 5. CART树算法的剪枝

CART回归树和CART分类树的剪枝策略除了在度量损失的时候一个使用均方差，一个使用基尼系数，算法基本完全一样，这里我们一起来讲。

由于决策时算法很容易对训练集过拟合，而导致泛化能力差，为了解决这个问题，我们需要对CART树进行剪枝，即类似于线性回归的正则化，来增加决策树的返回能力。但是，有很多的剪枝方法，我们应该这么选择呢？CART采用的办法是后剪枝法，即先生成决策树，然后产生所有可能的剪枝后的CART树，然后使用交叉验证来检验各种剪枝的效果，选择泛化能力最好的剪枝策略。

也就是说，CART树的剪枝算法可以概括为两步，第一步是从原始决策树生成各种剪枝效果的决策树，第二部是用交叉验证来检验剪枝后的预测能力，选择泛化预测能力最好的剪枝后的数作为最终的CART树。

首先我们看看剪枝的损失函数度量，在剪枝的过程中，对于任意的一刻子树T,其损失函数为：

$$C_{\alpha}(T_t) = C(T_t) + \alpha |T_t|$$

其中，α为正则化参数，这和线性回归的正则化一样。C\(Tt\)为训练数据的预测误差，分类树是用基尼系数度量，回归树是均方差度量。\|Tt\|是子树T的叶子节点的数量。

当α=0时，即没有正则化，原始的生成的CART树即为最优子树。当α=∞时，即正则化强度达到最大，此时由原始的生成的CART树的根节点组成的单节点树为最优子树。当然，这是两种极端情况。一般来说，α越大，则剪枝剪的越厉害，生成的最优子树相比原生决策树就越偏小。对于固定的α，一定存在使损失函数Cα\(T\)最小的唯一子树。

看过剪枝的损失函数度量后，我们再来看看剪枝的思路，对于位于节点t的任意一颗子树Tt，如果没有剪枝，它的损失是

$$C_{\alpha}(T_t) = C(T_t) + \alpha |T_t|$$

如果将其剪掉，仅仅保留根节点，则损失是

$$C_{\alpha}(T) = C(T) + \alpha$$

当α=0或者α很小时，$$C_{\alpha}(T_t) < C_{\alpha}(T)$$, 当α增大到一定的程度时
$$
C_{\alpha}(T) = C(T) + \alpha
$$
。当α继续增大时不等式反向，也就是说，如果满足下式：

$$\alpha = \frac{C(T)-C(T_t)}{|T_t|-1}$$

Tt和T有相同的损失函数，但是T节点更少，因此可以对子树Tt进行剪枝，也就是将它的子节点全部剪掉，变为一个叶子节点T。

最后我们看看CART树的交叉验证策略。上面我们讲到，可以计算出每个子树是否剪枝的阈值α，如果我们把所有的节点是否剪枝的值α都计算出来，然后分别针对不同的α所对应的剪枝后的最优子树做交叉验证。这样就可以选择一个最好的α，有了这个α，我们就可以用对应的最优子树作为最终结果。

好了，有了上面的思路，我们现在来看看CART树的剪枝算法。

输入是CART树建立算法得到的原始决策树T。

输出是最优决策子树Tα。

算法过程如下：

1）初始化$$\alpha_{min}= \infty$$， 最优子树集合ω={T}。

2）从叶子节点开始自下而上计算各内部节点t的训练误差损失函数Cα\(Tt\)（回归树为均方差，分类树为基尼系数）, 叶子节点数\|Tt\|，以及正则化阈值$$\alpha= min\{\frac{C(T)-C(T_t)}{|T_t|-1}, \alpha_{min}\}$$, 更新$$\alpha_{min}= \alpha$$

3\) 得到所有节点的α值的集合M。

4）从M中选择最大的值αk，自上而下的访问子树t的内部节点，如果$$\frac{C(T)-C(T_t)}{|T_t|-1} \leq \alpha_k$$时，进行剪枝。并决定叶节点t的值。如果是分类树，则是概率最高的类别，如果是回归树，则是所有样本输出的均值。这样得到αk对应的最优子树Tk

5）最优子树集合$$\omega=\omega \cup T_k$$$$M= M -\{\alpha_k\}$$。

6\) 如果M不为空，则回到步骤4。否则就已经得到了所有的可选最优子树集合ω.

7\) 采用交叉验证在ω选择最优子树Tα

# 6. CART算法小结

上面我们对CART算法做了一个详细的介绍，CART算法相比C4.5算法的分类方法，采用了简化的二叉树模型，同时特征选择采用了近似的基尼系数来简化计算。当然CART树最大的好处是还可以做回归模型，这个C4.5没有。下表给出了ID3，C4.5和CART的一个比较总结。希望可以帮助大家理解。

| 算法 | 支持模型 | 树结构 | 特征选择 | 连续值处理 | 缺失值处理 | 剪枝 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| ID3 | 分类 | 多叉树 | 信息增益 | 不支持 | 不支持 | 不支持 |
| C4.5 | 分类 | 多叉树 | 信息增益比 | 支持 | 支持 | 支持 |
| CART | 分类，回归 | 二叉树 | 基尼系数，均方差 | 支持 | 支持 | 支持 |

看起来CART算法高大上，那么CART算法还有没有什么缺点呢？有！主要的缺点我认为如下：

1）应该大家有注意到，无论是ID3, C4.5还是CART,在做特征选择的时候都是选择最优的一个特征来做分类决策，但是大多数，分类决策不应该是由某一个特征决定的，而是应该由一组特征决定的。这样绝息到的决策树更加准确。这个决策树叫做多变量决策树\(multi-variate decision tree\)。在选择最优特征的时候，多变量决策树不是选择某一个最优特征，而是选择最优的一个特征线性组合来做决策。这个算法的代表是OC1，这里不多介绍。

2）如果样本发生一点点的改动，就会导致树结构的剧烈改变。这个可以通过集成学习里面的随机森林之类的方法解决。

# 7. 决策树算法小结

终于到了最后的总结阶段了，这里我们不再纠结于ID3, C4.5和 CART，我们来看看决策树算法作为一个大类别的分类回归算法的优缺点。这部分总结于scikit-learn的英文文档。

首先我们看看决策树算法的优点：

1）简单直观，生成的决策树很直观。

2）基本不需要预处理，不需要提前归一化，处理缺失值。

3）使用决策树预测的代价是O\(log2m\)。 m为样本数。

4）既可以处理离散值也可以处理连续值。很多算法只是专注于离散值或者连续值。

5）可以处理多维度输出的分类问题。

6）相比于神经网络之类的黑盒分类模型，决策树在逻辑上可以得到很好的解释

7）可以交叉验证的剪枝来选择模型，从而提高泛化能力。

8） 对于异常点的容错能力好，健壮性高。

我们再看看决策树算法的缺点:

1）决策树算法非常容易过拟合，导致泛化能力不强。可以通过设置节点最少样本数量和限制决策树深度来改进。

2）决策树会因为样本发生一点点的改动，就会导致树结构的剧烈改变。这个可以通过集成学习之类的方法解决。

3）寻找最优的决策树是一个NP难的问题，我们一般是通过启发式方法，容易陷入局部最优。可以通过集成学习之类的方法来改善。

4）有些比较复杂的关系，决策树很难学习，比如异或。这个就没有办法了，一般这种关系可以换神经网络分类方法来解决。

5）如果某些特征的样本比例过大，生成决策树容易偏向于这些特征。这个可以通过调节样本权重来改善。
