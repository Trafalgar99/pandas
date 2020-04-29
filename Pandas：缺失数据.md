# Pandas：缺失数据
> Pandas在步入1.0后，对数据类型也做出了新的尝试，尤其是Nullable类型和String类型，了解这些可能在未来成为主流的新特性是必要的
```python
import pandas as pd
import numpy as np
df = pd.read_csv('data/table_missing.csv')
df.head()
```

### 一、缺失观测及其类型

**1. 了解缺失信息**

*isna和notna方法*
> isna和notna方法
```python
df['Physics'].isna().head()
df['Physics'].notna().head()
df.isna().head()
df.isna().sum()
df.info()
```
> 查看缺失值的所以在行
```python
df[df['Physics'].isna()]
df[df.notna().all(1)]
```

**2. 三种缺失符号**

*np.nan*
> np.nan是一个麻烦的东西，首先它不等与任何东西，甚至不等于自己
```python
    np.nan == np.nan
    np.nan == 0
    np.nan == None
    # 在用equals函数比较时，自动略过两侧全是np.nan的单元格，因此结果不会影响
    df.equals(df)
    # 其次，它在numpy中的类型为浮点，由此导致数据集读入时，即使原来是整数的列，只要有缺失值就会变为浮点型
    type(np.nan)
    pd.Series([1,2,3]).dtype
    pd.Series([1,np.nan,3]).dtype
    # 此外，对于布尔类型的列表，如果是np.nan填充，那么它的值会自动变为True而不是False
    pd.Series([1,np.nan,3],dtype='bool')
    s = pd.Series([True,False],dtype='bool')
s[1]=np.nan
s
```
*NaT*
> NaT是针对时间序列的缺失值，是Pandas的内置类型，可以完全看做时序版本的np.nan，与自己不等，且使用equals是也会被跳过

```python
s_time = pd.Series([pd.Timestamp('20120101')]*5)
s_time
s_time[2] = None
s_time
s_time[2] = np.nan
s_times_time[2] = pd.NaT
s_time
type(s_time[2])
s_time[2] == s_time[2]
s_time.equals(s_time)
s = pd.Series([True,False],dtype='bool')
s[1]=pd.NaT
s
```

**Nullable类型与NA符号**
> 这是Pandas在1.0新版本中引入的重大改变，其目的就是为了（在若干版本后）解决之前出现的混乱局面，统一缺失值处理方法
"The goal of pd.NA is provide a “missing” indicator that can be used consistently across data types (instead of np.nan, None or pd.NaT depending on the data type)."——User Guide for Pandas v-1.0
官方鼓励用户使用新的数据类型和缺失类型pd.NA
```python
# Nullable整形
# 对于该种类型而言，它与原来标记int上的# 符号区别在于首字母大写：'Int'
s_original = pd.Series([1, 2], dtype="int64")
s_original
s_new = pd.Series([1, 2], dtype="Int64")
s_new
s_original[1] = np.nan
s_original
s_new[1] = np.nan
s_new
s_new[1] = None
s_new
s_new[1] = pd.NaT
s_new

# Nullable布尔
# 对于该种类型而言，作用与上面的类似，记# 号为boolean
s_original = pd.Series([1, 0], dtype="bool")
s_original
s_new = pd.Series([0, 1], dtype="boolean")
s_new
s_original[0] = np.nan
s_original
s_original = pd.Series([1, 0], dtype="bool") #此处重新加一句是因为前面赋值改变了bool类型
s_original[0] = None
s_original
s_new[0] = np.nan
s_new
s_new[0] = None
s_new
s_new[0] = pd.NaT
s_new
s = pd.Series(['dog','cat'])
s[s_new]

# string类型
#该类型是1.0的一大创新，目的之一就是为了区分开原本含糊不清的object类型，这里将简要地提及string，因为它是第7章的主题内容
#它本质上也属于Nullable类型，因为并不会因为含有缺失而改变类型
s = pd.Series(['dog','cat'],dtype='string')
s
s[0] = np.nan
s
s[0] = None
s
```

**NA**
```python
# 逻辑运算
#只需看该逻辑运算的结果是否依赖pd.NA的取值，如果依赖，则结果还是NA，如果不依赖，则直接计算结果
True | pd.NA
pd.NA | True
False | pd.NA
False & pd.NA

# 算术运算和比较运算
pd.NA ** 0
1 ** pd.NA
```

**5. convert_dtypes方法**
> 这个函数的功能往往就是在读取数据时，就把数据列转为Nullable类型，是1.0的新函数
```python
pd.read_csv('data/table_missing.csv').dtypes
pd.read_csv('data/table_missing.csv').convert_dtypes().dtypes
```

### 二、缺失数据的运算与分组

**1. 加号与乘号规则**
> 使用加法时，缺失值为0

```python
s = pd.Series([2,3,np.nan,4])
s.sum()
```

> 使用乘法时，缺失值为1

```python
s.prod()
```

**groupby方法中的缺失值**
> 自动忽略为缺失值的组
```python
df_g = pd.DataFrame({'one':['A','B','C','D',np.nan],'two':np.random.randn(5)})
df_g
df_g.groupby('one').groups
```

### df_g.groupby('one').groups

**fillna**
```python
df['Physics'].fillna('missing').head()
df['Physics'].fillna(method='ffill').head()
df['Physics'].fillna(method='backfill').head()
```

**dropna**
```python
df_d = pd.DataFrame({'A':[np.nan,np.nan,np.nan],'B':[np.nan,3,2],'C':[3,2,1]})
df_d
df_d.dropna(axis=0)
df_d.dropna(axis=1)
```

### 四、插值（interpolation）

**1. 线性插值**
```python
s = pd.Series([1,10,15,-5,-2,np.nan,np.nan,28])
s
s.interpolate()

s.index = np.sort(np.random.randint(50,300,8))
s.interpolate()
#值不变
```

**2. 高级插值方法**
> 此处的高级指的是与线性插值相比较，例如样条插值、多项式插值、阿基玛插值等（需要安装Scipy），方法详情请看这里
关于这部分仅给出一个官方的例子，因为插值方法是数值分析的内容，而不是Pandas中的基本知识：
```python
ser = pd.Series(np.arange(1, 10.1, .25) ** 2 + np.random.randn(37))
missing = np.array([4, 13, 14, 15, 16, 17, 18, 20, 29])
ser[missing] = np.nan
methods = ['linear', 'quadratic', 'cubic']
df = pd.DataFrame({m: ser.interpolate(method=m) for m in methods})
df.plot()
```

**3. interpolate中的限制参数**
```python
s = pd.Series([1,np.nan,np.nan,np.nan,5])
s.interpolate(limit=2)
s = pd.Series([np.nan,np.nan,1,np.nan,np.nan,np.nan,5,np.nan,np.nan,])
s.interpolate(limit_direction='backward')
s = pd.Series([np.nan,np.nan,1,np.nan,np.nan,np.nan,5,np.nan,np.nan,])
s.interpolate(limit_area='inside')
s = pd.Series([np.nan,np.nan,1,np.nan,np.nan,np.nan,5,np.nan,np.nan,])
s.interpolate(limit_area='outside')
```