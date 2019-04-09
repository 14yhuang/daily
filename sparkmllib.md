RDD的api在mllib里，已不是主流

Dataframe的api在ml里，主学



# Pipelines

用于构造、评估和调优ML管道的工具



简言之，就是把所有工作搞成个流水线。



**DataFrame**:ML API使用这个来自Spark SQL的概念作为ML dataset，可以保存多种数据类型。比如：使用不同的列存储文本、特征向量、真实标签和预测结果。

**Transformer**:这是个是指一个算法将一个DataFrame transform成另一个DataFrame。也就是训练好的模型。比如：一个ML模型就是一个Transformer能够将一个特征数据的DataFrame转成预测结果的DataFrame。特征转换

**Estimator**:是指一个操作DataFrame产生Transformer的算法。比如：一个学习算法就是一个Estimator，可以在一个DataFrame上训练得到一个模型。评估器，算法。

**Pipeline**:一个Pipeline链是将多个Transformer和Estimator组合在一起组成一个ML workflow。

**Parameter**:所有的Transformer和Estimator共享一个公共的说明参数的API。



红框就是Estimator，两个蓝框是Transformer

![1553597260047](C:\Users\14yhuang\AppData\Roaming\Typora\typora-user-images\1553597260047.png)





# 统计学



## correlation关联系数

计算两个系列数据之间的相关性是一种常见的操作。



提供有Pearson相关系数（适合正太分布）和Spearman相关系数（看不出啥规律）



## Hypothesis假设检验



用来确定一个结果是否具有统计学意义



## Summarizer摘要器

我们通过摘要器为Dataframe提供向量列摘要统计信息。可用的指标是按列排列的最大值、最小值、均值、方差、非零数以及总数。



## 稠密向量

稠密向量：把向量所有值列出来

稀疏向量：只列出非0的值



# 特征提取



## TF-IDF

它反映了词在语料库中对文档的重要性。

弊端：不能识别近义词



用TT表示术语，用dd表示文档，用DD表示语料库。术语频率TF(t，d)是术语TT出现在文档dd中的次数，而文档频率df(t，D)是包含术语TT的文档数。

![1553603670043](C:\Users\14yhuang\AppData\Roaming\Typora\typora-user-images\1553603670043.png)



## Word2Vec

可以识别近义词

速度慢



## CountVectorizer

词数统计。

把字符数组转换为，每个字符对应个数的向量



## FeatureHasher

特征散列

是一种快速且空间利用率高的特征向量化方法，即将任意特征转换为向量或矩阵中的索引。它通过对特征应用散列函数并直接使用特征的散列值作为索引来工作



把许多特征转化为一个多维特征向量



# 特征转换



## Tokenizer

把文本转化为一个个的单词



## StopWordsRemover

停用词删除



## nn-gram

把几个词拼起来作为一个词特征



## Binarizer

二值化。

比如把图片色素转为二值。阈值



## PCA

降维



## PolynomialExpansion

造特征，升维



## Discrete Cosine Transform (DCT)





## StringIndexer

把某种特征，转换为索引数字。

比如3个特征，分别用0，1，2代表



## IndexToString

相反，要体现结果，转换回来



## OneHotEncoder

把某种特征，转化为数字向量



## Normalizer

正则化，规避某些特征数据很大，某些特征数据很小，导致两种特征影响力不一样。



## StandardScaler

均值方差的方式正则



## MinMaxScaler

最大最小值正则



## MaxAbsScaler

绝对值正则



## Bucketizer

数据分段，不同段的数据代表不同特征含义



## VectorAssembler

把每行特征值数据，全部放到一个向量里装着



## Imputer

填补缺失值

