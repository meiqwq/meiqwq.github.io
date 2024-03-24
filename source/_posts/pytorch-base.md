---
title: Pytorch 大杂烩
date: 2024-03-23 20:39:01
tags: [nlp,pytorch]
mathjax: true
---

# tensor

## flip
对某一个维度进行反转:
```python
torch.flip(input,dims)
```
返回反转完的tensor

dims是一个list or tuple

```python
t=torch.LongTensor([[2,3,4],[5,6,7],[1,2,3]])
print(t)
```
输出
```
tensor([[2, 3, 4],
        [5, 6, 7],
        [1, 2, 3]])
```
再执行
```python
t=torch.flip(t,dims=(1,))
print(t)
```
输出
```
tensor([[4, 3, 2],
        [7, 6, 5],
        [3, 2, 1]])
```
## sum
```python
t.sum()
```
可以直接返回t所有元素的和（以tensor的形式）。

当然还有别的用法
```python
t=torch.LongTensor([[2,3,4],[5,6,7],[1,2,3]])
print(torch.sum(t,0))
```
输出
```
tensor([ 8, 11, 14])
```
对第dim维求和，返回tensor（少了第dim维）

## cat
```python
torch.cat(tensors, dim=0, *, out=None) → Tensor
```
其中 tensors是tuple或者list 在dim维度拼接（也就是说第dim维度的东西会增多）

```python
t1=torch.LongTensor([[2,3,4],[5,6,7],[1,2,3]])
t2=torch.LongTensor([[1,1,4],[5,1,4],[1,9,1]])
print(torch.cat([t1,t2],dim=1))
```
输出
```
tensor([[2, 3, 4, 1, 1, 4],
        [5, 6, 7, 5, 1, 4],
        [1, 2, 3, 1, 9, 1]])
```
归纳，俩shape分别为$(a_1,a_2,...x_d,...a_n),(a_1,a_2,...y_d,...a_n)$的tensor在```torch.cat(dim=d)```后将会得到shape为$(a_1,a_2,...x_d+y_d,...a_n)$

## stack
```python
torch.stack(tensors, dim=0, *, out=None) → Tensor
```
其中tensors是列表或元组，在第dim维度堆叠
```python
t1=torch.LongTensor([[2,3,4],[5,6,7],[1,2,3]])
t2=torch.LongTensor([[1,1,4],[5,1,4],[1,9,1]])
print(torch.stack([t1,t2],dim=1))
```
输出
```
tensor([[[2, 3, 4],
         [1, 1, 4]],

        [[5, 6, 7],
         [5, 1, 4]],

        [[1, 2, 3],
         [1, 9, 1]]])
```

堆叠的所有tensor的shape一定要一致

归纳：将$k$个tensor堆叠后，他们的shape将会在dim维多出一个$k$.