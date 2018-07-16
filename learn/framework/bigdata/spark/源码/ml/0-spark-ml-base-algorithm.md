# spark ml 基本算法
## Collaborative Filtering -- https://blog.csdn.net/u011239443/article/details/51752904
### 推荐系统中用户和物品的交互数据分为显性反馈和隐性反馈数据
####  隐性反馈  implicitPrefs=True
*   输入参数：用户id,物品id,置信程度（用户对这个物品行为的强弱，如点击次数，观看时间长度等，非用户主观给出的数字）
*   隐式模型多了一个置信参数，alpha
    -   表示用户偏爱某个商品的置信程度
    -    alpha: Double = 1.0
    -    val c1 = alpha * math.abs(rating) // instead so that it is never negative. c1 is confidence - 1.0.
*   没有负反馈 
    -   在隐反馈模型中是没有评分的，仅仅表示用户和物品之间有没有交互
    -   用户和物品之间有交互就等于1，没有就等于0
*   隐式反馈是内在的噪音
    -   我们可能知道一个人的购买行为，但是这并不能完全说明偏好和动机，因为这个商品可能作为礼物被购买而用户并不喜欢它。
*   显示反馈的数值值表示偏好（preference），隐式回馈的数值值表示信任（confidence）
*   不過如果你是用 Spark ML 的 ALS(implicitPrefs=True) 的話，並不需要手動加入負樣本。對 implicit feedback 的 ALS 來說，手動加入負樣本（Rui = 0 的樣本）是沒有意義的，因為 missing value / non-observed value 對該演算法來說本來就是 0，表示用戶確實沒有對該物品做出行為，也就是 Pui = 0 沒有偏好，所以 Cui = 1 + alpha x 0 置信度也會比其他正樣本低。不過因為 Spark ML 的 ALS 只會計算 Rui > 0 的項目，所以即便你手動加入了 Rui = 0 或 Rui = -1 的負樣本，對整個模型其實沒有影響。

#### 显性反馈
*   输入参数：用户id,物品id,用户的打分（用户主管给出的评分）
*   