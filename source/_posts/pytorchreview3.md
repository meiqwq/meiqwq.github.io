---
title: pytorch巩固3 RNN
date: 2024-03-12 17:08:50
tags: [pytorch,nlp]
mathjax: true
---
恶补pytorch系列,数据与项目内容来自：[链接](https://www.bilibili.com/video/BV1Ky4y1g7Nk/?p=2),代码是自己写的，和up可能不大一样

# RNN 网络结构
![图片](rnn.png)

先把句子分词，然后从前往后扫每一个词，每次把当前的词和之前的记忆扔到RNN_cell里。这个结构是合理的，因为它模拟了人的阅读方式。

如果是翻译任务的话只要所有的输出即可.

## 代码
```python
class Mol(nn.Module):# 输入一个(batch_size,词数)的一个tensor,输出(batch_size,18)的tensor
    def __init__(self):
        super().__init__()
        self.emb=nn.Embedding(30,50,0)
        self.rnn=nn.RNNCell(50,100)
        self.fc1=nn.Linear(100,100)
        self.fc2=nn.Linear(100,18)
    def forward(self,x):
        b=x.shape[0]
        out=torch.zeros((b,100))
        embbed=self.emb(x)
        for i in range(15):
            out=self.rnn(embbed[:,i,:,],out)
        out=F.relu(self.fc1(F.dropout(out,0.5)))
        return self.fc2(F.dropout(out,0.5))
```

# 总体实现
这次转换函数在dataset外部实现了，所以还是放一下全部代码
```python
import numpy as np
import pandas as pd
import torch
from torch.utils.data import Dataset
from torch.utils.data import DataLoader
from torch import nn
from os.path import exists
import torch.nn.functional as F

src=pd.read_csv("./data/data.csv")

class mydata(Dataset):
    def __init__(self,typ):
        self.data=src[src.part==typ]
        self.typ=typ
    def __getitem__(self, idx):
        return self.data.iloc[idx]['x'],self.data.iloc[idx]['y']
    def __len__(self):
        return len(self.data)

def to_tensor(data):
    N=len(data)
    t1=np.zeros((N,15))
    t2=np.zeros((N))
    for i in range(N):
        x,y=data[i]
        x=x.split(',')+[0]*15
        x=x[:15]
        x=[int(xx) for xx in x]
        t1[i]=x
        t2[i]=int(y)
    return torch.LongTensor(t1),torch.LongTensor(t2)
train_set=mydata("train")
train_load=DataLoader(dataset=train_set,batch_size=100,shuffle=True,drop_last=True,collate_fn=to_tensor)

test_set=mydata("test")
test_load=DataLoader(dataset=test_set,batch_size=100,shuffle=True,drop_last=True,collate_fn=to_tensor)

val_set=mydata("val")
val_load=DataLoader(dataset=val_set,batch_size=100,shuffle=True,drop_last=True,collate_fn=to_tensor)

class Mol(nn.Module):
    def __init__(self):
        super().__init__()
        self.emb=nn.Embedding(30,50,0)
        self.rnn=nn.RNNCell(50,100)
        self.fc1=nn.Linear(100,100)
        self.fc2=nn.Linear(100,18)
    def forward(self,x):
        b=x.shape[0]
        out=torch.zeros((b,100))
        embbed=self.emb(x)
        for i in range(15):
            out=self.rnn(embbed[:,i,:,],out)
        out=F.relu(self.fc1(F.dropout(out,0.5)))
        return self.fc2(F.dropout(out,0.5))
mynn=Mol()

def test_accuracy(data_load):
    with torch.no_grad():
        siz=0
        ac=0
        for data in data_load:
            sen,tag=data
            output=mynn(sen)
            for x,y in zip(output,tag):
                x=x.argmax(dim=0)
                if x==y:
                    ac+=1
                siz+=1
        print("准确率:{:.3f}".format(ac/siz))

def train_model():
    epoch=0
    train_step=0
    loss_fn=nn.CrossEntropyLoss()
    optim=torch.optim.Adam(mynn.parameters(), lr=1e-3)

    for epoch in range(30):
        print("批次:{}".format(epoch))
        for data in train_load:
            optim.zero_grad()
            sen,tag=data
            output=mynn(sen)
            res_loss=loss_fn(output,tag)
            res_loss.backward()
            optim.step()
            train_step+=1
            if train_step%10==0:
                print("训练次数:{},loss:{}".format(train_step,res_loss))
        test_accuracy(test_load)
        torch.save(mynn.state_dict(),"./model/epoch_{}.pth".format(epoch))
    torch.save(mynn.state_dict(),"./model/final.pth")
if not exists("./model/final.pth"):
    train_model()
else:
    mynn.load_state_dict(torch.load("./model/final.pth"))
test_accuracy(val_load)
```
输出:
```准确率:0.692```