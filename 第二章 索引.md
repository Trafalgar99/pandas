# 第二章 索引
> 本章主要介绍Pandas数据结构中的各种索引方法
```python
import numpy as np
import pandas as pd
df = pd.read_csv('data/table.csv',index_col='ID')
df.head()
```
### 一、单机索引
#### 1. loc方法、iloc方法、[]操作符 (常用的三种)
```python
# df.loc[要索引行的名称, 要索引的列的名称] （支持切片）
df.loc[1103]  # 取ID为1103的行
df.loc[[1103, 1105]]  # 取两行
df.loc[::-1]
df.loc[:, 'Height'] # 取列
# df.iloc与df.loc相似，i代表index，之按序号索引
```
> 注：请不要在行索引为浮点时使用[]操作符，因为在Series中的浮点[]并不是进行位置比较，而是值比较，非常特殊
```python
# 在python语法中[]本身就是一种语法糖，所以[]索引与常规相似
s = pd.Series(df['Math'],index=df.index)
s[0:4]
s[lambda x: x.index[16::-6]]
s[s>80]
# 在dataframe中[]索引行是只能用对应的序号（0，1, ...）,对列时可以是名称
# 但是可以用这种方式替代
row = df.index.get_loc(1102)
df[row:row+1]
```

#### 2、布尔索引
>  布尔符号：'&','|','~'：分别代表和and，或or，取反not
>  loc和[]中相应位置都能使用布尔列表选择
 ```python
 df[(df['Gender']=='F')&(df['Address']=='street_2')].head()
```
> isin方法 （is + in）感觉类似于bool方法
```python
df[df['Address'].isin(['street_1','street_4'])&df['Physics'].isin(['A','A+'])]
```

#### 3、快速标量索引
> 当只需要取一个元素时，at和iat方法能够提供更快的实现：
```python
display(df.at[1101,'School'])
display(df.loc[1101,'School'])
display(df.iat[0,0])
display(df.iloc[0,0])
```

#### 4、区间索引
> 原作这写的太。。。，不做解释了
```python
pd.interval_range(start=0,end=5)
#closed参数可选'left''right''both''neither'，默认左开右闭
pd.interval_range(start=0,periods=8,freq=5)
#periods参数控制区间个数，freq控制步长
math_interval = pd.cut(df['Math'],bins=[0,40,60,80,100])
#注意，如果没有类型转换，此时并不是区间类型，而是category类型
df_i = df.join(math_interval,rsuffix='_interval')[['Math','Math_interval']]\
            .reset_index().set_index('Math_interval')
df_i.head()
df_i.loc[65].head()
#包含该值就会被选中
df_i.loc[[65,90]].head()
#df_i.loc[pd.Interval(70,75)].head() 报错
df_i[df_i.index.astype('interval').overlaps(pd.Interval(70, 85))].head()
```
### 二、多级索引
#### 1、创建多级索引
> 创建多级索引的方式是from_tuple和from_arrays
> 他们就收的第一个参数时一个合适的可迭代对象，第二个参数是每一级列的索引的名字
```python
tuples = [('A','a'),('A','b'),('B','a'),('B','b')]
mul_index = pd.MultiIndex.from_tuples(tuples, names=('Upper', 'Lower'))
pd.DataFrame({'Score':['perfect','good','fair','bad']},index=mul_index)
```
> 用zip函数
```python
L1 = list('AABB')
L2 = list('abab')
tuples = list(zip(L1,L2))
mul_index = pd.MultiIndex.from_tuples(tuples, names=('Upper', 'Lower'))
pd.DataFrame({'Score':['perfect','good','fair','bad']},index=mul_index)
```
*  **拓展：** 使用 for 循环迭代元素不用处理索引变量，还能避免很多缺陷，但是需要一些特殊的 实用函数协助。其中一个是内置的 zip 函数。使用 zip 函数能轻松地并行迭代两个或 更多可迭代对象，它返回的元组可以拆包成变量，分别对应各个并行输入中的一个元 素zip 函数的名字取自拉链系结物（zipper fastener），因为这个物品用于把 两个拉链边的链牙咬合在一起，这形象地说明了 zip(left, right) 的作 用。zip 函数与文件压缩没有关系*

#### 2、多级索引切片
```python
#df_using_mul.loc['C_2','street_5']
#当索引不排序时，单个索引会报出性能警告
#df_using_mul.index.is_lexsorted()
#该函数检查是否排序
df_using_mul.sort_index().loc['C_2','street_5']
#df_using_mul.sort_index().index.is_lexsorted()
```
> 第一类特殊情况：由元组构成列表
```python
df_using_mul.sort_index().loc[[('C_2','street_7'),('C_3','street_2')]]
#表示选出某几个元素，精确到最内层索引
```
> 第二类特殊情况：由列表构成元组
```python
df_using_mul.sort_index().loc[(['C_2','C_3'],['street_4','street_7']),:]
#选出第一层在‘C_2’和'C_3'中且第二层在'street_4'和'street_7'中的行
```

