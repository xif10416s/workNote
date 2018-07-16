# Recommend 推荐算法 
##  人口统计学的推荐（Demographic-based Recommendation） --
*   根据系统用户的基本信息发现用户的相关程度，这种被称为基于人口统计学的推荐（Demographic-based Recommendation）
*   因为不使用当前用户对物品的喜好历史数据，所以对于新用户来讲没有“冷启动（Cold Start）”的问题。
*   这个方法不依赖于物品本身的数据，所以这个方法在不同物品的领域都可以使用，它是领域独立的（domain-independent）。
*   缺点和问题
    *   基本信息对用户进行分类的方法过于粗糙，尤其是对品味要求较高的领域，比如图书，电影和音乐等领域
    *   用户信息不是很好获取

## 基于内容的推荐(Content Based Recommendation 简称CB) -- 参考
*   http://www.cnblogs.com/breezedeus/archive/2012/04/10/2440488.html
*   http://www.linkedkeeper.com/detail/blog.action?bid=1061
*   https://www.analyticsvidhya.com/blog/2015/08/beginners-guide-learn-content-based-recommender-systems/
*   根据推荐物品或内容的元数据，发现物品或者内容的相关性，这种被称为基于内容的推荐（Content-based Recommendation）
*   抽取item的特征向量 -> 计算余弦相似度 -> 推荐
*   物品有自己标题内容或者属性特征，根据内容计算物品相似度
*   每个用户的profile都是依据他本身对item的喜好获得的，自然就与他人的行为无关
*   新的item可以立刻得到推荐（New Item Problem）：只要一个新item加进item库，它就马上可以被推荐，被推荐的机会和老的item是一致的
*   CB的缺点
    -   item的特征抽取一般很难（Limited Content Analysis）
        +   需要对物品进行分析和建模，推荐的质量依赖于对物品模型的完整和全面程度。在现在的应用中我们可以观察到关键词和标签（Tag）被认为是描述物品元数据的一种简单有效的方法。
    -   无法挖掘出用户的潜在兴趣（Over-specialization）
        +   物品相似度的分析仅仅依赖于物品本身的特征，这里没有考虑人对物品的态度。
    -   无法为新用户产生推荐（New User Problem）
        +   因为需要基于用户以往的喜好历史做出推荐，所以对于新用户有“冷启动”的问题。
*   计算结构
    *   用户 =》 内容特征 《=  物品  
        *   内容=》词 =》 向量特征
        *   物品 =》内容
        *   用户 =》行为物品 =》内容  
*   协同过滤区别 ==》 相似度是基于书籍内容的，准确来说是标题，而不是根据使用数据

## 协同过滤（基于用户行为的推荐Collaborative Filtering Recommendations） -- 参考 
*   协同过滤领域主要的两种方式是最近邻（neighborhood）方法和潜在因子（latent factor）模型
*   最近邻（neighborhood）方法 --属于Memory-based类型
    -   用户推荐（UserCF）
    -   物品推荐（ItemCF)
*   潜在因子（latent factor）  --属于Model-based类型
    -   SVD奇异值分解矩阵
    -   ALS交替最小二乘

### 用户推荐（UserCF）--推荐那些和他有共同兴趣爱好的用户喜欢的物品
*   根据用户推荐重点是反应和用户兴趣相似的小群体的热点，是某个群体内的物品热门程度
*   UserCF需要维护一个用户相似度矩阵
*   User-based算法存在两个重大问题：
    -   1. 数据稀疏性。一个大型的电子商务推荐系统一般有非常多的物品，用户可能买的其中不到1%的物品，不同用户之间买的物品重叠性较低，导致算法无法找到一个用户的邻居，即偏好相似的用户。
    -   2. 算法扩展性。最近邻居算法的计算量随着用户和物品数量的增加而增加，不适合数据量大的情况使用。

### 物品推荐（ItemCF)--推荐那些和他之前喜欢的物品类似的物品
*   根据物品推荐着重与用户过去的历史兴趣，反应本人的兴趣爱好，更加个性化
*   ItemCF需要维护一个物品相似度矩阵

