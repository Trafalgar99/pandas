# 第一章 Pandas基础

1. *导入pandas与numpy模块：*

   ```python
   import pandas as pandas
   import numpy as np
   ```

2. *查看pandas的版本：*

   ```python
   print(pd.__version__)  # 打印处pd的__version__属性
   ```

   输出结果：'1.0.3'

## 一、文件读取与写入

#### 1.读取

*(a) csv格式*

```python
df = pd.read_csv('data/table.csv')  # 此处参数是要读取的文件的路径(.csv格式)
print(type(df))  # 获取df的类型，为pandas.core.frame.DataFrame类
print(df.head())  # head()返回数据的前五行(类似于Linux中的head命令)
```

输出结果：

 ![image-20200419111640933](.\images\image-20200419111640933.png)



*(b) txt格式*

```python
df_txt = pd.read_table('data/table.txt')  # 可设置sep分隔符参数
print(df_txt)
```

输出结果：<img src=".\images\image-20200419112045497.png" alt="image-20200419112045497" style="zoom:67%;" />



*(c) xls/xlsx格式*

```python
# 需要安装xlrd包
df_excel = pd.read_excel('data/table.xlsx')
print(df_excel.head())
```

输出结果：

<img src=".\images\image-20200419112243496.png" alt="image-20200419112243496" style="zoom: 67%;" />



#### 2、写入

> 指定写入格式，通过传入参数还可以在转换格式时设置其他文本属性
>
> 关于to_csv和to_excel的参数信息详见：https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.to_csv.html

*(a) csv格式*

```python
df.to_csv('data/new_table.csv')  # 将df零保存为csv格式，此处参数为新文件名
```

结果：data文件夹下生成一新文件，名为new_table.csv，并且内容不变



*(b) xls或xlsx格式*

```python
# 需要安装openpyxl
df.to_excel('data/new_table2.xlsx', sheet_name='Sheet1')
```

结果：data文件夹下生成一新文件，名为new_table.csv，并且内容不变



## 二、基本数据结构



#### 1、Series

> Series对象本质上是一个NumPy的数组，因此NumPy的数组处理函数可以直接对Series进行处理。但是Series除了可以使用位置作为下标存取元素之外，还可以使用标签下标存取元素，这一点和字典相似。每个Series对象实际上都由两个数组组成。
>
> 对于一个Series，其中最常用的属性为：
>
> 值（values）：可以是一个可迭代的对象。
>
> 索引（index）：可自行设定
>
> 名字（name），类型（dtype）。



*（a）创建一个Series*

```python
s = pd.Series(np.random.randn(5),index=['a','b','c','d','e'],name='这是一个Series',dtype='float64')
print(s)
```

输出结果：<img src=".\images\image-20200419122342372.png" alt="image-20200419122342372" style="zoom:67%;" />

*（b）访问Series属性*

```python
print(s.values)
print(s.name)
print(s.index)
print(s.dtype)
```

输出结果：<img src=".\images\image-20200419123154263.png" alt="image-20200419123154263" style="zoom:80%;" />

*（c）取出某一个元素*

```python
print(s['a'])  # 说明Series内部实现了__getitem__方法
```

输出结果：0.241320545363

（d）调用series的方法

```python
print(s.mean())
# 查看Series的其他方法
print([attr for attr in dir(s) if not attr.startswith('_')])
```

输出结果：3.0

['T', 'a', 'abs', 'add', 'add_prefix', 'add_suffix', 'agg', 'aggregate', 'align', 'all', 'any', 'append'...]



#### **2、DataFrame**

> data frame 是一种将数据存储在矩形网格中的方法，网格的每一行对应于实例的度量或值，而每一列则包含特定变量的数据的向量。



*（a）创建一个DataFrame*

```python
df = pd.DataFrame({'col1':list('abcde'),'col2':range(5,10),'col3'
                   [1.3,2.5,3.6,4.6,5.8]},index=list('一二三四五'))
print(df)
```

输出结果：<img src=".\images\image-20200419124404028.png" alt="image-20200419124404028" style="zoom:67%;" />

*（b）从DataFrame取出一列为Series*

```python
print(df['col1'])
print(type(df))
```

输出结果：<img src=".\images\image-20200419124912600.png" alt="image-20200419124912600" style="zoom:67%;" />

*（c）修改行或列名*

```python
df = df.rename(index={'一':'one'},columns={'col1':'new_col1'})
print(df)
```

输出结果：<img src=".\images\image-20200419125123612.png" alt="image-20200419125123612" style="zoom:67%;" />



*（d）调用属性和方法*

```python
print(df.index)
print(df.columns)
print(df.values)
print(df.shape)  # df的[行数，列数]
print(df.mean())  # 求每列的平均值，仅针对数字
```

输出结果：

<img src=".\images\image-20200419125411259.png" alt="image-20200419125411259" style="zoom: 50%;" />

*（e）索引对齐特性*

> 如下代码，在执行df1-df2时，会将key相同value相减

```python
df1 = pd.DataFrame({'A':[1,2,3]},index=[1,2,3])
df2 = pd.DataFrame({'A':[1,2,3]},index=[3,1,2])
df1-df2 #由于索引对齐，因此结果不是0
```

输出结果：<img src=".\images\image-20200419130001471.png" alt="image-20200419130001471" style="zoom:67%;" />

*（f）列的删除与添加*

