---
layout:     post
title:      "如何使用 Pandas"
subtitle:   "How To Use Pandas"
date:       2019-04-09
author:     "PengTuo"
catalog:    true
header-img: "img/home-bg-tmp.jpg"
categories: [Towards Data Science]
tags:
    - Python
    - Pandas
---

本文概览：
* TOC
{:toc}

## 一、什么是 pandas
`pandas` 是一个第三方工具包，可比作可 `Python` 操作的 `Excel`，其是你用 `Python` 操作数据的必备且目前来说是最好用的工具包。接下来介绍一些常用的 `pandas` 操作方法，供大家以及我自己在使用过程中可查阅。

## 二、Pandas 最常使用方法

导入 `pandas` 工具包
```python
import pandas as pd
```

### 2.1. Reading data

读 `CSV` 文件或者 `EXCEL` 文件

函数文档解释：
```python
pandas.read_csv(filepath_or_buffer, sep=', ', delimiter=None, header='infer', names=None, index_col=None, usecols=None, squeeze=False, prefix=None, mangle_dupe_cols=True, dtype=None, engine=None, converters=None, true_values=None, false_values=None, skipinitialspace=False, skiprows=None, nrows=None, na_values=None, keep_default_na=True, na_filter=True, verbose=False, skip_blank_lines=True, parse_dates=False, infer_datetime_format=False, keep_date_col=False, date_parser=None, dayfirst=False, iterator=False, chunksize=None, compression='infer', thousands=None, decimal=b'.', lineterminator=None, quotechar='"', quoting=0, escapechar=None, comment=None, encoding=None, dialect=None, tupleize_cols=None, error_bad_lines=True, warn_bad_lines=True, skipfooter=0, doublequote=True, delim_whitespace=False, low_memory=True, memory_map=False, float_precision=None)


pandas.read_excel(io, sheet_name=0, header=0, names=None, index_col=None, usecols=None, squeeze=False, dtype=None, engine=None, converters=None, true_values=None, false_values=None, skiprows=None, nrows=None, na_values=None, parse_dates=False, date_parser=None, thousands=None, comment=None, skipfooter=0, convert_float=True, **kwds)
```

使用方法：
```python
data = pd.read_csv('file.csv')

data = pd.read_csv('file.csv', sep=';', encoding='utf-8', nrows=1000, skiprows=[2,5])

data = pd.read_excel('tmp.xlsx', index_col=None, header=None)
```

### 2.2. Writing data

将数据写入 `CSV` 文件：

函数文档解释：
```python
DataFrame.to_csv(path_or_buf=None, sep=', ', na_rep='', float_format=None, columns=None, header=True, index=True, index_label=None, mode='w', encoding=None, compression=None, quoting=None, quotechar='"', line_terminator='\n', chunksize=None, tupleize_cols=None, date_format=None, doublequote=True, escapechar=None, decimal='.')
```

使用方法：
```python
data.to_csv('new_file.csv', index=None)
```

### 2.3. Seeing the data

```python
# 查看数据的行数和列数
data.shape

# 计算一些基本的统计信息，例如个数、平均数、方差、分位数等
data.describe()

# 查看数据的头或尾的多行
data.head(10)
data.tail(10)

# 查看数据的第 x 行
data.loc[x]

# 查看数据的第 x 行的 column_y 列
data.loc[x, 'column_y']

# 查看数据的第4行到第6行（不包括第6行）
data.loc[range(4,6)]
```

### 2.4. Logical operations

`pandas` 中可以使用逻辑运算选取中特定的子数据部分。

```python
# 选出 column_1 列值是 french 的数据
data[data['column_1']=='french']

# 选出 column_1 列值是 french 且 year_born 是 1990 的数据
data[(data['column_1']=='french') & (data['year_born']==1990)]

# 选出 column_1 列值是 french 且 year_born 是 1990 ，但是 city 不是 London 的数据
data[(data['column_1']=='french') & (data['year_born']==1990) & ~(data['city']=='London')]

# 选出 column_1 的值在 ['french', 'english'] 内的数据
data[data['column_1'].isin(['french', 'english'])]
```

### 2.5. Updating the data

```python
# 将第 8 行的 column_1 列值更新为 english
data.loc[8, 'column_1'] = 'english'

# 将 column_1 列值是 french 的数据的 column_2 列值更新为 French
data.loc[data['column_1']=='french', 'column_2'] = 'French'
```


## 三、Pandas 一些更厉害的操作

### 3.1. Counting occurrences

