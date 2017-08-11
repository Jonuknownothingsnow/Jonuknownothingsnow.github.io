


平时工作一直在用pandas，感觉也该写一下相关的心得，来对这么好用的一个工具表示感谢（笑）。本文只是随手记下的一些知识点，如果要系统的学习的话还是~~不厌其烦地~~向大家推荐《利用python进行数据分析》

## dataframe

虽然说Series更加基础，但其实我们日常使用最多的还是dataframe。说简单点的话dataframe其实就是python版的excel，用来操作数据，有的同学可能喜欢使用sql语句对数据进行切片、聚合之类的处理，但是就我个人而言，dataframe操作起来要比sql人性化多了。


### 切片
dataframe里最基础的操作就是切片，相当于excel的选中行，选中列。在dataframe上切片的操作方法有很多，不过一般来说用的就那么几个。



```python
import pandas as pd
li = [[1,2,3],[4,5,6]]
df = pd.DataFrame(li,columns=["a","b","c"])
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>a</th>
      <th>b</th>
      <th>c</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>2</td>
      <td>3</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4</td>
      <td>5</td>
      <td>6</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 选中列,单个列名返回一个Series，列表形式选中返回DataFrame，即使是列表只有一个元素
df["a"]
```




    0    1
    1    4
    Name: a, dtype: int64




```python
df[["a","b"]]
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>a</th>
      <th>b</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4</td>
      <td>5</td>
    </tr>
  </tbody>
</table>
</div>




```python
df[["a"]]
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>a</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>




```python
#选中行
df.iloc[0]
```




    a    1
    b    2
    c    3
    Name: 0, dtype: int64




```python
df.loc[0,:]
```




    a    1
    b    2
    c    3
    Name: 0, dtype: int64




```python

```
