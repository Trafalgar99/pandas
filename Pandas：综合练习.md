# Pandas：综合练习
> 准备
```python
import numpy as np
import pandas as pd
df = pd.read_csv('data/2002年-2018年上海机动车拍照拍卖.csv')
```

**练习一**

2002 年-2018 年上海机动车拍照拍卖
(1) 哪一次拍卖的中标率首次小于 5%?
```python
df['Auction winning rate'] = df1['Total number of license issued'] / df['Total number of applicants']
df1[df1['Auction winning rate']<0.05]['Date'].values[0]
```
(3) 将第一列时间列拆分成两个列，一列为年份(格式为 20××)，另一列为月份(英语缩写)，添加到列表作为第一第二列，并将原表第一列删除， 其他列依次向后顺延
```python
years = []
months = []
for i in list(df['Date'].values):
    years.append(i.split('-')[0])
    months.append(i.split('-')[1])
df['Year'] = years
df['Year'] = df['Year'].apply(lambda x: '200'+str(x) if len(str(x))==1 else '20'+str(x))
df['Month'] = months
dfYearMonth = df.reindex(columns=['Year', 'Month', 'Date', 'Total number of license issued', 'lowest price ', 'avg price', 'Total number of applicants', 'Auction winning rate'])
dfYearMonth = dfYearMonth.drop(columns='Date')
del dfYearMonth['Auction winning rate']
dfYearMonth.head()
```
(2)按年统计拍卖最低价的下列统计量：最大值、均值、0.75 分位数，要求显示在同一张表上。
```python
dfSum = dfYearMonth.groupby('Year')
dfSum.describe(percentiles=[.75])
```

(4) 现在将表格行索引设为多级索引，外层为年份，内层为原表格第二至第五列的变量名，列索引为月份
```python
mulIndex = pd.MultiIndex.from_product([['Year'],['Total number of license issued', 'lowest price ', 'avg price', 'Total number of applicants']],names=('Upper', 'Lower'))
pd.pivot_table(dfYearMonth,index=['Year'],columns=['Month']
```
(5) 一般而言某个月最低价与上月最低价的差额，会与该月均值与上月均值的差额具有相同的正负号，哪些拍卖时间不具有这个特点
```python
df2=df[['Year','Month','lowest price ','avg price']].copy()
df2=df2.iloc[1:].reset_index()[['Month','lowest price ','avg price']].join(df2,rsuffix='_lastmonth',how='outer')
df2[((df2['lowest price ']-df2['lowest price _lastmonth'])*(df2['avg price']-df2['avg price_lastmonth']))<0][['Year','Month']]
```