```python
# 计算数据的 column_1 列的所有值出现的次数及数据类型，如下图
data['column_1'].value_counts()
```
![](https://live.staticflickr.com/7838/33686190558_06c87e0d3d_o.jpg)

### 3.2. Operations on full rows, columns, or all data

`.map()` 可以将一个函数作用于某一列的所有值。
```python
data['column_1'].map(len).map(lambda x: x/100).plot()
```

`pandas` 的另一个很优秀的特质就是可以链式连接函数，例如上述例子，高效且简洁。

`.apply()` 则是可以选择将一个函数作用于列还是行。
```python
#  {0 or ‘index’, 1 or ‘columns’}, default 0
# Axis along which the function is applied:
# 0 or ‘index’: apply function to each column.
# 1 or ‘columns’: apply function to each row.
df.apply(np.sum, axis=0)
```

`DataFrame.applymap(func)` 则是将函数作用于整个 `DataFrame` 中的每一个元素。

### 3.3. Correlation
还可以直接计算 `DataFrame` 中数据的相关性。

```python
data.corr()
data.corr().applymap(lambda x: int(x*100)/100)
```
![](https://live.staticflickr.com/7847/47563774851_20c7d0977e_o.jpg)

### 3.4. The SQL join

将两个 `DataFrame` 根据其中的某些列合并。

函数文档解释：
```python
DataFrame.merge(right, how='inner', on=None, left_on=None, right_on=None, left_index=False, right_index=False, sort=False, suffixes=('_x', '_y'), copy=True, indicator=False, validate=None)
```

其中参数 `how` 的解释解释如下：
```
{‘left’, ‘right’, ‘outer’, ‘inner’}, default ‘inner’
left: use only keys from left frame, similar to a SQL left outer join; preserve key order
right: use only keys from right frame, similar to a SQL right outer join; preserve key order
outer: use union of keys from both frames, similar to a SQL full outer join; sort keys lexicographically
inner: use intersection of keys from both frames, similar to a SQL inner join; preserve the order of the left keys
```

```python
A.merge(B, left_on='lkey', right_on='rkey', how='outer')
```
上述代码示例：
![](https://live.staticflickr.com/7904/33686886948_de8c1aae7c_o.png)


### 3.5. Grouping

这个方法也使用很多，将数据根据某一列进行聚合，然后对聚合后的数据进行操作。

```python
# 数据根据 column_1 聚合，然后对 column_2 列值进行求和操作，.reset_index() 是将结果重新显示为一个 `DataFrame`
data.groupby('column_1')['column_2'].apply(sum).reset_index()
```
![](https://live.staticflickr.com/7863/40597877843_f7f6308dff_o.jpg)



## 四、如何利用 Pandas 来处理大数据量文件

这里说的大数据量文件并不是指大数据，一般大数据的都是要在 PB 级别的数据，这里的大数据量只是单纯的指代在单机上的、远远超过单机内存能够容纳的数据量，例如一个50G的文本文件。那么这种没有办法直接读入的数据文件该如何使用 Pandas 操作呢？

### 4.1. 利用 chunksize 参数

Pandas 中读取数据文件的方法主要使用的是 `pandas.read_csv(filepath, ...)` 方法，直接传入CSV文件的路径，Pandas 就会将数据读入内存，但是当面对40多G的文件时，则需要利用其另一个参数 -- `chunksize`

```python
# read the large csv file with specified chunksize 
df_chunk = pd.read_csv(r'../input/data.csv', chunksize=1000000)
```

chunksize参数的官方解释是：
![](https://farm8.staticflickr.com/7895/47534358661_eb6c4b8f3d_o.png)

加入了这个参数后，`df_chunk` 则不是一个 `dataframe` 对象，而是一个 `TextFileReader` 对象，可以以后用来迭代访问，例如下述代码所展示的，我们接下来可以迭代访问 `df_chunk` 中的每一个块(`chunk`)，对每一个块我们进行自己的数据操作，如代码中的 `chunk_preprocessing` 调用，每个块处理完成后加入一个 `list` 中，这个 `list` 此时就是能够符合内存大小的集合，然后使用 `pandas.concat()` 函数将这个 `list` 拼接成一个 `dataframe`。

```python
chunk_list = []  # append each chunk df here 

# Each chunk is in df format
for chunk in df_chunk:  
    # perform data filtering 
    chunk_filter = chunk_preprocessing(chunk)
    
    # Once the data filtering is done, append the chunk to list
    chunk_list.append(chunk_filter)
    
# concat the list into dataframe 
df_concat = pd.concat(chunk_list)
```

### 4.2. 选出重要的数据列来节省内存

在将数据读入以后，你就可以通过 `dataframe` 提供的操作来对数据进行操作，但是我们可以将数据中必要的或者重要的数据列提取出来，以达到加速数据的操作、计算以及更进一步的节省单机内存的需求。
```python
df = df[['col_1','col_2', 'col_3', 'col_4']]
```

### 4.3. 改变 dataframe 中数据类型 (dtypes)

`dtypes` 在 `pandas` 中用来表示数据类型，当需要某一个数据列进行较多的操作，或者将这列的数据需要放进机器学习模型进行训练，改变数据的类型将会非常有助于提高效率。

改变数据类型的方法有两种，首先是我们可以使用 `pandas` 中使用 `astype()` 来改变数据的类型。例如将64位整型转换为32位整型。

```python
# Change the dtypes (int64 -> int32)
df[['col_1','col_2','col_3','col_4','col_5']] = df[['col_1','col_2','col_3','col_4','col_5']].astype('int32')
```

另一种方法就是将分类数据属性数字化，较为常用的一种数字化的方法叫做 `One-Hot Encoding`，这种编码方式就是用 N 位 0、1 来表示该属性中包含的数据类别。例如某个数据属性包含 `[a, b, c, d]` 四种数据，则我们可以采用 `[1000, 0100, 0010, 0001]` 来表示这些类别。

这种表示的好处有两个，其一就是我们所说的数字化后又能节省更多的内存空间，另一个就是很多机器学习的模型都擅长处理数字化的特征，对分类特征进行数字化更有助于模型的训练和表现。


## 五、参考
[1] https://towardsdatascience.com/be-a-more-efficient-data-scientist-today-master-pandas-with-this-guide-ea362d27386

[2] https://towardsdatascience.com/why-and-how-to-use-pandas-with-large-data-9594dda2ea4c