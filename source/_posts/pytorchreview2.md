---
title: Pytorch巩固2 CNN
date: 2024-03-07 23:52:44
tags: [pytorch,nlp]
mathjax: true
---
恶补pytorch系列,数据与项目内容来自：[链接](https://www.bilibili.com/video/BV1Ky4y1g7Nk/?p=2),代码是自己写的，和up可能不大一样

这次任务是个18分类的问题，
除了网络结构不一样，其它和上一个基本一致.
只贴代码了

```python
import numpy as np
import pandas as pd
import torch
from torch.utils.data import Dataset
from torch.utils.data import DataLoader
from torch import nn
from os.path import exists
batch_size=100
word_cnt=29

src=pd.read_csv("./data/data.csv")


class mydata(Dataset):
    def __init__(self,typ):
        self.data=src[src.part==typ]
        self.typ=typ
    def __getitem__(self, idx):
        sen=[int(x) for x in self.data.iloc[idx]['x'].split(',')]
        oht=np.zeros((15,word_cnt))
        for i in range(min(len(sen),15)):
            oht[i,sen[i]-1]=1
        return torch.FloatTensor(oht),int(self.data.iloc[idx]['y'])
    def __len__(self):
        return len(self.data)
train_set=mydata("train")
train_load=DataLoader(dataset=train_set,batch_size=batch_size,shuffle=True)

test_set=mydata("test")
test_load=DataLoader(dataset=test_set,batch_size=batch_size,shuffle=True)

val_set=mydata("val")
val_load=DataLoader(dataset=val_set,batch_size=batch_size,shuffle=True)

class Mol(nn.Module):
    def __init__(self):
        super().__init__()
        self.h=50
        self.mol=nn.Sequential(
            nn.Conv1d(15,self.h,5,2),nn.ELU(),
            nn.Conv1d(self.h,self.h,5,2),nn.ELU(),
            nn.Conv1d(self.h,self.h,5,1),nn.ELU(),
        )
        self.lin=nn.Linear(self.h,18)
    def forward(self,x):
        y1=self.mol(x).squeeze(dim=2)
        return self.lin(y1)
mynn=Mol()

def test_accuracy(data_load):
    with torch.no_grad():
        siz=0
        ac=0
        for data in data_load:
            sen,tag=data
            out=mynn(sen)
            for x,y in zip(out,tag):
                x=x.argmax(dim=0)
                siz+=1
                if x==y:
                    ac+=1
        print("准确率为{:f}".format(ac/siz))
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

输出:```准确率为0.707317```