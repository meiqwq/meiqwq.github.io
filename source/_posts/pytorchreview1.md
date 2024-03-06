---
title: Pytorch巩固1 one-hot 编码
date: 2024-03-06 10:22:34
tags: [pytorch,nlp]
mathjax: true
---
恶补pytorch系列,数据与项目内容来自：[链接](https://www.bilibili.com/video/BV1Ky4y1g7Nk/?p=2),代码是自己写的，和up可能不大一样

# One-hot
将句子分词后，生成一个向量 $\vec{v}$,向量第 $i$维为1当且仅当词$i$出现过。
# 本次任务
数据集是csv文件，包括了单词转化为编码的形式，每个句子的标签（0，1二分类），train/val/test标注。
建立一个NN，本次重心不在backbone上，因此骨架只是一个单词数量映射到2的一个线性层。

# 训练准备
由于每次训练的训练集数据都是批量拿取的，因此我们再复习下Dataset类和Dataloader的使用。


前置条件:

```python
batch_size=200
word_num=8945
```
## Dataset 类
先调库:
```python 
from torch.utils.data import Dataset
```
读入数据:
```python
src=pd.read_csv("data/数字化数据.csv")
```
要继承Dataset类，需要写好构造函数```__init__()```,重写两个函数```__getitem__```与```__len__```。

构造函数可以记录一些基本的数据，记录好表示(比如train/val/test)
```python
    def __init__(self,typ):
        self.typ=typ
        self.data=src[src.part==typ]
```
这里存下了数据作用与对应的dataframe
### get_item
get_item会读入一个参数```idx```,函数要返回第```idx```个数据。

这里同时要对数据完成onehot编码，其中```-1```表示不认识这个词，忽略
```python
    def __getitem__(self, idx):
        ts=torch.zeros(word_num,dtype=torch.float)
        sentence=self.data.iloc[idx]['x']
        seq=[int(x) for x in sentence.split(',')]
        for x in seq:
            if x!=-1:
                ts[x]=1
        tag=int(self.data.iloc[idx]['y'])
        ts_tar=torch.zeros(2,dtype=torch.float)
        ts_tar[tag]=1
        return ts,ts_tar
```
### len
返回数据集大小即可
```python
    def __len__(self):
        return len(self.data)
```

最后产生具体对象
```python
train_set=mydata("train")
```
## Dataloader
调的库在
```python
from torch.utils.data import DataLoader
```
本质是用Dataloader类创建一个对象,几个参数如下:
1. dataset 就是Dataset类构建出来的
2. batch_size 每次取出的个数
3. shuffle最好设为```True```
```python
train_load=DataLoader(dataset=train_set,batch_size=batch_size,shuffle=True)
```
同样的还要构造好```val_load```,```test_load```
# 神经网络骨架
不是这次的重点，只整一个线性层:
```python
from torch import nn
class Mol(nn.Module):
    def __init__(self):
        super().__init__()
        self.mol=nn.Sequential(
            nn.Linear(word_num,2),
        )
    def forward(self, x):
        y_out = self.mol(x)
        return y_out
mynn=Mol()
```
模型单个数据最后会输出一个shape为(2)的tensor，分别表示0类和1类的概率。

# 测试准确率
注意在测试时模型参数不能动，因此要设置
```python
with torch.no_grad():
```
```python
def test_accuracy(data_load):
    with torch.no_grad(): 
        siz=0
        ac=0
        for data in data_load:
            sen,tag=data
            out=mynn(sen)
            for x,y in zip(out,tag):
                if (x[0]>x[1])==(y[0]>y[1]):
                    ac+=1
                siz+=1
        print("准确率为{}".format(ac/siz))
```
# 训练过程
## 工具
损失函数设定为交叉熵，优化器为随机梯度下降，学习速率为```0.01```
```python
loss_fn=nn.CrossEntropyLoss()
    optim=torch.optim.SGD(mynn.parameters(),lr=0.01)
```
训练分多个批次(epoch),每个批次都会把所有数据训练一遍，每次训练先算当前神经网络输出，再算损失函数，然后反向传播梯度下降。
```python
    for epoch in range(10):
        print("训练批次数：{}".format(epoch))
        for data in train_load:
            optim.zero_grad()
            sen,tag=data
            output=mynn(sen)
            res_loss=loss_fn(output,tag)
            res_loss.backward()
            optim.step()
            if train_step%10==0:
                print("训练次数:{},loss={}".format(train_step,res_loss.item()))
            train_step+=1
        test_accuracy(test_load)
        torch.save(mynn.state_dict(),"./model/epoch{}.pth".format(epoch))
```
## 模型保存/读取
这次我只保存了模型参数：
```python
torch.save(mynn.state_dict(),"./model/final.pth")
```
模型读取(要先创建好骨架)：
```python
mynn.load_state_dict(torch.load("./model/final.pth"))
```
## 训练结果
```准确率为0.8693622448979592```

# 完整代码
后续放github上