> 对于删除而言，可以使用drop函数或del或pop
>
> 设置inplace=True后会直接在原DataFrame中改动，相当于df = df.drop(...)

```python
df.drop(index='五',columns='col1') # 删除第五行和第一列
print(df)
```

输出结果：<img src=".\images\image-20200419130900925.png" alt="image-20200419130900925" style="zoom:67%;" />

```python
df['col1']=[1,2,3,4,5]
del df['col1']  # 删除引用
print(df)
```

输出结果：<img src=".\images\image-20200419131046194.png" alt="image-20200419131046194" style="zoom:67%;" />

```python
df['col1']=[1,2,3,4,5]
print(df.pop('col1'))  # 这里的pop机制类似于普遍的pop函数
```

结果：<img src=".\images\image-20200419131250968.png" alt="image-20200419131250968" style="zoom:67%;" />

> 可以直接增加新的列，也可以使用assign方法

```python
df1['B']=list('abc')
df1.assign(C=pd.Series(list('def')))  # assign不会修改原对象，会返回修改后的对象
```

输出结果：<img src=".\images\image-20200419131457194.png" alt="image-20200419131457194" style="zoom:67%;" />

*（g）根据类型选择列*

```python
print(df.select_dtypes(include=['number']).head())
print(df.select_dtypes(include=['float']).head())
```

输出结果：<img src=".\images\image-20200419132155599.png" alt="image-20200419132155599" style="zoom:67%;" />

*（h）将Series转换为DataFrame*

```python
s = df.mean()
s.name='to_DataFrame'
print(s)
```

输出结果：<img src=".\images\image-20200419132336522.png" alt="image-20200419132336522" style="zoom:67%;" />

```python
print(s.to_frame())
```

输出结果：<img src=".\images\image-20200419132518729.png" alt="image-20200419132518729" style="zoom:67%;" />

> 使用T符号可以转置

```python
print(s.to_frame().T)
```

输出结果：<img src=".\images\image-20200419132652170.png" alt="image-20200419132652170" style="zoom:67%;" />



## 三、常用基本函数

1. 初始化接下来一节要用的文件

```python
df = pd.read_csv('data/table.csv')
```



#### 1、head和tail

```python
print(df.head())
print(df.tail())
print(df.head(3))  # 可以指示显示多少行
```

输出结果：

<img src=".\images\image-20200419133256079.png" alt="image-20200419133256079" style="zoom:50%;" />

#### 2、unique和nunique

> nunique显示有多少个唯一值

```python
print(df['Physics'].nunique())
```

输出结果：7

> unique显示所有的唯一值

```python
print(df['Physics'].unique())
```

输出结果：<img src=".\images\image-20200419133526620.png" alt="image-20200419133526620" style="zoom:67%;" />

#### 3、count和value_counts

> count返回非缺失值元素个数

```python
print(df['Physics'].count())
```

输出结果：35

> value_counts返回每个元素有多少个

```python
print(df['Physics'].value_counts())
```

输出结果：<img src=".\images\image-20200419133821687.png" alt="image-20200419133821687" style="zoom:67%;" />

#### 4、 describe和info

> info函数返回有哪些列、有多少非缺失值、每列的类型

```python
print(df.info())
```

输出结果：<img src=".\images\image-20200419134001777.png" alt="image-20200419134001777" style="zoom: 50%;" />

> describe默认统计数值型数据的各个统计量

```python
print(df.describe()) # 可通过percentiles=[.05, .25, .75, .95]控制位数
```

输出结果：<img src=".\images\image-20200419134200554.png" alt="image-20200419134200554" style="zoom:67%;" />

#### 7、 apply函数

```python
df['Math'].apply(lambda x:str(x)+'!') #可以使用lambda表达式，也可以使用函数

# 由python理论可知，高阶函数的使用可以转化为推导式的使用，所以上述代码等价于
df['Math'] = [str(n)+'!' for n in df['Math']]

```



## 练习一

代码：

```python
s1 = pd.read_csv('data/Game_of_Thrones_Script.csv')  # 要统计的对象
print('共计人数：', s1["Name"].nunique())  # 出现人数
print('(按单元统计)说话最多：', s1['Name'].value_counts().index[0])
# 按单词统计
s1.assign(Words=s1['Sentence'].apply(lambda  x:len(x.split()))).sort_values(by='Name')
s1_list = list(zip(s1['Name'], s1['Words']))
l1 = []
for n in s1_list:
    for i in range(n[1]):
        l1.append(n[0])
c = collections.Counter(l1)  # 需要collections模块
print('(按单词统计)说话最多：', c.most_common()[0][0], "说了：", c.most_common()[0][1], '个词')

```

输出结果：<img src=".\images\image-20200419163412583.png" alt="image-20200419163412583" style="zoom:67%;" />



## 练习二

代码：

```python
s2 = pd.read_csv('data/Kobe_data.csv', index_col='shot_id')
s2_two_list = pd.Series(list(zip(s2['action_type'],s2['combined_shot_type']))).value_counts()
print('最多组合：', s2_two_list.index[0])
print('最常遇到的队伍ID：', s2['game_id'].value_counts().index[0])


print(pd.Series(list(list(zip(*(pd.Series(list(zip(s2['game_id'], s2['opponent'])))
                                .unique()).tolist()))[1])).value_counts().index[0])
```

输出结果：<img src=".\images\image-20200419170000446.png" alt="image-20200419170000446" style="zoom:67%;" />

