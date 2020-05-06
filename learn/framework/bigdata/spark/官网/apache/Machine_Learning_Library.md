#		Machine Learning Library 
##	Local vector--局部向量

*   一个局部向量由一个从0开始的整数类型索引和一个double类型的值组成，被存储在一个单独的机器上。MLlib支持两种类型的局部向量：密集型和稀疏行。一个密集型依靠一个double型数组来代表他的entry值，而一个稀疏型向量依靠两个并行数组：索引数组和值数组。举个例子，一个向量(1.0,0.0,3.0)可以被表示为密集型格式：[1.0, 0.0, 3.0] 或者被表示为稀疏型格式：(3, [0,2], [1.0, 3.0])，元组的第一个值3是向量的数量，第二个是索引的数组，第三个是值的数组。
*   Labeled point --带double值作为label的局部向量
*   labeled point 是一个局部向量，要么是密集型的要么是稀疏型的，用一个label/response进行关联。在MLlib里，labeled points被用来监督学习算法。我们使用一个double数来存储一个label，因此我们能够使用labeled points进行回归和分类。在二进制分类里，一个label可以是 0（负数）或者 1（正数）。在多级分类中，labels可以是class的索引，从0开始：0,1,2,......

##   分类

*   有监督学习 《== 输入有标签  -- （分类，回归）
*   无监督学习   《-- 输入无标签 -- 聚类

##   local matrix--矩阵

*   局部矩阵由整数型行和列的索引和浮点数类型的值组成，存储在一个单独节点上。MLlib支持密集矩阵，entry值被存储在一个一维浮点数数组，以列为排序
主键。而稀疏矩阵，non-zero entry值，以Compressed Sparse Column (CSC) 格式存储，以列主键排序。例如，下面的密集矩阵
*   distributed matrix--分布式矩阵
*   拥有long类型的行和列的索引已经double值，存储在分布式一个或多个RDD中，选择合适的格式存储分布式矩阵，分布式矩阵的转换需要全局的shuffle，开销很大，三种分布式矩阵实现将被引入
*   基础的类型RowMatrix，基于行的分布式矩阵，没有行的索引，一组特征向量的集合 。依托于RDD的行，每一行是一个局部向量。假设列的数量不是非常巨大，一个局部向量可以用来代表与driver的通信，并且可以被一个节点存储和操作，IndexedRowMatrix是带行的分布式行矩阵，

##   QR分解

*   QR分解法是三种将矩阵分解的方式之一。这种方式，把矩阵分解成一个正交矩阵与一个上三角矩阵的积。QR
分解经常用来解线性最小二乘法问题。QR
分解也是特定特征值算法即QR算法的基础。
*   Singular Value Decomposition 奇异值分解 《--主成分分析

##   correlation -- 相关性

*   Pearson相关系数定义为这两个变量的协方差与二者标准差积的商用来度量正态分布的定居变量间的线性相关关系。
*   Spearman秩相关系数通常被认为是排列后的变量之间的Pearson线性相关系数，是非参数测度，在实际计算中，根据数据的秩而不是根据实际值计算的，即先对原始变量的数据排秩，再根据公式计算

##   假设检验（Hypothesis Testing）

*   什么是假设检验

    假设检验是用来判断样本与样本，样本与总体的差异是由抽样误差引起还是本质差别造成的统计推断方法。其基本原理是先对总体的特征作出某种假设，然后通过抽样研究的统计推理，对此假设应该被拒绝还是接受作出推断。

*   显著性检验（Significance Testing）

    显著性检验就是事先对总体（随机变量）的参数或总体分布形式做出一个假设，然后利用样本信息来判断这个假设（原假设）是否合理，即判断总体的真实情况与原假设是否显著地有差异。或者说，显著性检验要判断样本与我们对总体所做的假设之间的差异是纯属机会变异，还是由我们所做的假设与总体真实情况之间不一致所引起的。

*   显著性检验是针对我们对总体所做的假设做检验，其原理就是“小概率事件实际不可能性原理”来接受或否定假设。
*   抽样实验会产生抽样误差，对实验资料进行比较分析时，不能仅凭两个结果（平均数或率）的不同就作出结论，而是要进行统计学分析，鉴别出两者差异是抽样误差引起的，还是由特定的实验处理引起的。

##   数学公式
###    损失函数(loss function)
通常而言，损失函数由损失项(loss term)和正则项(regularization term)组成。
http://www.ics.uci.edu/~dramanan/teaching/ics273a_winter08/lectures/lecture14.pdf

