# Pandas：变形
> 准备环境
```python
import numpy as np
import pandas as pd
df = pd.read_csv('data/table.csv')
df.head()
```

### 一、透视表

**pivot**
> 一般状态下，数据在DataFrame会以压缩（stacked）状态存放，例如上面的Gender，两个类别被叠在一列中，pivot函数可将某一列作为新的cols：
```python
df.pivot(index='ID',columns='Gender',values='Height').head()
```

**pivot_table**
```python
pd.pivot_table(df,index='ID',columns='Gender',values='Height').head()
# 多功能
%timeit df.pivot(index='ID',columns='Gender',values='Height')
%timeit pd.pivot_table(df,index='ID',columns='Gender',values='Height')
```
**pivot_table常用参数**
```python
#aggfunc：对组内进行聚合统计，可传入各类函数，默认为'mean'
pd.pivot_table(df,index='School',columns='Gender',values='Height',aggfunc=['mean','sum']).head()

# margins：汇总边际状态
pd.pivot_table(df,index='School',columns='Gender',values='Height',aggfunc=['mean','sum'],margins=True).head()


# 行、列、值都可以为多级
pd.pivot_table(df,index=['School','Class'],columns=['Gender','Address'],
values=['Height','Weight'])
```

**crosstab**
> 交叉表是一种特殊的透视表，典型的用途如分组统计，如现在想要统计关于街道和性别分组的频数
```python
pd.crosstab(index=df['Address'],columns=df['Gender'])
# 参数
# values和aggfunc：分组对某些数据进行聚合操作，这两个参数必须成对出现
pd.crosstab(index=df['Address'],columns=df['Gender'],values=np.random.randint(1,20,df.shape[0]),aggfunc='min')

# 除了边际参数margins外，还引入了normalize参数，可选'all','index','columns'参数值
pd.crosstab(index=df['Address'],columns=df['Gender'],normalize='all',margins=True)
```

### 二、其他变形方法

**melt**
> melt函数可以认为是pivot函数的逆操作，将unstacked状态的数据，压缩成stacked，使“宽”的DataFrame变“窄“
```python
df_m = df[['ID','Gender','Math']]
df_m.head()
df.pivot(index='ID',columns='Gender',values='Math').head()
# melt函数中的id_vars表示需要保留的列，value_vars表示需要stack的一组列
pivoted = df.pivot(index='ID',columns='Gender',values='Math')
result = pivoted.reset_index().melt(id_vars=['ID'],value_vars=['F','M'],value_name='Math').dropna().set_index('ID').sort_index()
#检验是否与展开前的df相同，可以分别将这些链式方法的中间步骤展开，看看是什么结果
result.equals(df_m.set_index('ID'))
```

**stack/unstack**
> 这是最基础的变形函数，总共只有两个参数：level和dropna¶
```python
df_s = pd.pivot_table(df,index=['Class','ID'],columns='Gender',values=['Height','Weight'])
df_s.groupby('Class').head(2)
# stack函数可以看做将横向的索引放到纵向，因此功能类似与melt，参数level可指定变化的列索引是哪一层（或哪几层，需要列表）
df_stacked = df_s.stack(0)
df_stacked.groupby('Class').head(2) 

# unstack：stack的逆函数，功能上类似于pivot_table
df_stacked.head()
result = df_stacked.unstack().swaplevel(1,0,axis=1).sort_index(axis=1)
result.equals(df_s)
#同样在unstack中可以指定level参数
```

### 三、哑变量与因子化

**Dummy Variable**
> 哑变量

```python
df_d = df[['Class','Gender','Weight']]
df_d.head()

# 现在希望将上面的表格前两列转化为哑变量，并加入第三列Weight数值
pd.get_dummies(df_d[['Class','Gender']]).join(df_d['Weight']).head()
#可选prefix参数添加前缀，prefix_sep添加分隔符
```
**factorize**
> 该方法主要用于自然数编码，并且缺失值会被记做-1，其中sort参数表示是否排序后赋值
```python
codes, uniques = pd.factorize(['b', None, 'a', 'c', 'b'], sort=True)
display(codes)
display(uniques)
```
