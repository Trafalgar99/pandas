# 第三章 分组

+ 初始代码：
```python
import numpy as np
import pandas as pd
df = pd.read_csv('data/table.csv',index_col='ID')
df.head()
```
### SAC
> SAC指的是分组操作中的split-apply-combine过程
其中split指基于某一些规则，将数据拆成若干组，apply是指对每一组独立地使用函数，combine指将每一组的结果组合成某一类数据结构

**groupby**
> df.groupby(para)表示按几个列进行分会返回一个groupby对象，group对象调用相应方法会有相应显示
```python
grouped_single = df.groupby('School')
# 取出school中为s_1的行 
grouped_single.get_group('S_1').head()

grouped_mul = df.groupby(['School','Class'])
grouped_mul.get_group(('S_2','C_4'))
```
```python
grouped_single.size() # 每个组的数量
grouped_mul.size() 
grouped_single.ngroups # 有几组
grouped_mul.ngroups
```
```python
# groupby是一个可迭代对象
for name,group in grouped_single:
    print(name)
    display(group.head())
```
```python
# 对第二级索引，列，分组，取出“s_1”组
df.set_index(['Gender','School']).groupby(level=1,axis=0).get_group('S_1').head()
```
> groupby分组依据十分灵活
```python
df.groupby(np.random.choice(['a','b','c'],df.shape[0])).get_group('a').head()
# np.random.choice(['a','b','c'],df.shape[0])会返回一列，groupby将这列数据与index对应，但是不将这列新加入数据，分组时按找这‘不可见’的一列数据分组
```
*注：groupby的参数还支持函数，尤其时匿名函数，但是python现在并不提倡使用匿名函数，尽量用生成器代替*
> groupby与[]
```python
# 按gender和school分组后每组的数学成绩的均值
df.groupby(['Gender','School'])['Math'].mean()>=60
```

**聚合groupby.agg**
> 所谓聚合就是把一堆数，变成一个标量，因此mean/sum/size/count/std/var/sem/describe/first/last/nth/min/max都是聚合函数
```python
group_m = grouped_single['Math']
group_m.std().values/np.sqrt(group_m.count().values)== group_m.sem().values

group_m.agg(['sum','mean','std']) # 同时使用多个聚合函数

group_m.agg([('rename_sum','sum'),('rename_mean','mean')]) # 利用元组进行重命名

grouped_mul.agg({'Math':['mean','max'],'Height':'var'}) # 指定哪些函数作用哪些列
```
**过滤**
> filter函数是用来筛选某些组的（务必记住结果是组的全体），因此传入的值应当是布尔标量
```python
grouped_single[['Math','Physics']].filter(lambda x:(x['Math']>32).all()).head()
```
**apply函数**
```python
df.groupby('School').apply(lambda x:print(x.head(1)))
```