一、损失项

    对回归问题，常用的有：平方损失(for linear regression)，绝对值损失；
    
    对分类问题，常用的有：hinge loss(for soft margin SVM)，log loss(for logistic regression)。


    说明：
    
    对hinge loss，又可以细分出hinge loss（或简称L1 loss）和squared hinge loss（或简称L2 loss）。国立台湾大学的Chih-Jen Lin老师发布的Liblinear就实现了这2种hinge loss。L1 loss和L2 loss与下面的regularization是不同的，注意区分开。

二、正则项

    常用的有L1-regularization和L2-regularization。上面列的那个资料对此还有详细的总结。
    
    防止过度拟合

  http://52opencourse.com/133/coursera%E5%85%AC%E5%B

  过拟合问题往往源自过多的特征，

##    SGD(Stochastic Gradient Descent-随机梯度下降)

###   classification -- 分类

*   linear Support Vector Machines (SVMs)     
*   logistic regression  --> 支持multiclass classification

###     regression -- 回归

*   least squares 最小二乘法

###    分类回归区别：

*   定量输出称为回归，或者说是连续变量预测；
*   定性输出称为分类，或者说是离散变量预测。

###    举个例子：

*   预测明天的气温是多少度，这是一个回归任务；
*   预测明天是阴、晴还是雨，就是一个分类任务。

##    Decision trees--决策树
广泛使用，容易解释，能处理多分类，Random Forests和GBTs属于ensemble learning algorithms（集成学习算法）
####     算法：
​    熵（entropy） -- 分类
​    Gini Impurity （基尼不纯度）方法 --分类
​    Variance 方差--回归

###    MLlib中的Random Forests和Boosting--集成学习算法

*   简言之，集成学习算法（Ensemble Learning Algorithms）是对已有的机器学习算法进行组合。组合后的模型将比原有的任意一个子模型更加的强大和精确。
*   在MLlib 1.2中，我们使用 Decision Trees（决策树）作为基础模型，同时还提供了两个集成方法：    Random Forests与        Gradient-Boosted Trees（GBTs）。两个算法的主要区别在于各个部件树（component tree）的训练顺序。
*   在Random Forests中，各个部件树会使用数据的随机样本进行独立地训练。对比只使用单棵决策树，这种随机性可以帮助训练出一个更健壮的模型，同时也能避免造成在训练数据上的过拟合。
*   GBTs一次训练一棵树，每次加入的新树用于纠正已训练的模型误差。因此，随着越来越多树被添加，模型变得越来越有表现力。

