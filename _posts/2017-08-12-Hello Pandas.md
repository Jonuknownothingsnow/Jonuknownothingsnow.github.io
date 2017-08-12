---
title: Hello Pandas
excerpt_separator: <!--more-->
categories: 
- 工具经验
date: 2017-08-12
---

平时工作一直在用pandas，感觉也该写一下相关的心得，来对这么好用的一个工具表示感谢（笑）。本文只是随手记下的一些知识点，如果要系统的学习的话还是~~不厌其烦地~~向大家推荐《利用python进行数据分析》

<!--more-->

## dataframe

虽然说Series更加基础，但其实我们日常使用最多的还是dataframe。说简单点的话dataframe其实就是python版的excel，用来操作数据，有的同学可能喜欢使用sql语句对数据进行切片、聚合之类的处理，但是就我个人而言，dataframe操作起来要比sql人性化多了。



### 切片

dataframe里最基础的操作就是切片，相当于excel的选中行，选中列。在dataframe上切片的操作方法有很多，不过一般来说用的就那么几个。

```python
import pandas as pd
li = [[1,2,3],[4,5,6]]
df = pd.DataFrame(li,columns=["a","b","c"])


# 选中列,单个列名返回一个Series，列表形式选中返回DataFrame，即使是列表只有一个元素
df["a"]

df[["a","b"]]

df[["a"]]

# 选中行
df.iloc[0]

df.loc[0,:]
```

提醒一下，dataframe切片返回的是原数据的一个视图，对返回的数据进行操作会直接反应在原始数据上，之所以这样设计是因为dataframe常常应用于较大数据的操作，假如每次切片都复制的话对内存的消耗太大。

如果需要获得原始数据的一个副本，使用.copy()方法即可。

除了直接使用index和列名来进行选中以外，还有根据条件筛选来选取数据的方法。

```python
# 筛选数据，选取"a"属性大于3的列
df[df["a"]>3]
```

这是最常见的筛选方式，实际上是df["a"]>3返回了一个布尔值列表，df[bool_list]返回了所有对应True值的行，了解这点后可以玩出许多花样

```python
df[df["a"].isnull()==False]
df[(df["a"]>3)&(df["b"])<3]
```

除了传入布尔值列表以外，还可以用类似sql的语法进行筛选，不过这种我用的不多

```python
df.query("a==1")
```



### 数据处理

在dataframe中最简单的数据处理可以这样来进行：

```python
#对a列每个元素加1
df["a"] = df["a"]+1
df["x"] = df["a"]*df["b"]
```



最常用的排序语法是这样子的，注意dataframe默认使用的是快速排序法，这并不是一个stable的算法。可以通过设置kind="mergesort"来得到stable的排序，但这样操作的话仅支持对单列进行排序

```python
#这是我们常用的排序
df.sort_values("a",ascending=False)#按a列降序排序
#与其他操作不同的是，df.sort_values有个inplace参数，可以直接在原始数据上操作
df.sort_values(["a","b"],inplace=True)
```



dataframe里最常用最通用的是.apply()方法

```python
#对a列逐个加1
df["a"].apply(lambda x:x+1)

#如果处理太复杂，可以定义一个方法
def foo(x):
  x += 1
  return x

df["a"].apply(foo)

#注意，dataframe处理后只是返回一个结果，不会直接作用在原始数据上，如果需要修改原始数据，需要再进行赋值
df["a"] = df["a"].apply(lambda x:x+1)

#直接在视图上赋值会被pandas警告，有的时候是无法这样赋值，建议赋值操作的时候使用loc方法
df.loc[:,"a"] = df["a"].apply(lambda x:x+1)

#对dataframe直接进行apply会按列传入到方法，如果需要逐行操作需要设置axis=1
```



apply是最自由的方法，使用起来非常爽快，除此之外dataframe还有许多快捷的方法，注意一下和apply一样，dataframe所有的操作都是返回一个结果，而非直接在原始数据上修改。

```python
#数据拼接
df1 = df1.append(df2)

#数据联结，类似于sql里面的JOIN。how为联结方式，on为以某一列为基准进行联结，也可以是多列，也可以用left_on,right_on以两个dataframe里不同名的列进行合并。
df1 = pd.merge(df1, df2, how="left", on="a")

#等效于
df1 = df1.merge(df2, how="left", on="a")

#去重
df = df.drop_duplicates("a")#除去a列值相同的行，保留第一行
df = df.drop_duplicates(["a","b"],keep="last")#除去a、b列值相同的行，保留最后一行
df = df.drop_duplicates(keep=False)#除去完全相同的行，不保留

#去空
df = df.dropna()

#填充
df = df.fillna()

df.ffill()
df.fillna(method="ffill")#等效于上一行

df.fillna(value="mean")#以均值进行填
```



