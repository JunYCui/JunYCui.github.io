---
title: Transformer 
author: JYC
description: study about transformer algorithm
date: 2025-1-10 18:43:30 
categories: [Study, Machine Learning]
tags: [ai, NLP]     # TAG names should always be lowercase
math: true
comments: true
--- 

## Transformer 输入部分

Transformer 的输入主要由文本嵌入层和位置编码器组成，下面将分别介绍

### 文本嵌入层

文本嵌入层的主要的作用是对于文本进行升维，将文本通过向量进行表示，希望在高维空间捕获词汇间的关系。\
>文本嵌入层有两个属性分别是`vocab`和`d_model`，前者描绘词表的大小，后者描绘维度的大小。

代码的实现
```python
class Embeddings(nn.Module):
    def __init__(self,d_model,vocab):
        # d_model: 词嵌入的唯独
        # vocab: 词表的大小
        super(Embeddings, self).__init__()
        # 定义Embedding 层
        self.lut = nn.Embedding(vocab,d_model)
        # 将参数传入类中
        self.d_model = d_model

    def forward(self,x):
        return  self.lut(x)*math.sqrt(self.d_model)
```

> `sqrt(self.d_model)`的作用，有助于梯度的快速变化{.prompt-tip}

### 位置编码器
位置编码器的作用顾名思义是对于每个词汇的位置进行编码，很直觉的一种解释是不同的词汇在不同的地方有着不同的含义。\
对于位置编码的公式
$$
PE_{(pos,2i)} = sin(pos/10000^{2i/d_{model}}) \\
PE_{(pos,2i+1)} = cos(pos/10000^{2i/d_{model}})
$$
> 选择这个公式的原因：这样更容易学习位置的相对关系，每个PE之间都可以线性表示 {.prompt-tip}

代码实现
``` python
class PositionalEncoding(nn.Module):
    def __init__(self, d_model, dropout, max_len=5000):
        #d_model：词嵌入的维数
        #dropout：置0比例
        #max_len：每个句子的最大维度
        super(PositionalEncoding, self).__init__()

        # 定义Dropout层
        self.dropout = nn.Dropout(p = dropout)

        #初始化一个位置编码矩阵
        pe = torch.zeros(max_len,d_model)

        #初始化一个绝对位置矩阵
        position = torch.arange(0, max_len).unsqueeze(1)

        #对于位置参数进行缩放，有助于梯度下降过程中更快的下降
        div_term =  torch.exp(torch.arange(0,d_model,2)*
                              -(math.log(10000.0)/d_model))
        pe[:,0::2] = torch.sin(position * div_term)
        pe[:,1::2] = torch.cos(position * div_term)

        #使用unsqueeze 拓展维度
        pe = pe.unsqueeze(0)

        #把pe位置编码注册成buffer（不随优化参数而改变）
        self.register_buffer('pe',pe)

    def forward(self, x):
        #max_len维度太长，对于x进行一个适配工作
        #x.size(1)表示 pe 的长度与输入的长度相同
        #requires_grad = False 表示不进行梯度优化
        x = x + Variable(self.pe[:,:x.size(1)],requires_grad=False)
        return  self.dropout(x)
```


