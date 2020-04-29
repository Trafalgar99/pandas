# Pandas：合并
> 准备环境
```python
import numpy as np
import pandas as pd
df = pd.read_csv('data/table.csv')
df.head()
```

### 一、append与assign

**append**
```python
# 利用序列添加行（必须指定name）
df_append = df.loc[:3,['Gender','Height']].copy()
df_append
s = pd.Series({'Gender':'F','Height':188},name='new_row')
df_append.append(s)

# 用DataFrame添加表
df_temp = pd.DataFrame({'Gender':['F','M'],'Height':[188,176]},index=['new_1','new_2'])
df_append.append(df_temp)
```

**assign**
> 该方法主要用于添加列，列名直接由参数指定：
```python
s = pd.Series(list('abcd'),index=range(4))
df_append.assign(Letter=s)

# 可以一次添加多个列：
df_append.assign(col1=lambda x:x['Gender']*2,col2=s)
```

### 二、combine与update

**comine**
> comine和update都是用于表的填充函数，可以根据某种规则填充
```python
# 填充对象可以看出combine方法是按照表的顺序轮流进行逐列循环的，而且自动索引对齐，缺失值为NaN，理解这一点很重要
df_combine_1 = df.loc[:1,['Gender','Height']].copy()
df_combine_2 = df.loc[10:11,['Gender','Height']].copy()
df_combine_1.combine(df_combine_2,lambda x,y:print(x,y))

# 一些例子
# 例子1
df1 = pd.DataFrame({'A': [1, 2], 'B': [3, 4]})
df2 = pd.DataFrame({'A': [8, 7], 'B': [6, 5]})
df1.combine(df2,lambda x,y:x if x.mean()>y.mean() else y)

# 例子二
df2 = pd.DataFrame({'B': [8, 7], 'C': [6, 5]},index=[1,2])
df1.combine(df2,lambda x,y:x if x.mean()>y.mean() else y)

# 例子三
df1.combine(df2,lambda x,y:x if x.mean()>y.mean() else y,fill_value=-1)
```

**combine_first**
> 这个方法作用是用df2填补df1的缺失值，功能比较简单，但很多时候会比combine更常用，下面举两个例子：
```python
df1 = pd.DataFrame({'A': [None, 0], 'B': [None, 4]})
df2 = pd.DataFrame({'A': [1, 1], 'B': [3, 3]})
df1.combine_first(df2)

df1 = pd.DataFrame({'A': [None, 0], 'B': [4, None]})
df2 = pd.DataFrame({'B': [3, 3], 'C': [1, 1]}, index=[1, 2])
df1.combine_first(df2)
```

**update**
> 返回的框索引只会与被调用框的一致
> 第二个框中的nan元素不会起作用
> 没有返回值，直接在df上操作
```python
df1 = pd.DataFrame({'A': [1, 2, 3],
'B': [400, 500, 600]})
df2 = pd.DataFrame({'B': [4, 5, 6],
'C': [7, 8, 9]})
df1.update(df2)
df1

# 部分填充
df1 = pd.DataFrame({'A': ['a', 'b', 'c'],
                    'B': ['x', 'y', 'z']})
df2 = pd.DataFrame({'B': ['d', 'e']}, index=[1,2])
df1.update(df2)
df1

# df1 = pd.DataFrame({'A': [1, 2, 3],
                    'B': [400, 500, 600]})
df2 = pd.DataFrame({'B': [4, np.nan, 6]})
df1.update(df2)
df1
```

### concat方法
**concat**
> concat方法可以在两个维度上拼接，默认纵向凭借（axis=0），拼接方式默认外连接¶
所谓外连接，就是取拼接方向的并集，而'inner'时取拼接方向（若使用默认的纵向拼接，则为列的交集）的交集
```python
df1 = pd.DataFrame({'A': ['A0', 'A1'],
'B': ['B0', 'B1']},index = [0,1])
df2 = pd.DataFrame({'A': ['A2', 'A3'],
'B': ['B2', 'B3']}, index = [2,3])
df3 = pd.DataFrame({'A': ['A1', 'A3'],'D': ['D1', 'D3'],'E': ['E1', 'E3']},index = [1,3])

# 默认状态拼接
pd.concat([df1,df2])

# 其他状态拼接
pd.concat([df1,df2],axis=1)
pd.concat([df3,df1],join='inner')
pd.concat([df3,df1],join='outer',sort=True) #sort设置列排序，默认为False

# 可以添加Series
s = pd.Series(['X0', 'X1'], name='X')
pd.concat([df1,s],axis=1)

pd.concat([df1,df2], keys=['x', 'y'])
pd.concat([df1,df2], keys=['x', 'y']).index
```

### 四、merge与join

**merge**
> merge函数的作用是将两个pandas对象横向合并，遇到重复的索引项时会使用笛卡尔积，默认inner连接，可选left、outer、right连接
所谓左连接，就是指以第一个表索引为基准，右边的表中如果不再左边的则不加入，如果在左边的就以笛卡尔积的方式加入
merge/join与concat的不同之处在于on参数，可以指定某一个对象为key来进行连接
```python
left = pd.DataFrame({'key1': ['K0', 'K0', 'K1', 'K2'],
                     'key2': ['K0', 'K1', 'K0', 'K1'],
                      'A': ['A0', 'A1', 'A2', 'A3'],
                      'B': ['B0', 'B1', 'B2', 'B3']}) 
right = pd.DataFrame({'key1': ['K0', 'K1', 'K1', 'K2'],
                      'key2': ['K0', 'K0', 'K0', 'K0'],
                      'C': ['C0', 'C1', 'C2', 'C3'],
                      'D': ['D0', 'D1', 'D2', 'D3']})
right2 = pd.DataFrame({'key1': ['K0', 'K1', 'K1', 'K2'],
                      'key2': ['K0', 'K0', 'K0', 'K0'],
                      'C': ['C0', 'C1', 'C2', 'C3']})

# 以key1为准则连接，如果具有相同的列，则默认suffixes=('_x','_y')：
pd.merge(left, right, on='key1')

pd.merge(left, right, on=['key1','key2'])
```

**join**
> join函数作用是将多个pandas对象横向拼接，遇到重复的索引项时会使用笛卡尔积，默认左连接，可选inner、outer、right连接
```python
left = pd.DataFrame({'A': ['A0', 'A1', 'A2'],
                     'B': ['B0', 'B1', 'B2']},
                    index=['K0', 'K1', 'K2'])
right = pd.DataFrame({'C': ['C0', 'C2', 'C3'],
                      'D': ['D0', 'D2', 'D3']},
                    index=['K0', 'K2', 'K3'])
left.join(right)

# 对于many_to_one模式下的合并，往往join更为方便
#同样可以指定key
left = pd.DataFrame({'A': ['A0', 'A1', 'A2', 'A3'],
                     'B': ['B0', 'B1', 'B2', 'B3'],
                     'key': ['K0', 'K1', 'K0', 'K1']})
right = pd.DataFrame({'C': ['C0', 'C1'],
                      'D': ['D0', 'D1']},
                     index=['K0', 'K1'])
left.join(right, on='key')
```