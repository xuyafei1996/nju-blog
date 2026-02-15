---
title: Transformer：现代 AI 的基石与意义矩阵的编解码艺术
date: 2026-02-16 15:10:00
tags:
  - Transformer
  - AI
  - Deep Learning
  - Architecture
categories:
  - AI Theory
  - Engineering
---

## TL;DR
- **核心地位**：Transformer 取代 RNN/CNN 成为 NLP 及多模态领域的统一架构，是现代大模型 (LLM) 的物理地基。
- **架构精髓**：基于自注意力机制 (Self-Attention) 的 Encoder-Decoder 结构，实现并行计算与长距离依赖捕捉。
- **演进分野**：分化为 Encoder-only (BERT)、Decoder-only (GPT) 与 Encoder-Decoder (T5) 三大流派，分别专攻理解、生成与翻译。
- **哲学隐喻**：人生即编解码——前半生 Encoder 压缩世界为含义矩阵，后半生 Decoder 向世界输出价值。

<!-- more -->

## 概念与痛点
### 它是什么
Transformer 是 Google 团队在 2017 年论文《Attention Is All You Need》中提出的深度学习架构。它完全抛弃了循环与卷积，纯粹依赖注意力机制来处理序列数据。

### 解决什么问题
- **串行计算瓶颈**：RNN/LSTM 必须按时间步顺序计算，无法利用 GPU 并行加速，训练极慢。
- **长距离遗忘**：序列过长时，早期信息在传递中衰减，无法捕捉“上下文”。
- **信息瓶颈**：Seq2Seq 模型将整句压缩为一个定长向量，信息有损。

## 发展历史与演进
1.  **RNN/LSTM 时代**：序列建模的统治者，但受限于串行与梯度消失。
2.  **Attention 引入**：Bahdanau Attention 让模型能“聚焦”原文不同部分，但仍挂载于 RNN 上。
3.  **Transformer 诞生 (2017)**：彻底移除 RNN，提出 Multi-Head Attention，开启并行训练新纪元。
4.  **百花齐放 (2018-2020)**：
    - **BERT (Encoder-only)**：双向理解，横扫 NLP 榜单。
    - **GPT (Decoder-only)**：单向生成，暴力美学初现。
    - **T5/BART (Encoder-Decoder)**：统一文本到文本任务。
5.  **大模型时代 (2020+)**：Decoder-only 架构因其训练稳定性与涌现能力，成为 ChatGPT 等 LLM 的首选。

## 核心模块
![Transformer 架构图](../img/illustration/transformer.png)
1.  **Encoder (编码器)**：
    - **作用**：负责“理解”。将输入序列转化为富含语义的上下文向量 (Context Vector)。
    - **结构**：Self-Attention + Feed Forward Network (FFN)，层层堆叠，双向可见。
2.  **Decoder (解码器)**：
    - **作用**：负责“生成”。根据上下文向量与已生成的 Token，预测下一个 Token。
    - **结构**：Masked Self-Attention (防剧透) + Cross-Attention (看 Encoder) + FFN。
3.  **Self-Attention (自注意力)**：
    - 核心公式：$Attention(Q, K, V) = softmax(\frac{QK^T}{\sqrt{d_k}})V$
    - 让每个词都能关注句中其他所有词，计算相关性权重。

## 运行机制与工程实践
### 独立工作模式
Transformer 的 Encoder 和 Decoder 既可合体 (如机器翻译)，亦可独立门户：

1.  **Encoder-only (理解流)**：
    - **代表**：BERT, RoBERTa。
    - **训练**：Masked Language Modeling (完形填空)。随机遮盖词，让模型猜。
    - **应用**：情感分析、文本分类、命名实体识别。
    - ![Encoder-only 架构图](../img/illustration/encode-only.png)
2.  **Decoder-only (生成流)**：
    - **代表**：GPT 系列, LLaMA。
    - **训练**：Causal Language Modeling (文本接龙)。根据上文预测下文。
    - **应用**：对话生成、代码补全、创意写作。
    - ![Decoder-only 架构图](../img/illustration/decode-only.png)
3.  **Encoder-Decoder (互译流)**：
    - **代表**：T5, BART。
    - **应用**：机器翻译、文本摘要。

### 运算流程与数学基础
1.  **Embedding**：将离散单词映射为连续向量空间的点 (线性代数基底)。
2.  **Positional Encoding**：注入位置信息 (正弦/余弦函数)，弥补无序性。
3.  **QKV 计算**：
    - 输入向量 $X$ 乘以三个权重矩阵 $W_Q, W_K, W_V$ (线性变换)。
    - $Q \times K^T$：计算向量夹角 (点积)，衡量相似度/相关性 (线代+几何)。
    - $Softmax$：将相似度归一化为概率分布 (概率论)。
    - $\times V$：加权求和，提取信息。
4.  **FFN**：非线性变换，增加模型表达能力。

### 监督 vs 自监督
- **监督学习**：有标签数据 (Input: "猫", Label: "Cat")。成本高，数据难得。
- **自监督学习 (Self-Supervised)**：无标签，自己挖坑自己填。
    - 无论是 BERT 的挖词填空，还是 GPT 的预测下一个词，本质都是利用海量文本本身作为监督信号。这是 AI 爆发的**核动力**。

## 高级特性：生成与创造性
### 输入输出成本差异
为何 Output Token 比 Input Token 贵/慢？
- **Input (Prefill)**：一次性并行计算所有 Token 的 KV Cache。GPU 利用率高。
- **Output (Decoding)**：**自回归 (Auto-regressive)** 生成。每生成一个词，都要将其加入输入再次前向传播。串行过程，受限于内存带宽 (Memory Bandwidth Bound)。

### 创造性的调节
模型输出本质是概率分布采样。
1.  **Temperature (温度)**：
    - $T < 1$ (低温)：概率分布变尖锐，只选大概率词。输出保守、准确、逻辑严密。
    - $T > 1$ (高温)：概率分布变平缓，低概率词也有机会被选中。输出多样、发散、有创意。
2.  **Top-k / Top-p**：
    - **Top-k**：仅在概率最高的 k 个词中采样，截断长尾垃圾词。
    - **Top-p (Nucleus)**：按概率累加直到 p (如 0.9)，动态调整候选池大小。
