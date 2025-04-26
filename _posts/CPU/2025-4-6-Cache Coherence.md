---
title: Cache Coherence
author: JYC
description: Cache Coherence implement and MESI Protocol
date: 2025-4-5 19:12:30 
categories: [Digital Circuit, Protocol]
tags: [mutiprocessor, protocol]     # TAG names should always be lowercase
--- 

## Cache Coherence 介绍

![Cache ARCH](../assets/img/Cache%20Coherence/implement.png)

为了实现多核一致性，我们假设一种存储架构——Cache和LLC(Last Level Cache)以及一致性控制器(Coherence Controller)，其本质是一个有限状态机，这些状态机的交互由一致性协议指定，一致性控制器有着许多的职能，在Cache端的Controller，我们称作缓存控制器（Cache Controller），其主要处理两个source的请求，在core端，缓存控制器接收来自核的Load和Store的请求，返回Load Values。缓存的缺失一定会导致控制器通过发出一致性请求（因为这个block将包含这个cpu访存的地址）来初始化一个一致性事务。这个一致性请求被发送到互联网络被其他一致性控制器接收，这个事物包含一个request和其他信息来满足这个请求。作为每个事务的一部分发送的事务类型和消息取决于具体的一致性协议。在Memory\LLC端的一致性控制器称为内存控制器（Cache Contorller）。\
每一个一致性控制器维持对于每一个block块维持着独立且相同的有限状态机，每一个block的状态转换只取决于block当前的状态。

## Snooping vs Directory

主要有两种主要的一致性协议-Snooping和Derictory
- Snooping \
  缓存控制器通过向所有其他一致性控制器广播请求消息来发起对块的请求。一致性控制器集体“做正确的事”，例如，如果它们是数据的所有者，则响应其他核心的请求发送数据。窥探协议依赖于互连网络以一致的顺序将广播消息传递给所有核心。大多数窥探协议假设请求按顺序到达，例如通过共享总线，但更先进的互连网络和松散的顺序也是可能的。
- Derictory \
  缓存控制器通过单播（unicast）向管理该数据块的主内存控制器（home memory controller）发起请求。每个内存控制器维护一个目录（directory），记录末级缓存（LLC）/内存中每个块的状态信息，例如当前所有者（owner）或当前共享者（sharers）的身份。当请求到达主内存控制器时，控制器会查询该块的目录状态：\
  若请求是GetS（获取共享副本）：

   - 内存控制器检查目录以确定所有者。
   - 如果LLC/内存是所有者，则直接向请求者返回数据响应，完成事务。
   - 如果某个缓存控制器是所有者，则内存控制器将请求转发给该所有者缓存；所有者缓存收到转发请求后，向请求者发送数据响应以完成事务。



## MESI

