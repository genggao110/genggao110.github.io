---
layout:      post
title:       "KMeans Cluster"
subtitle:    " \"Hello Blog, Hello Cluster\""
date:        2019-01-16 15:38:00
author:      "Ming"
catalog: true
header-img:  "img/post-bg-2015.jpg"
tags:
    - cluster
---

> “Yeah It's on. ”

### KMeans聚类
---
#### 关于分类和聚类
> kmeans属于聚类算法中的一种。分类和聚类是不同的概念。虽然两者的目的都是对数据进行分类，但是却有一定的区别。
- 分类是按照某种标准给对象贴标签，再根据标签来区分归类。
- 聚类是事先没有给出标签，刚开始并不知道如何对数据分类，完全是算法自己来判断各条数据之间的相似性，相似的就放在一起。

在聚类的结论出来之间，不能知道每一类有什么特点，最后一定要根据聚类的结果通过人为的经验来分析才能知道聚成的这一类大概有什么特点。简而言之，聚类就是“物以类聚，人以群分”的原理。

#### 基本概念
K-means算法是典型的基于距离的聚类算法，采用距离作为相似性的评价指标，即认为两个对象的距离越近，其相似度就越大。该算法认为簇是由距离靠近的对象组成的，因此把得到紧凑且独立的簇作为最终目标。

简单来说，kmeans算法的原理就是：给定K的值，K代表要将数据分成的类别数，然后根据数据间的相似度将数据分成K个类，也称为K个簇。

度量数据相似度的方法一般是用数据点之间的距离来衡量，比如欧式距离、汉明距离、曼哈顿距离等。一般来说，我们使用欧式距离来度量数据间的相似性。而对于每一个簇，我们用簇中所有点的中心来描述，该中心也称为质心。我们通过对簇中的所有数据点取均值的方法来计算质心。
#### 算法描述
1. 创建K个点作为初始质心(通常是随机选择)
2. 当任意一个点的簇分类结果发生变化时：(1)对数据的每一个点，计算每一个质心与该数据点的距离，将数据点分配到距离其最近的簇。(2)对于每一个簇，计算簇中所有点的均值并将均值作为质心。

伪代码描述如下：

```
kmeans(dataset,k)

输入：数据集dataset,k的值
输出：包含质心的集合，簇分配结果
（1）选择k个数据点作为质心(通常是随机选取)
（2）当任意一个点的簇分类结果发生改变时

选择K个点作为初始质心  
repeat  
    将每个点指派到最近的质心，形成K个簇  
    重新计算每个簇的质心  
until 簇不发生变化或达到最大迭代次数 
```
#### 算法实现
```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

#----------------欧式距离函数---------------------
euclidean_Metric = np.linalg.norm

#------------------数据读入------------------------
def readData(filePath):
    Df = pd.read_csv(filePath)
    print(Df)
    DataSet = Df.values
    Rows_i,Dim_j = DataSet.shape
    return DataSet,Rows_i,Dim_j

#-----------------随机初始化聚类中心-----------------
def init_Center(DataSet,k,Rows_i,Dim_j):
    Center = np.zeros((k,Dim_j))
    for j in range(k):
        n_index = np.random.random_integers(0,Rows_i)
        Center[j, :] = DataSet[n_index, :]
    return Center,k

#----------------根据距离将数据分成不同的类-------------
#cluster_Data第一列是类别代码，第二列是距离聚类中心距离的平方
def cluster_Set(Center,k,Rows_i):
    cluster_Data = np.zeros((Rows_i, 2))
    for i in range(Rows_i):
        class_index = 0
        for j in range(k):
            temp1 = euclidean_Metric(DataSet[i,:]-Center[j,:])
            temp2 = euclidean_Metric(DataSet[i,:]-Center[class_index,:])
            if temp1 < temp2:
                class_index = j
        cluster_Data[i,0] = class_index
        cluster_Data[i,1] = temp2**2
    return cluster_Data

#--------------------把各类的质心作为新的聚类中心-------------------
def center_Update(cluster_data, Center, k):
    for j in range(k):
        get_Data = DataSet[np.nonzero(cluster_data[:,0]==j)[0]]
        Center[j,:] = np.mean(get_Data,axis= 0)
    return Center

#---------------------代价函数为各类到中心距离的平方和----------------
def cost_f(cluster_Data):
    cost = sum(cluster_Data[:,1])
    return cost

#---------------------作为4类的分类显示-----------------------------
def show(DataSet, cluster_Data):
    df = pd.DataFrame(DataSet, index=cluster_Data[:,0], columns=['x1','x2'])
    df1 = df[df.index==0]
    df2 = df[df.index==1]
    df3 = df[df.index==2]
    df4 = df[df.index==3]
    plt.figure(figsize=(10,8), dpi=80)
    axes = plt.subplot()
    type1 = axes.scatter(df1.loc[:,['x1']], df1.loc[:,['x2']],s=50, c='red',marker='d')
    type2 = axes.scatter(df2.loc[:,['x1']], df2.loc[:,['x2']],s=50, c='green',marker='*')
    type3 = axes.scatter(df3.loc[:,['x1']], df3.loc[:,['x2']],s=50, c='brown',marker='p')
    type4 = axes.scatter(df4.loc[:,['x1']], df4.loc[:,['x2']],s=50, c='black')
    type_center = axes.scatter(Center[:,0], Center[:,1], s=40, c='blue')
    plt.xlabel('x', fontsize=16)
    plt.ylabel('y', fontsize=16)
    axes.legend((type1, type2, type3, type4, type_center),('0','1','2','3','center'), loc=1)
    plt.show()

if __name__ == "__main__":
    DataSet,Rows_i,Dim_j = readData('E:/testdata.csv')
    Center,k = init_Center(DataSet,4,Rows_i,Dim_j)
    cost = 100000
    cost_temp = 1
    while cost_temp != cost:  # 代价函数不变时停止
        cost_temp = cost
        cluster_Data = cluster_Set(Center,k,Rows_i)  # 根据距离将数据点划分到不同的类别中
        show(DataSet, cluster_Data)           # 显示一次分类结果
        cost = cost_f(cluster_Data)           #计算代价函数
        print(cost)
        Center = center_Update(cluster_Data, Center, k) # 重新计算各类数据的中心作为聚类中心
    print('finish')

```