![](http://img.ptcms.csdn.net/article/201503/11/54ffef92ebf14.jpg)

*   Random Forests：鉴于Random Forests中每棵树都独立地进行训练，因此多个树的训练可以并行进行（同时，单个树上的训练也可以并行地执行）。MLlib就是这样做的：可变数量的子树并行地进行训练，而具体的数量则在内存限制的基础上进行迭代优化。树越多过度拟合越小
*   GBTs：鉴于GBTs一次只能训练一棵树，只能实现单棵树级别的并行化。树越多过度拟合越多
*   Random Forests训练的速度无疑更快，但是如果想达到同样的误差，它们往往需要深度更大的树。GBTs每次迭代都可以进一步减少误差，但是如果迭代次数太多，它很可能造成过拟合。
*   在大型训练数据集上的效果。使用更多的数据时，两个方法的训练时间都有所增长，但是显然也都得到了一个更好的结果。

##   Random Forests

*   独立计算每棵树，可以并行处理
*   每次迭代从元数据集随机抽取数据
*   为每个树节点随机抽取不同的特征

参数：

    numTrees 
    
        增加树可以减少预测方差，提高模型的准确度，训练时间随树的量线性增加
    
     maxDepth
    
    增加深度，可以增加表达力，可能过度拟合
    
     subsamplingRate
    
    指定每棵树的数据比例，默认是1，减少可以提高速度
    
    featureSubsetStragy
    
    每个节点树的候选特征量

梯度提升决策树Gradient Boosted Tree (GBT)

    按顺序迭代决策树，减少损失函数，不断调整
    
    不支持多分类

##   Naive Bayes - spark.mllib--贝叶斯分类器

    贝叶斯分类是一类分类算法的总称，这类算法均以贝叶斯定理为基础，故统称为贝叶斯分类
    
    多分类

##    Collaborative Filtering - spark.mllib

    协同过滤--推荐系统
    
    填补没有的用户物品的关系矩阵
    
    协同过滤分成了两个流派，一个是Memory-Based，一个是Model-Based 。 spark支持Model-Based
    
    spark使用alternating least squares 算法学习潜在因子latent factors --交替最小二乘法(alternative least sqares,简称ALS).  
    
    参数：
    
    numblocks--并行计算的数量
    
    rank--潜在因子的数量
    
    iterations--迭代次数
    
    lambda--正则参数
    
    implicitPrefs--决定了是用显性反馈ALS的版本还是用适用隐性反馈数据集的版本。    
    
    alpha--是一个针对于隐性反馈 ALS 版本的参数，这个参数决定了偏好行为强度的基准。    

显式用户反馈(explicit user feedback)指用户给出的显式倾向,如电影评分、商品打分等;

隐式用户反馈(implicit user feedback),如转发微博、浏览网站或购买商品等.收集隐式反馈数据不需要用户额外的努力,同时不易引起用户反感,因此它具有以下特点:收集成本更低、应用场景更广、数据规模更大.  


可以调整这些参数，不断优化结果，使均方差变小。比如：iterations越多，lambda较小，均方差会较小，推荐结果较优。



##   Clustering - spark.mllib无监督学习

###    K-means

    最常用的聚类算法之一
    
    基本实现：1.随机选k个聚类中心 2.算其他点到这k个点的距离  3.重新计算k个簇各自的中心，计算方法是取簇中所有元素各自维度的算术平均数。4重复，直到聚类结果不再变化
    
    参数--------------------
    
    k--期望的分类数
    
    maxIterations--最大的迭代数
    
    initializationMode--这个参数决定了是用随机初始化还是通过 k-means|| 进行初始化。    
    
    runs
         是跑 k-means 算法的次数（k-mean 算法不能保证能找出最优解，如果在给定的数据集上运行多次，算法将会返回最佳的结果）。    
    
    initializiationSteps
         决定了 k-means|| 算法的步数。    
    
    epsilon
         决定了判断 k-means 是否收敛的距离阀值。    

###    Gaussian mixture

###    Power iteration clustering (PIC)--快速迭代法

    计算data set之间的相似度矩阵，迭代规则化矩阵，利用kmeans处理矩阵进行分类
    
    该算法利于分布计算

###   Latent Dirichlet allocation (LDA)

    是一种文档主题生成模型，也称为一个三层贝叶斯概率模型，包含词、主题和文档三层结构
    
    根据一组文档推测出主题
    
    返回的数据格式为：documents: RDD[(Long, Vector)]，其中：Long为文章ID，Vector为文章分词后的词向量；用户可以读取指定目录下的数据，通过分词以及数据格式的转换，转换成RDD[(Long, Vector)]即可。
    
    k: 主题数，或者聚类中心数
    DocConcentration：文章分布的超参数(Dirichlet分布的参数)，必需>1.0
    TopicConcentration：主题分布的超参数(Dirichlet分布的参数)，必需>1.0
    MaxIterations：迭代次数
    setSeed：随机种子
    CheckpointInterval：迭代计算时检查点的间隔
    Optimizer：优化计算方法，目前支持"em", "online"

###   Bisecting k-means --二分K-means聚类

    比普通K-means块
    
    二分K均值算法可以加速K-means算法的执行速度，因为它的相似度计算少了
    
    不受初始化问题的影响，因为这里不存在随机点的选取，且每一步都保证了误差最小

###   Streaming k-means

##   Dimensionality Reduction - spark.mllib--降维

    隐藏特征提取
    
    在保持结构的前提下压缩数据

###   Singular value decomposition (SVD)--奇异值分解

    降维
    
    特征提取
    
    推荐

###   Principal component analysis (PCA)

    PCA把原先的n个特征用数目更少的m个特征取代，新特征是旧特征的线性组合

##    Feature Extraction and Transformation-特征提取和转换
###   TF-IDF--

    TFIDF的主要思想是：如果某个词或短语在一篇文章中出现的频率TF高，并且在其他文章中很少出现，则认为此词或者短语具有很好的类别区分能力，适合用
    来分类。TFIDF实际上是：TF * IDF，TF词频(Term Frequency)，IDF逆向文件频率(Inverse Document
    Frequency)。
    
    TF-IDF是一种特征向量化方法，这种方法多用于文本挖掘，通过算法可以反应出词在语料库中某个文档中的重要性。文档中词记为t，文档记为d , 语料库记为D . 词频TF(t,d) 是词t 在文档d 中出现的次数。
    
    Spark.mllib 中实现词频率统计使用特征hash的方式，原始的特征通过hash函数，映射到一个索引值。

###   Word2Vec

    词表征为实数值向量

##   Frequent Pattern Mining

*   用于挖掘经常一起出现的item集合（称为频繁项集），通过挖掘出这些频繁项集，当在一个事务中出现频繁项集的其中一个item，则可以把该频繁项集的其他item作为推荐。比如经典的购物篮分析中啤酒、尿布故事，啤酒和尿布经常在用户的购物篮中一起出现，通过挖掘出啤酒、尿布这个啤酒项集，则当一个用户买了啤酒的时候可以为他推荐尿布，这样用户购买的可能性会比较大，从而达到组合营销的目的。

*   常见的频繁项集挖掘算法有两类，一类是Apriori算法，另一类是FPGrowth。Apriori通过不断的构造候选集、筛选候选集挖掘出频繁项集，
需要多次扫描原始数据，当原始数据较大时，磁盘I/O次数太多，效率比较低下。FPGrowth算法则只需扫描原始数据两遍，通过FP-tree数据结构
对原始数据进行压缩，效率较高。
FP-growth--找出频繁出现的item的组合

*   FPGrowth
算法主要分为两个步骤：FP-tree构建、递归挖掘FP-tree。FP-tree构建通过两次数据扫描，将原始数据中的事务压缩到一个FP-tree
树，该FP-tree类似于前缀树，相同前缀的路径可以共用，从而达到压缩数据的目的。接着通过FP-tree找出每个item的条件模式基、条件FP-
tree，递归的挖掘条件FP-tree得到所有的频繁项集。算法的主要计算瓶颈在FP-tree的递归挖掘上交易号，物品号

###   Association Rules

关联规则挖掘过程主要包含两个阶段：

      第一阶段必须先从资料集合中找出所有的高频项目组(Frequent Itemsets)，
    
      第二阶段再由这些高频项目组中产生关联规则(Association Rules)。

##   结果统计－－

###   BinaryClassification

    Precision－－准确率
    
    是指检出的相关文献数占检出文献总数的百分比。查准率反映检索准确性，其补数就是误检率。
    
    Recall －－查全率
    
    是指检出的相关文献数占系统中相关文献总数的百分比。查全率反映检索全面性，其补数就是漏检率。
    
    前者是衡量检索系统和检索者检出相关信息的能力，后者是衡量检索系统和检索者拒绝非相关信息的能力。两者合起来，即表示检索效率。
    
    以检索为例，可以把搜索情况用下图表示：

 / |相关|不相关
--------|------|----
检索到   | A    | B
未检索到 |  C   |  D     


    A：检索到的，相关的                （搜到的也想要的）
    B：检索到的，但是不相关的          （搜到的但没用的）
    C：未检索到的，但却是相关的        （没搜到，然而实际上想要的）
    D：未检索到的，也不相关的          （没搜到也没用的）
    
    如果我们希望：被检索到的内容越多越好，这是追求“查全率”，即A/(A+C)，越大越好。
    
    如果我们希望：检索到的文档中，真正想要的、也就是相关的越多越好，不相关的越少越好，
    
    这是追求“准确率”，即A/(A+B)，越大越好。
    
    F-measure－－－F-Measure是Precision和Recall加权调和平均：
    
    正确率 * 召回率 * 2 / (正确率 + 召回率) （F 值即为正确率和召回率的调和平均值）
    
    areaUnderPR
    
    areaUnderROC
    
      面积越大，表示分类性能越好

##    Evaluation Metrics--评测模型的指标
###    Classification model evaluation
###   Binary classification

    分类模型给每个分类输出的score越高，表示越明显
    
        False Negative（假负 , FN）实际正预测负；
    
        True Positive （真正, TP）实际正预测正；
    
        True Negative（真负 , TN）实际负预测负 ；
    
        False Positive （假正, FP）实际负预测正；

精确率--实际正预测正(预测对的)/预测为正的总量--在预测值中的准确率

p = TP/(TP+FP)      

召回率--查全率--实际正预测正/实际为正的总量--在实际为正的准确率

R=TP/(TP+FN)
###Ranking systems --排名系统

    Precision at k
         -- 平均所有用户中，前k个推荐文档中有多少个在相关文档中
    
    Mean Average Precision --有大号个推荐文档在正相关文档中
    
    Normalized Discounted Cumulative Gain - -平均所有用户中，前k个推荐文档中有多少个在相关文档中



