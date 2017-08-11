---
title:Hello Pandas
describe:平时工作一直在用pandas，感觉也该写一下相关的心得，来对这么好用的一个工具表示感谢（笑）
categories:- 工具经验
date: 2017-08-12
---

平时工作一直在用pandas，感觉也该写一下相关的心得，来对这么好用的一个工具表示感谢（笑）。本文只是随手记下的一些知识点，如果要系统的学习的话还是~~不厌其烦地~~向大家推荐《利用python进行数据分析》

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

dataframe里最常用最通用的是.apply()方法

```python
#对a列逐个加1
da["a"].apply(lambda x:x+1)

#如果处理太复杂，可以定义一个方法
def foo(x):
  x += 1
  return x

da["a"].apply(foo)

#注意，dataframe处理后只是返回一个结果，不会直接作用在原始数据上，如果需要修改原始数据，需要再进行赋值
da["a"] = da["a"].apply(lambda x:x+1)

#直接在视图上赋值会被pandas警告，有的时候是无法这样赋值，建议赋值操作的时候使用loc方法
da.loc[:,"a"] = da["a"].apply(lambda x:x+1)

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

大部分的dataframe操作都通过axis参数来指定在列或者行上进行操作，大家自己试试看吧。



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