一般来说apply和以上这些方法可以满足大部分的数据操作需求，理论上来说循环能做的事情apply也能做，但是实际操作当中可能还是存在一些需要迭代才能完成的需求，dataframe的迭代与想象中其实不太一样

```python
#想象中这样会逐行打印
for x in df:
  print(x)
#理想很丰满，现实很骨感，这样做其实等效于：
for x in df.columns:
  print(x)

#要想打印出每一列的值，需要
for x in df.iterrows():
  print(x[1])

#可能有同学会问，为什么不直接print(x)呢，因为x其实是个列表，x[0]是该行的index，x[1]才是这以行的值(一个Series)

#当然打印这个操作其实也可以由apply完成的，尽量还是多使用apply吧
df.apply(lambda row:print(row),axis=1)
```



此外大部分的dataframe操作都通过axis参数来指定在列或者行上进行操作，大家自己试试看吧。



### 数据分析

dataframe可以进行快速多样的数据分析，首先是基础的数据特征：

```python
#平均值
df.mean()
df["a"].mean()

#标准差
df.std()

#计数
df.count()

#最大值最小值
df.max()
df.min()

#相关系数矩阵！！
df.corr()
df.cov()#协方差矩阵

#一键数据描述大礼包！！
df.describe()#包括最大值最小值平均值标准差四分位数，让你一次看到饱玩到爽！

```

更进一步的分析往往要使用到数据聚合，举例来说我们手上有一份数据，每一行其实是代表了一个订单的信息，但是我们想看的其实是每个用户的信息，这时候就要对数据进行聚合

```python
#假设用户的唯一主代码是user_id，订单金额是amount
group = df.groupby("uesr_id")
#计算每个用户的总消费金额
user = group["amount"].sum()#会返回一个以user_id为index，名称为amount的Series
user = group[["amount"]].sum()#会返回一个以user_id为index，只有一列amount列的DataFrame

#对新的DataFrame追加新的列
user["count"] = group["order_id"].count()

#group对象也可以迭代
for x in group:
  print(x[0])#返回每个用户的user_id
  print(x[1])#每个用户全部订单的DataFrame，即df[df["user_id"]==x[0]]
```

可以做到类似的分析的还有层次索引，数据透视等方法，不过我觉得还是使用聚合操作来的直接一些。



### 数据读取

在操作数据上非常方便外，dataframe读取各种数据源也是非常便捷，前面说到dataframe与excel差不多，其实dataframe读取excel也非常方便。

```python

# excel_to_pandas
file = pd.read_excel("xxx.xlsx")

#pandas_to_excel
file.to_excel("xxxx.xlsx")
```

这里需要注意一下，pandas本身是可以读取.xlsx的，但是无法写出.xlsx，要写出.xlsx文件需要安装openpyxl,当然pip装一下很快的，如果经常涉及到和excel的交互的话还是装起来比较好。

除了excel以外"pd.read_xxx"/"to_xxx"系列的方法还有很多,用的比较多的还有csv格式、html格式等，甚至有read_sql，在要求不高的时候可以临时用来读取下数据库。可以自行尝试一下

这两个方法的有几个常用的参数

```python
file = pd.read_excel("xxx.xlsx",header=None,converters={0:str})#无标题行，读取第0列为字符串格式
file.to_excel("xxx.xlsx",index=False)#不输出序号列
```

在和excel交互的时候需要非常小心excel的智障特性，比如把id识别为数字，然后就采用科学计数法来表示，converters参数可以有效防止这个这个智障特性作怪



虽然说pandas有read_sql的方法，不过我想很少人会使用这个方法来直接获取数据库的数据，大部分时候还是用先获取到数据之后，再转化为dataframe。由于我最近习惯直接写SQL语句，比较少用ORM,这里写下对query结果的转化方法

```python
#以pymysql的cursor为例
cur.execute("balabalabala")
raw_data = cur.fetchall()

#raw_data的格式类似于((1,2,3),(4,5,6))
df = pd.Series(raw_data)
df = df.apply(pd.Series)

#这是我自己搞出来的野路子转化法，可能有更官方，更有效的转化方法，我还没研究出来
```









