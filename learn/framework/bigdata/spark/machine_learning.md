#Machine Learning
##预处理
*   PCA（Principal Component Analysis）
    -   用于提取数据的主要特征分量，常用于高维数据的降维
    -   去除线性相关的变量，保留线性不相关的变两
    -   应用
        -    模式识别，图像识别
        -    图像压缩
*    Discrete Cosine Transform (DCT) 离散余弦变换
    -   图像压缩
*    Normalizer
*    StandardScaler
    -    数据集的标准化：当个体特征太过或明显不遵从高斯正态分布时，标准化表现的效果较差
    -    回归模型中的目标值处理
*    MinMaxScaler
    -   将训练集缩放至[0,1]
*    MaxAbsScaler
    -   将训练集缩放至[-1,1]
*    StringIndexer
    -    给label 创建索引，按label出现的频率排序
*    VectorIndexer
    -    给Vector中的特征索引分类特征列
    -    从给定的dataset中自动区分分类特征和连续特征，分类数由maxCategories决定，值的个数


##特征选择
*    VectorSlicer
    -    特征截取
*    ChiSqSelector
    -   自动选择合适的特征


##算法
*    分类回归
    -    分类
        -    输出值是离散值
        -    二元分类
            -    Logistic regression
                -    逻辑回归
                -    是一个非线性模型，sigmoid函数
                -    sigmoid可以轻松处理0/1分类问题。
                    -    垃圾邮件
                    -    欺诈
                -    参数
                    -    MaxIter
                        -    最大迭代次数默认100
                    -    RegParam
                        -    正则化参数，默认0.0
                        -    为防止过度拟合的模型出现（过于复杂的模型），在损失函数里增加一个每个特征的惩罚因子。这个就是正则化
                    -    ElasticNetParam
                        -    ElasticNet 是一种使用L1和L2先验作为正则化矩阵的线性回归模型
                        -    ElasticNet是Lasso和Ridge回归技术的混合体。它使用L1来训练并且L2优先作为正则化矩阵。当有多个相关的特征时，ElasticNet是很有用的。
                        -   默认0，使用L2正则化
            -    Decision tree
                -    Random Forests
                -    Gradient-Boosted Trees (GBTs)
            -    Naive Bayes
        -    多元分类
            -    Multilayer perceptron
                -    多分类
                -    多层感知机
            -    One-vs.-rest
                -    为每一个类建立一个唯一的分类器，属于此类的所有样例均为正例，其余的全部为负例
    -    回归
        -    输出连续值
        -    线性回归
        -    linear regression
        -    Generalized linear regression
        -    Decision tree regression
        -    Random forest regression
        -    Gradient-boosted tree regression
    -    正则化
        -    L1正则化
            -    表示各个参数绝对值之和
            -    Lasso
        -    L2正则化
            -    各个参数的平方的和的开方值
            -    Ridge
        -    http://blog.csdn.net/vividonly/article/details/50723852
        -    L1会趋向于产生少量的特征，而其他的特征都是0，而L2会选择更多的特征，这些特征都会接近于0。Lasso在特征选择时候非常有用，而Ridge就只是一种规则化而已。在所有特征中只有少数特征起重要作用的情况下，选择Lasso比较合适，因为它能自动选择特征。而如果所有特征中，大部分特征都能起作用，而且起的作用很平均，那么使用Ridge也许更合适。
*    Collaborative Filtering
    -     alternating least squares (ALS) algorithm
*    聚类
    -    非监督学习
    -    K-means
        -    k-means 的结果是每个数据点被 assign 到其中某一个 cluster 
    -    Gaussian mixture
        -     GMM 则给出这些数据点被 assign 到每个 cluster 的概率
        -    可以把这个概率转换为一个 score
    -    Power iteration clustering (PIC)
        -    幂迭代聚类
    -    Latent Dirichlet allocation (LDA)
        -    主题模型
        -    主要用在数据挖掘（dm）中的text mining和自然语言处理中，主要是用来降低维度的
        -    语义挖掘
        -    running for more iterations dramatically improves the results
        -     identify stop words 
    -    Bisecting k-means
        -    不适用于非球形簇的聚类，而且不同尺寸和密度的类型的簇，
    -    Streaming k-means
*    Dimensionality Reduction-降维
    -    隐藏特征提取
    -    Singular value decomposition --SVD
    -    Principal component analysis (PCA)
        -    特征抽取是指将高纬度的特征经过某个函数映射至低纬度作为新的特征




##检验
*    roc
    -    受试者工作特征曲线 （receiver operating characteristic curve，简称ROC曲线），又称为感受性曲线（sensitivity curve）。
*    auc
    -   roc曲线以下 的面积
*    base
    -    False Negative（假负 , FN）实际正预测负
    -   True Positive （真正, TP）实际正预测正
    -   True Negative（真负 , TN）实际负预测负 
    -   False Positive （假正, FP）实际负预测正
    -   精确率--实际正预测正(预测对的)/预测为正的总量--在预测值中的准确率
    -   p = TP/(TP+FP)      
    -   召回率--查全率--实际正预测正/实际为正的总量--在实际为正的准确率
    -   R=TP/(TP+FN)
    -   FPR=FP/(FP+TN)--预测为正实际是负（预测错误）/ 实际所有负的 比例，误诊率
    -   TPR=TP/ (TP+ FN)--预测正实际正/所有正的比例，准确率
    -   F-measure－－－F-Measure是Precision和Recall加权调和平均：
    -   正确率 * 召回率 * 2 / (正确率 + 召回率) （F 值即为正确率和召回率的调和平均值）
*    RMSE  --均方根误差亦称标准误差


##optimization problems--凸优化
*    Gradient descent
    -    梯度下降（GD）是最小化风险函数、损失函数的一种常用方法
    -    每迭代一步，都要用到训练集所有的数据，如果m很大，速度慢
*    Stochastic gradient descent (SGD)
    -     随机梯度下降
*    Limited-memory BFGS (L-BFGS)

##spark-sklearn/
*    https://pypi.python.org/pypi/spark-sklearn/0.2.0
*    http://blog.csdn.net/sunbow0/article/details/50848719