### 介绍
MESI协议是一种广泛使用的缓存一致性协议，用于确保多核处理器系统中各个核心的缓存数据保持一致。它通过四种状态（Modified、Exclusive、Shared、Invalid）和总线嗅探机制，管理缓存行的状态转换，从而避免数据不一致问题。\
[模拟MESI协议的网址](https://www.scss.tcd.ie/Jeremy.Jones/VivioJS/caches/MESI.htm)

### 状态解析
M - Modified表示该数据已被更改，此时缓存内的数据与内存中不一致\
E - Exclusive表示该数据仅被当前缓存包含，其他缓存并不包含\
S - Shared表示该数据被多个缓存共享。\
I - Invalid表示当前缓存中的数据无效  


### 总线嗅探机制

CPU感知其他CPU的行为（比如读、写某个缓存行）就是是通过嗅探（Snoop）线性中其他CPU发出的请求消息完成的，有时CPU也需要针对总线中的某些请求消息进行响应。这被称为"总线嗅探机制"。这些消息类型分为请求消息和响应消息两大类，细分为6小类。

| 消息类型               | 请求/响应 | 消息描述                                                                                                                                             |
| ---------------------- | --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| Read                   | 请求      | 通知其他处理器将要读某个地址，该消息包含数据的地址                                                                                                   |
| Read Response          | 响应      | 该消息内包含了被请求读取的数据。该消息可能是主内存返回的，也可能是其他高速缓存嗅探到Read 消息返回的                                                  |
| Invalid                | 请求      | 通知其他处理器删除指定内存地址的数据副本                                                                                                             |
| Invalidate Acknowledge | 响应      | 接收到Invalidate消息的处理器必须回复此消息，表示已经删除了其高速缓存内对应的数据副本                                                                 |
| Read Invalidate        | 请求      | 此消息为Read 和 Invalidate消息组成的复合消息，主要是用于通知其他处理器当前处理器准备更新一个数据了，并请求其他处理器删除其高速缓存内对应的数据副本。 |
| WriteBack              | 响应      | 消息包含了需要写入内存的数据和其对应的内存地址                                                                                                       |

### Read Invalidate执行过程
总线嗅探（Bus Snooping）中的 Read Invalidate（读-无效）消息 是MESI协议中用于**强制获取数据独占权**的一种关键总线事务。它的核心目的是允许某个缓存核心在修改数据前，同时完成两个操作：**Read(确保拥有最新值)** 和 **Invalidate(确保独占写入权)**
#### 触发条件
当一个核心（例如Core A）需要对某个缓存行执行写入操作，但当前本地缓存的状态可能是**Invalid**和**Shared**两种状态，此时，CoreA需通过总线发送 Read Invalidate 消息，以达成以下目标：
- 获取最新数据（若其他核心的缓存中存在Modified状态数据）；
- 其他所有缓存中该缓存行的状态必须变为Invalid（失去使用权）。

#### 执行流程
假设Core A需要写入地址X的数据，其本地缓存的X行状态为Shared或Invalid：
1. 发送Read Invalidate消息
2. 总线监听（Snooping）\
    所有其他核心（如Core B/C/D）监听总线，接收到这个消息后，检查自身是否有地址X的缓存行：
    - 若有Modified状态：\
持有Modified状态的Core（例如Core B）需要将数据写回主存（或直接传给Core A），然后将自身缓存行状态降为Invalid。
   - 若有Shared或Exclusive状态：\
直接将该缓存行标记为Invalid（无需写回主存）。
3. Core A接收数据：
   - 如果某个核心（如Core B）响应并提供了最新的Modified数据，Core A将数据载入缓存。
   - 如果无核心响应，Core A从主存读取数据。
4. 更新状态 \
   Core A将缓存行置为Modified（独占且可修改），其他所有核心对该行的状态均为Invalid。

#### 与普通Read的区别
- 普通Read请求：
仅请求数据，缓存行进入Shared或Exclusive状态，允许其他核心继续共享。
- Read Invalidate：
请求数据的同时强制其他核心的副本失效，保证请求者后续的写入操作可直接进行，而无需再次协调其他缓存。

### 状态转移图

![状态转移图](../assets/img/Cache%20Coherence/State.png)

### 状态表

![状态表](../assets/img/Cache%20Coherence/table.png)

## MESI引入的问题

缓存的一致性消息传递是要时间的，这就使其切换时会产生延迟。当一个缓存被切换状态时其他缓存收到消息完成各自的切换并且发出回应消息这么一长串的时间中CPU都会等待所有缓存响应完成。可能出现的阻塞都会导致各种各样的性能问题和稳定性问题。

### 解决的方法

- Store Buffers  
  为了避免这种CPU运算能力的浪费，Store Bufferes被引入使用。处理器把它想要写入到主存的值写到缓存，然后继续去处理其他事情。当所有失效确认（Invalidate Acknowledge）都接收到时，数据才会最终被提交。
- 使用Store Buffers也带来了风险。
  1. 就是处理器会尝试从存储缓存（Store buffer）中读取值，但它还没有进行提交。这个的解决方案称为Store Forwarding，它使得加载的时候，如果存储缓存中存在，则进行返回。
  2. 不能保证什么时候完成，需要使用内存屏障。

- Invalidate Buffers  
因为进行store操作时要发送Invalid事务，此时要等待Invalidate Acknowledge才能对于缓存进行写入，所以为了提高效率并不一定要等到相应缓存置为无效时才返回Invalidate Acknowledge，此时要对于Invalidate事务进行缓存

## 参考文献
[并发吹剑录（一）：CPU缓存一致性协议MESI](https://zhuanlan.zhihu.com/p/351550104)\
[全知乎最详细的并发研究之CPU缓存一致性协议(MESI)有这一篇就够了！](https://zhuanlan.zhihu.com/p/467782159)\
《A Primer on Memory Consistency and Cache Coherence》