#### slope one Item-Based 的协同过滤推荐算法
*   https://blog.csdn.net/whaoxysh/article/details/19038453
*   人们并不总是能给出评分，当用户只提供二进制数据（购买与否）的时候，就无法应用Slope One 和其它基于评分的算法
*   该算法适用于物品更新不频繁，数量相对较稳定并且物品数目明显小于用户数的场景。依赖用户的用户行为日志和物品偏好的相关内容。
    *   优点：
        *   1.算法简单，易于实现，执行效率高；
        *   2.可以发现用户潜在的兴趣爱好；
    *   缺点：
        *   依赖用户行为，存在冷启动问题和稀疏性问题。

### 用户推荐（UserCF）vs 物品推荐（ItemCF)
|项目   |    UserCF|ItemCF|
|-|-|-|
|性能|适用于用户较少的场合，如果用户过多，</br>计算用户相似度矩阵的代价交大|适用于物品数明显小于用户数的场合，如果物品很多，</br>计算物品相似度矩阵的代价交大|
|领域|实效性要求高，用户个性化兴趣要求不高|长尾物品丰富，用户个性化需求强烈|
|实时性|用户有新行为，不一定需要推荐结果立即变化|用户有新行为，一定会导致推荐结果的实时变化|


### ALS交替最小二乘 -- 矩阵分解-潜在因子
*   算法理解：
    *   m*n的评分矩阵R，可以被近似分解成U*(V)T
    *   U为m*d的用户特征向量矩阵
    *   V为n*d的产品特征向量矩阵（(V)T代表V的转置）
    *   d为user/product的特征值的数量
    *   先固定 UU 求解 VV，再固定 VV 求解 UU ，如此迭代下去，问题就可以得到解决了。
*   spark als分布式实现
    -   

###  混合的推荐机制
*   加权的混合（Weighted Hybridization）: 用线性公式（linear formula）将几种不同的推荐按照一定权重组合起来，具体权重的值需要在测试数据集上反复实验，从而达到最好的推荐效果。
*   切换的混合（Switching Hybridization）：前面也讲到，其实对于不同的情况（数据量，系统运行状况，用户和物品的数目等），推荐策略可能有很大的不同，那么切换的混合方式，就是允许在不同的情况下，选择最为合适的推荐机制计算推荐。
*   分区的混合（Mixed Hybridization）：采用多种推荐机制，并将不同的推荐结果分不同的区显示给用户。其实，Amazon，当当网等很多电子商务网站都是采用这样的方式，用户可以得到很全面的推荐，也更容易找到他们想要的东西。
*   分层的混合（Meta-Level Hybridization）: 采用多种推荐机制，并将一个推荐机制的结果作为另一个的输入，从而综合各个推荐机制的优缺点，得到更加准确的推荐。

### 应用场景
*   新闻类网站采用UserCF
    -   用户大都喜欢热门新闻，特别细粒度的个性化可忽略不计
    -   个性化新闻推荐更强调热点，热门程度和实效性是推荐的重点，个性化重要性则可降低

#### UserCf  vs ItemCf
*   Item CF 的推荐有很好的新颖性，很擅长推荐长尾里的物品
*   User CF 总是倾向于推荐热门,在推荐长尾里项目方面的能力不足


### spark 相似度算法实现，http://www.lxway.com/642408066.htm


### 探索推荐引擎内部的秘密，第 3 部分: 深入推荐引擎相关算法 - 聚类
*   https://www.ibm.com/developerworks/cn/web/1103_zhaoct_recommstudy1/index.html

## 阅读
*   http://www.infoq.com/cn/articles/user-portrait-collaborative-filtering-for-recommend-systems

## 参考
*   https://www.ibm.com/developerworks/cn/web/1103_zhaoct_recommstudy1/index.html
*   http://www.lxway.com/642408066.htm
*   http://www.lxway.com/685091984.htm
*   http://www.lxway.com/4050452466.htm


