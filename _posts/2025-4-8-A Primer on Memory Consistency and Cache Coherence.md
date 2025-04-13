---
title: A Primer on Memory Consistency and Cache Coherence
author: JYC
description: Note about A Primer on Memory Consistency and Cache Coherence
date: 2025-4-7 19:12:30 
categories: [Digital Circuit, Notes about book]
tags: [memory consistency,  cache coherence]     # TAG names should always be lowercase
--- 


# Definition of Coherence invariants

## Single-Writer, Multiple-Read (SWMR) Invariant.

对于任何内存位置，在任意指定的时间，只能存在一个core对于内存进行读写或者多个core对于内存进行只读。

## Data-Value Invariant

对于一个内存的值应该从第一个epoch到read-write的epoch保持相同 

## Another Coherence invariants

- every store is visible to all cores
- writes to the same memory location are serialized 
- Hennessy and Patterson 还提出了一种关于 Coherence invariants 的定义，具体可以见书P14


# The Granularity of Coherence

一个core正常访问内存的细粒度可以从1B到64B，但是在实际应用中，内存一致性的维护在cache-block的细粒度
