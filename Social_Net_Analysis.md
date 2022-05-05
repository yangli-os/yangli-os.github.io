# 社交网络分析方法以及基本图例
## 数据读取
我一般会使用pandas和os的包读取我的数据。
```
import pandas as pd
import os
import numpy as pd
from collections import Counter               # 基本计数库


# 为方便数据和程序文件的移动，一般采用相对路径，可以使用“../”的形式，但建议使用os.getcwd获取当前程序路径，进行路径拼接

origin_path = os.getcwd()
# 获取当前文件夹下所有的文件的列表
File_list = os.listdir(origin_path)
source_path = os.path.join(origin_path,"data.csv")
```
在数据读取时一般有几种数据格式：csv，xlsx，txt等。
数据也有几种不同的格式：GBK,UTF-8，ISO-8859-1等。
数据也有是否有表头，是否有行标签等。
```
df1 = pd.read_csv(source_path,header = None)        # 无表头的读取方式
df1 = pd.read_excel(source_path,encoding = 'utf-8') # 以utf-8格式存储的数据

# 显示一些数据基本信息
df1.columns                                         # 显示数据列标签
df1.iloc[i]                                         # 以字典形式显示第i行
```
## 数据层面的操作
数据层面的操作更多依赖于pandas和numpy两个库，依赖的数据类型主要是DataFrame，array等。

```
# 分割矩阵
# 数据，等分份数，分割方式axis=0/1 上下/左右
dx,b = np.split(data,2,axis=0)        # 上下均等分离矩阵
dy = b.reset_index(drop=True)         # 对第二个矩阵重新赋index，不然会出现index缺失导致逻辑错误

# 建立一个长度为Nob_dict的全0,int型矩阵
Second_list = np.zeros((len(Nob_dict),len(Nob_dict)),dtype=np.int)
# 为矩阵添加行列标签
Frame_Second = pd.DataFrame(Second_list,columns = list(Nob_dict.values()),index = list(Nob_dict.values()))
```
获取数据标签的集合，使用字典
```
# 获取ID的字典，若不在ID字典当中，则扩充字典。max_dict为最后一个ID的序号，后续序号需要进行+1
Nob_dict = {}
for i in range(len(df1["Gotchi"])):      # Gotchi为计数列
    Nob_dict.update({df1["Gotchi"][i]:i+1})
    max_dict = i+1
```
DataFrame上的批量操作
```
df2 = pd.DataFrame(columns=df1.columns)# columns可选，是否创建列名
df2 = df2.append([dict(df1.iloc[i])],ignore_index=True) # 可以使用append的形式以行添加dict形式的一行数据ignore_index是以行形式添加

boston = pd.concat([features,label],axis =1) # 合并按照列合并数据
```
数据排序
```
sorted(list(boston['feature']),reverse=True) # 排序，以倒叙进行排序
```
计算特征数量
```
Counter(df1['feature']).most_common # 按照数量最多的开始显示，可选参数为个数
```
计算相关系数和对应的显著性
```
r,p_value = stats.pearsonr(boston['feature1'],boston['feature2']) # 计算相关系数和对应的显著性
print('feature1与feature1相关系数为{:.3f},p值为{:.5f}'.format(r,p_value)) # 相关系数保留3位小数，p值保留5位小数
```
透视表格重构
```
buyer_seller = buyer_seller.pivot_table(index='feature', columns='index', values='Value').fillna(0)
```

### 生成网络节点图
```
import networkx as nx

# 计算平均度
edgeNum = 5000    # 边数
nodeNum = 1878    # 节点数
average_degree=edgeNum*2.0/nodeNum
print("平均度："+str(average_degree))
degree_distribute=nx.degree_histogram(buyer_seller_or) # buyer_seller_or关系列
x=range(len(degree_distribute))
y=[z/float(sum(degree_distribute))for z in degree_distribute]
plt.loglog(x,y)
plt.show()
```
参考链接: [Facebook社交网络的特征--基于小世界网络](https://zhuanlan.zhihu.com/p/58594681)\
## 画图

### 绘制散点矩阵图
```
import matplotlib.pyplot as plt
import seaborn as seb
seb.pairplot(data = boston,vars = [feature]) # feature是所选的特征
plt.savefig('scatter fig.png',dpi=500,hue="species")#绘图结果存到本地
```
### 绘制相关系数的热力图
```
import seaborn as seb
r_pearson = boston.corr()
#seb.heatmap(data = r_pearson)
seb.heatmap(data = r_pearson,cmap="YlGnBu")
```

### 绘制散点图
```
plt.scatter([x for x in range(len(boston['feature']))],boston['feature']) #绘制y的曲线
plt.show()
```
### 绘制正太曲线图
```
import matplotlib.mlab as mlab
import seaborn as sns

import warnings
warnings.filterwarnings('ignore') # 不发出警告

sns.set(context='notebook',font='simhei',style='whitegrid')# 设置风格尺度和显示中文
plt.rcParams['axes.unicode_minus']=False # 用来正常显示负号
 
# 直方图
from scipy.stats import norm # 使用直方图和最大似然高斯分布拟合绘制分布

s=np.log(list(boston['feature'])) # 特征数据np.log(list)幂律分布
s=boston['feature']               # 特征数据正太分布

mu =np.mean(s) # 计算均值 
sigma =np.std(s) 
num_bins = len(s) # 直方图柱子的数量 
n, bins, patches = plt.hist(s, num_bins, density=1, facecolor='blue', alpha=0.55,width = 0.0050) 
# 直方图函数，x为x轴的值，normed=1表示为概率密度，即和为一，绿色方块，色深参数0.55，返回n个概率，直方块左边线的x值，及各个方块对象 
y = norm.pdf(bins, mu, sigma)#拟合一条最佳正态分布曲线y
#str='Histogram : $\mu=5.8433$'+str(mu)+',$\sigma=0.8253$';
plt.grid(False) # 无网格
plt.plot(bins, y, 'r--') # 绘制y的曲线 
plt.xlabel('x-label') # 绘制x轴 
plt.ylabel('y-label') #绘制y轴 
plt.title(r'Histogram : $\mu={}$，$\sigma={}$'.format(mu,sigma)) # 在题目中显示mu与sigma

#plt.subplots_adjust(left=0.15) # 左边距 
plt.savefig('Probability fig.png',dpi=1000,bbox_inches = 'tight') # 绘图结果存到本地
plt.show()
```

## 保存数据
```
pd.DataFrame(save_data).to_csv(save_path) # save_data可以是list形式的二维表格
save_date.to_excel(save_path,index = False)# save_data是DataFrame格式的数据# index = False表示没有行标签
```
## 生成关系网络
使用Gephi或者network可以生成关系网络，直接加载两个关系列，然后需要调整K-核心和巨人组件。
## 我的博客主页
[我的博客主页](https://yangli-os.github.io "我的博客主页")