#### 3、多层索引中的slice对象
```python
L1,L2 = ['A','B','C'],['a','b','c']
mul_index1 = pd.MultiIndex.from_product([L1,L2],names=('Upper', 'Lower'))
L3,L4 = ['D','E','F'],['d','e','f']
mul_index2 = pd.MultiIndex.from_product([L3,L4],names=('Big', 'Small'))
df_s = pd.DataFrame(np.random.rand(9,9),index=mul_index1,columns=mul_index2)
df_s
```

### 三、索引设定
> index_col是read_csv中的一个参数，而不是某一个方法
```python
pd.read_csv('data/table.csv',index_col=['Address','School']).head()
```
> reindex是指重新索引，它的重要特性在于索引对齐，很多时候用于重新排序
```python
df.reindex(index=[1101,1203,1206,2402])
df.reindex(columns['Height','Gender','Average']).head()
```
> 可以选择缺失值的填充方法：fill_value和method（bfill/ffill/nearest），其中method参数必须索引单调
```python
df.reindex(index=[1101,1203,1206,2402],method='bfill')
#bfill表示用所在索引1206的后一个有效行填充，ffill为前一个有效行，nearest是指最近的
```
> reindex_like的作用为生成一个横纵索引完全与参数列表一致的DataFrame，数据使用被调用的表
```python
df_temp = pd.DataFrame({'Weight':np.zeros(5),
                        'Height':np.zeros(5),
                        'ID':[1101,1104,1103,1106,1102]}).set_index('ID')
df_temp.reindex_like(df[0:5][['Weight','Height']])
```

### 四、常用索引函数
#### 1，where,mask函数
```python
df.where(df['Gender']=='M').head()
#不满足条件的行全部被设置为NaN
df.mask(df['Gender']=='M').dropna().head()
```
#### 2、query函数
> query函数中的布尔表达式中，下面的符号都是合法的：行列索引名、字符串、and/not/or/&/|/~/not in/in/==/!=、四则运算符
```python
df.query('(Address in ["street_6","street_7"])&(Weight>(70+10))&(ID in [1303,2304,2402])')
```

### 五、重复元素处理
#### 1. duplicated方法
```pthon
df.duplicated('Class').head()
```

#### 2. drop_duplicates方法
> 从名字上看出为剔除重复项，这在后面章节中的分组操作中可能是有用的，例如需要保留每组的第一个值：
```python
df.drop_duplicates('Class')
```

### 六、抽样函数
> 即sample
```python
df.sample(n=5)
# n为样本量
df.sample(frac=0.05)
# frac为抽样比
df.sample(n=df.shape[0],replace=True).head()
# replace为是否放回
```


### 练习
【练习一】 现有一份关于UFO的数据集，请解决下列问题：
（a）在所有被观测时间超过60s的时间中，哪个形状最多
（b）对经纬度进行划分：-180°至180°以30°为一个划分，-90°至90°以18°为一个划分，请问哪个区域中报告的UFO事件数量最多？
```python
import pandas as pd
import numpy as np
# a
df = pd.read_csv('data/UFO.csv')
df[df['duration (seconds)']>60]['shape'].value_counts().index[0]

# b
df = pd.read_csv('data/UFO.csv')
index_lo = pd.cut(df['longitude'],bins=pd.interval_range(start=-180,end=180,freq=30))
index_la = pd.cut(df['latitude'],bins=pd.interval_range(start=-90,end=90,freq=18))
df['index_lo'] = index_lo
df['index_la'] = index_la
df.set_index(['index_lo','index_la']).index.value_counts().index[0]

```
【练习二】 现有一份关于口袋妖怪的数据集，请解决下列问题
（a）双属性的Pokemon占总体比例的多少？
（b）在所有种族值（Total）不小于580的Pokemon中，非神兽（Legendary=False）的比例为多少？
（c）在第一属性为格斗系（Fighting）的Pokemon中，物攻排名前三高的是哪些？
（d）请问六项种族指标（HP、物攻、特攻、物防、特防、速度）极差的均值最大的是哪个属性（只考虑第一属性，且均值是对属性而言）？
（e）哪个属性（只考虑第一属性）的神兽比例最高？该属性神兽的种族值也是最高的吗？

```python
# a
df = pd.read_csv('data/Pokemon.csv')
df['Type 2'].count()/df.shape[0]

# b
df[df["Total"]>=580][df['Legendary']==False].shape[0]/df.shape[0]

# c
df[df['Type 1']=='Fighting'].set_index('Attack').sort_index()[::-1][0:3]

# d  不是很理解这道题的意思
df['range'] = df.iloc[:,5:11].max(axis=1)-df.iloc[:,5:11].min(axis=1)
attribute = df[['Type 1','range']].set_index('Type 1')
max_range = 0
result = ''
for i in attribute.index.unique():
    temp = attribute.loc[i,:].mean()
    if temp.values[0] > max_range:
        max_range = temp.values[0]
        result = i
result

# e
df[df['Legendary']==True]['Type 1'].value_counts().index[0]
attribute = df.query('Legendary == True')[['Type 1','Total']].set_index('Type 1')
max_value = 0
result = ''
for i in attribute.index.unique()[:-1]:
    temp = attribute.loc[i,:].mean()
    if temp[0] > max_value:
        max_value = temp[0]
        result = i
result

```
