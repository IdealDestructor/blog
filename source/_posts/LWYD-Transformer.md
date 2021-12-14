---
title: 论文阅读笔记之Attention Is All You Need 
date: 2021-12-02 14:22:35
tags: [深度学习,Transformer]
categories: 论文阅读
widgets: null
---

转载自https://www.bilibili.com/read/cv13759416?spm_id_from=333.788.b_636f6d6d656e74.52

<!--more-->

Transformer论文逐段精读【论文精读】

**1. 标题 + 作者**

Transformer 开创了继 MLP 、CNN 和 RN 之后的第四大类模型。200 页综述（来自评论区： https://arxiv.org/pdf/2108.07258.pdf ）建议将 Transformer 作为基础模型。

**标题**：XXX is all you need. 头条标题。

Attention is all you need. 英文语法正确，集中注意力。

**作者**：8 个共同一作 *，排序随机、贡献相等，推荐在论文中阐述作者的具体贡献。

**2. 摘要**

03:22

sequence transduction: 序列转录，序列到序列的生成。input 一个序列，output 一个序列。e.g. 机器翻译：输入一句中文，输出一句英文。

**第 1 句：主流的序列转录模型**

包括一个 encoder 和 一个 decoder 的 RNN 或者 CNN 架构。

**第 2 句：表现好的序列转录模型：用了 attention**

表现好的 sequence transduction 模型在 encoder 和 docoder 之间使用了 attention。

**第 3 句：本文提出 基于 attention 的 Transformer**

Transformer 变相金刚 的贡献：简单 simple (褒义词)，跟之前的（表现好的）循环 or 卷积架构不一样。

**第 4 句：实验总结 - 并行化、更少时间训练**

**第 5-6 句：实验 BLEU 提分介绍**

2 个机器翻译任务的实验结果

*   英 - 德：提高 2 BLEU
*   英 - 法：SOTA，41.8 BLEU，只需 8GPUs 的 3.5 天的训练

**第 7 句：能很好的泛化到其他任务**

本文是从机器翻译的角度写的，后续 图片、视频 transformer 出圈了。

**3. 结论**

08:17

**第 1 句：**介绍了 Transformer 模型，第一个仅仅使用注意力、做序列转录的模型，把之前在 encoder - decoder 的结构换成了 multi-headed self-attention.

**第 2 句：机器翻译任务**

SOTA，比其他结构训练快

**第 3 句：纯注意力的模型其他任务的应用**

图片、音频、视频；使生成不那么时序化 less sequential

代码：本文在结论部分，目前推荐放在摘要最后，方便了解论文详细内容和复现。

**4. 导言**

10:06

摘要（1-3 句）的扩充。

sequence model and transduction 问题：language modeling, machine translation

2017 年常用方法是 RNN, LSTM, GRU.

语言模型、编码器 - 解码器架构

RNN 特点（缺点）：从左往右一步一步计算，对第 t 个状态 ht，由 ht-1（历史信息）和 当前词 t 计算。

*   难以并行。e.g. 100 个词要算 100 步
*   过早的历史信息可能被丢掉。时序信息是一步一步往后传递的，e.g. 时序长的时候
*   一个大的 ht 存历史信息。每一个 计算步都需要存储，内存开销大

最近的工作通过 factorization 分解 tricks 和 conditional computation 并行化来提升计算效率，但 sequential computation 的问题本质依然存在。

attention 在 RNN 上的应用: attention 用在怎么把 encoder 的信息有效的传给 decoder，允许建模 input or output sequence 与距离无关的 dependencies

本文 Transformer 网络不再使用循环结构、**纯 attention**、并行度高、较短时间达到很好的效果 (8 P100 GPU 12 hours)。

导言较短，8 页内容尽可能涵盖提出的新内容。

**5. 相关工作**

14:36

**3 联系**：**CNN**（局部像素 --> 全部像素；多通道 --> multi-head），**Self-attention** 他人提出和应用，**Memory network** 使用 recurrent attention mechanism 而不是 sequence-aligned recurrence.

**1 区别：**Transformer **仅依赖 self-attention** 计算输入输出的表征，没有使用 sequence-aligned RNNs or convolution.

**CNN：**

(cons) CNN 替换 RNN 来减少时序的计算，但 CNN 对较长的序列难以建模。因为卷积计算的时候看一个比较小的窗口，i.e., 3 * 3 窗口，如果 2 个像素隔得比较远，需要用很多 3 * 3 的卷积层、一层一层的叠加上去，才能把隔得很远的 2 个像素联系起来。

Transformer 的 attention mechanism 每一次看到所有的像素，一层能够看到整个序列。

(pros) 多个输出通道，每个通道可以识别不同的模式。

Transformer 的 multi-head self-attention 模拟 CNNs 多通道输出的效果。

**Self-attention 别人提出， 17 年 memory nework** 也是一个研究热点，不知道可跳过。

区别：**fiirst transduction model relying entirely on self-attention** to compute representations of its input and output without using sequencealigned RNNs or convolution.

background 章节：跟你论文**相关的是谁**？跟你的**联系与区别**

**6. 模型**

最重要的章节

16:35

**第一段： encoder-decoder 架构 + auto-regressive in decoder**

sequence transduction models 比较好的结构是 encoder-decoder

encoder 将 （x1, x2, ... , xn）（原始输入） 映射成 （z1, z2, ..., zn）（机器学习可以理解的向量）

i.e., 一个句子有 n 个词，xt 是第 t 个词，zt 是第 t 个词的向量表示。

decoder 拿到 encoder 的输出，会生成一个长为 m 的序列（y1, y2, ... , ym）

n 和 m 可以一样长、可以不一样长。

i.e., 中英互译：Hello World 你好世界

![](https://i.loli.net/2021/12/02/iN5goJwLSQB3TfU.png)

encoder 和 decoder 的区别：decoder 的输出词是一个一个生成的，auto-regressive 的模型

encoder 一次性很可能看全整个句子。i.e., 翻译的时候，看到整句英语：Hello World

decoder 在解码的时候，只能一个一个的生成。 auto-regressive，输入又是你的输出。

i.e., 给定 z 向量 (z1, ..., zn) 生成 y1，在得到 y1 之后可以生成 y2。在生成 yt 的时候，要把之前的 y1 到 yt-1 都拿到。在翻译的时候，一个词一个词往外蹦。

过去时刻的输出会作为你当前时刻的输入，自回归 auto-regressive。

**第二段：Transformer 使用了 encoder-decoder 架构**

堆叠的 stacked self-attention and point-wise, fully-connected layers，展示在 图 1

写论文：全局图、一张图解释所有

20:00

![](https://i0.hdslb.com/bfs/note/c5ee2a5333abe62140542cbf15a88d51ac7e5a49.png)

i.e., 中 译 英

Inputs: 中文句子

Outputs: decoder 在做预测的时候 是没有输入的。**Shifted right** 指的是 decoder 在之前时刻的一些输出，作为此时的输入。一个一个往右移。

**Inputs ---- Input Embedding**

输入经过一个 Embedding 层， i.e., 一个词进来之后表示成一个向量。得到的向量值和 Positional Encoding （3.5）相加。

**Encoder 的核心架构**

Nx：N 个 Transformer 的 block 叠在一起。

i.e., ResNet 中 N 个残差块 的叠加。

**Transformer 的 block**

Multi-Head attention

Add & Norm: 残差连接 + Layernorm

Feed Forward: 前馈神经网络 MLP

21:42

encoder 的输出 作为 decoder 的输入

![](https://i0.hdslb.com/bfs/note/6e1642abf0d7c90ce53cadb1fee42444aa00f58a.png)

decoder 和 encoder 有一点像，图中红框部分是相同的

![](https://i0.hdslb.com/bfs/note/328f4e70d3f6781dabd89e187947cea68f0bb082.png)

decoder 多了一个 Masked Multi-Head Attention

![](https://i0.hdslb.com/bfs/note/72f3d8b7ba34b37fdf1a75b7c34ae5b2c36da848.png)

decoder 是 encoder 相同部分 和 Masked Multi-Head Attention 组成一个块，重复 Nx 次

![](https://i0.hdslb.com/bfs/note/2aee098549d6df77ba93337e24f5a8be32d92868.png)

decoder 的输出进入一个 Linear 层，做一个 softmax，得到输出。

Linear + softmax: 一个标准的神经网络的做法

![](https://i0.hdslb.com/bfs/note/274137b72c79654dcda9b7469efd45280ead6018.png)

总结：Transformer 是一个比较标准的 encoder - decoder 架构。区别：encoder、decoder 内部结构不同，encoder 的输出 如何作为 decoder 的输入有一些不一样。

![](https://i0.hdslb.com/bfs/note/9c23ecc3a853a26df9f0477c4fef0f3b24a29b00.png)

3.1 Encoder and Decoder Stacks

22:38

Encoder 结构：重复 6 个图中红色的 layer

![](https://i0.hdslb.com/bfs/note/7794add66f2d7de63c3e93f97bf449ebc892d746.png)

每个 layer 有 2 个 sub-layers。

*   第一个 sub-layer 是 multi-head self-attention
*   第二个 sub-layer 是 simple, position-wise fully connected feed-forward network, 简称 MLP

23:33

每个 sub-layer 的输出做 残差连接 和 LayerNorm

公式：LayerNorm(x + Sublayer(x) )

Sublayer(x) 指 self-attention 或者 MLP

residual connections 需要输入输出维度一致，不一致需要做投影。简单起见，固定 每一层的输出维度 dmodel = 512

简单设计：只需调 2 个参数 dmodel 每层维度有多大 和 N 多少层，影响后续一系列网络的设计，BERT、GPT。

Remark：和 CNN、MLP 不一样。MLP 通常空间维度往下减；CNN 空间维度往下减，channel 维度往上拉。

25:07

LayerNorm

写作：不要假设读者都知道所有的细节。可以的话，花几句话讲清楚内容

LayerNorm 和 BatchNorm 的画图对比

25:41

BatchNorm 简单的 2 维 情况

每一行是一个样本 X，每一列是 一个 feature

BatchNorm：每次把一列（1 个 feature）放在一个 mini-batch 里，均值变成 0， 方差变成 1 的标准化。

![](https://i0.hdslb.com/bfs/note/137d2befec9350a5cddeaf2591eff2af142c170e.png)

How：（该列向量 - mini-batch 该列向量的均值）/（mini - batch 该列向量的方差）

训练时：mini-batch 计算均值；

测试时：使用 全局 均值、方差。

BatchNorm 还会学 lambda1 beta，BatchNorm 可以通过学习将向量 放缩成 **任意均值、任意方差** 的一个向量。

Layernorm 画图示例

27:04

LayerNorm 跟 BatchNorm 在很多时候几乎是一样的，除了实现的方法有点不一样之外。

二维输入：

LayerNorm：对每个样本做 Normalization（把每一行变成 均值为 0、方差为 1），不是对每个特征做 normalization。

![](https://i0.hdslb.com/bfs/note/cd4ee7ceb31d8b206eb89528ba4117b28519808f.png)

**LayerNorm 在操作上 和 BatchNorm (二维输入) 的关系**

LayerNorm 整个把数据转置一次，放到 BatchNorm 里面出来的结果，再转置回去，基本上可以得到 LayerNorm 的结果。

**三维输入**

27:48

Transformer 和 RNN 里面：3 维输入。

输入的是一个序列的样本，每个样本中有很多元素，是一个序列。

一个句子里面有 n 个词，每个词对应一个向量，+ 一个 batch --> 3 维

**3 维输入示意图**

列 是 seq 序列长度 n；第 3 维 feature 是每个词额外的向量，d = 512 in transformer

![](https://i0.hdslb.com/bfs/note/aa25c004e7bf42c831a5981c10b8d14571bf4c43.png)

BatchNorm

每次取一个特征，切一块（蓝色线），拉成一个向量，均值为 0 、方差为 1 的标准化。

![](https://i0.hdslb.com/bfs/note/9cad119fc87ac5101cec33a9d5fe04d8f720a55b.png)

LayerNorm (橙色)

横着切

![](https://i0.hdslb.com/bfs/note/57586df9146de6af02bd9ca49644487a9c1141b5.png)

**LayerNorm 为什么用的多？**

时序数据中 样本长度可能不一样。

举例分析：4 个长度不一样的样本，0 填充到 max_len

29:33

![](https://i0.hdslb.com/bfs/note/cc6490ee39534af1d1b45e3b3097f04dedb55136.png)

BatchNorm 切出来的结果

![](https://i0.hdslb.com/bfs/note/18083696bf3e3df838f6717d535c86281be6b3fa.png)

BatchNorm 计算均值和方差，有效的是阴影部分，其余是 0

![](https://i0.hdslb.com/bfs/note/ea4ec21bdfbe6f1a131145303288a9483233eda2.png)

Mini-batch 的均值和方差：如果样本长度变化比较大的时候，每次计算小批量的均值和方差，均值和方差的抖动大。

全局的均值和方差：测试时遇到一个特别长的全新样本 （最上方蓝色阴影块），训练时未见过，训练时计算的均值和方差可能不好用。

![](https://i0.hdslb.com/bfs/note/127886ba478d8dc9f65f146163599446ce8eb5b2.png)

LayerNorm 切出来的结果

![](https://i0.hdslb.com/bfs/note/6f89922ce09c6306d2e2ff2ff045dfab73045cad.png)

LayerNorm 每个样本自己算均值和方差，不需要存全局的均值和方差。

![](https://i0.hdslb.com/bfs/note/b5e833ecb288b2697416b5c7b1113555436b4723.png)

LayerNorm 更稳定，不管样本长还是短，均值和方差是在每个样本内计算。

Q：不同的 feature 之间做 normalization 有意义吗？

Decoder 架构

32:06

decoder 和 encoder 很像，6 个 相同 layer 的堆叠、每个 sub-layer 的 residual connections、layer normalization。

每个 layer 里有 2 个 encoder 中的 sub-layers, decoder 有第 3 个 sub-layer，对 encoder 的输出做 multi-head attention。

decoder 是 auto-regressive 自回归。当前时刻的输入集 是 之前一些时刻的输出。做预测时，decoder 不能看到 之后时刻的输出。

attention mechanism 每一次能看完完整的输入，要避免这个情况的发生。

在 decoder 训练的时候，在预测第 t 个时刻的输出的时候，decoder 不应该看到 t 时刻以后的那些输入。它的做法是通过一个带掩码 masked 的注意力机制。--> 保证 训练和预测时 行为一致。

3.2 Attention

33:36

注意力函数是 一个将一个 query 和一些 key - value 对 映射成一个输出的函数，其中所有的 query、key、value 和 output 都是一些向量。

具体来说，output 是 value 的一个加权和 --> 输出的维度 == value 的维度。

output 中 value 的权重 = 查询 query 和对应的 key 的相似度 or compatibility function

权重等价于 query 和对应的 key 的相似度

**示意图**

![](https://i0.hdslb.com/bfs/note/e2a5de335ae18a3440a22b6a337bea8300fa6b55.png)

虽然 key-value 并没有变，但是随着 query 的改变，因为权重的分配不一样，导致 输出会有不一样，这就是注意力机制。

**3.2.1 Scaled Dot-Product Attention**

35:55

不同的相似度函数 导致 不同的注意力机制

Scaled Dot-Product Attention，最简单的注意力机制。

query 和 key 的长度是等长的，都等于 dk。value 的维度是 dv，输出也是 dv。

因为 query 和 key 可以不等长，不等长是有别的办法算的。

注意力的具体计算是：对每一个 query 和 key 做内积，然后把它作为相似度。

两个向量做内积：如果这两个向量的 norm 是一样 d 的话，那么内积的值越大，它的余弦值越大，这两个向量的相似度就越高。如果你的内积的值为 0 ，这两个向量正交了，没有相似度。

attention = softmax(两个向量的内积值 / sqrt(dk)) * V ，dk 是向量的长度

使用 softmax ：一个 query 给 n 个 key - value pair ，这个 query 会跟每个 key - value pair 做内积，会产生 n 个相似度值。传入 softmax 得到 n 个非负、求和为 1 的权重值。把 softmax 得到的权重值 与 value 矩阵 V 相乘 得到 attention 输出。

实际计算：不会一个 query 一个 query 的计算，因为运算比较慢。把多个 query 写成 一个矩阵，并行化运算。

Q：n * dk

K: m * dk

Q * K T：(n * dk) * (m * dk)T = (n * m)

每一行蓝色的线：一个 query 对所有 key 的内积值，然后再除以 sqrt(dk)， 再做 softmax。 softmax 是对每一行的值做 softmax，然后每一行之间是独立的，会得到权重。

![](https://i0.hdslb.com/bfs/note/6fc0f5e7c61c35d53f275b6de8a3659d00c28422.png)

权重 softmax(Q * K T / sqrt(dk) ) (n * m) 再乘以 V （m * dv）= (n * dv) 矩阵。

![](https://i0.hdslb.com/bfs/note/911b19dbda04b91a60ab961cd11597a4e17b717c.png)

绿色的每一行它就是 attention。

attention 的计算：2 次矩阵乘法、并行计算

39:33

Scaled Dot-Product Attention 和 别的注意力机制的区别

2 种常见的注意力机制：加性的注意力机制（它可以处理你的 query 和 key 不等长的情况，点积 dot-product 的注意力机制 （本文采用 scaled，➗ sqrt(dk) ），所以你可以看到它的名字它叫做 scale 的。

选用 dot-product 原因：两种注意力机制其实都差不多， 点乘实现 简单、高效，两次矩阵乘法计算。

**scale** dot-product 原因 ➗ sqrt(dk) ：防止 softmax 函数的梯度消失。

dk 不是很大的时候，➗ 不➗ 都 ok。dk 比较大时 （2 个向量的长度比较长的时候），点积的值会比较大，or 会比较小。

当你的值比较大的时候，相对的差距会变大，导致最大值 softmax 会更加靠近于 1，剩下那些值就会更加靠近于 0。值就会更加向两端靠拢，算梯度的时候，梯度比较小。

softmax 会让大的数据更大，小的更小

因为 softmax 最后的结果是希望 softmax 的预测值，置信的地方尽量靠近，不置信的地方尽量靠近零，以保证收敛差不多了。这时候梯度就会变得比较小，那就会跑不动。

在 trasformer 里面一般用的 dk 比较大 (本文 512)，所以➗ sqrt(dk) 是一个不错的选择。

![](https://i0.hdslb.com/bfs/note/72fd12f3417407d35c18f35b8d2b7a4cd4b635f0.png)

**怎么做 mask ？**

42:01

避免在 t 时刻，看到 t 时刻以后的输入。

在计算权重的时候，t 时刻只用了 v1, ..., vt-1 的结果，不要用到 t 时刻以后的内容。

把 t 时刻以后 Qt 和 Kt 的值换成一个很大的负数，如 1 ^ (-10)，进入 softmax 后，权重为 0。 --> 和 V 矩阵做矩阵乘法时，没看到 t 时刻以后的内容，只看 t 时刻之前的 key - value pair。

理解：mask 是个 0 1 矩阵，和 attention（scale QK）size 一样，t 时刻以后 mask 为 0。

**3.3.2 Multi-head attention**

44:13

与其做一个单个的注意力函数，不如说把整个 query、key、value 整个投影 project 到 1 个低维，投影 h 次。然后再做 h 次的注意力函数，把每一个函数的输出 拼接在一起，然后 again projected，会得到最终的输出。

![](https://i0.hdslb.com/bfs/note/cc400f7ce8bcf60164c2fef0a319f256ab17c637.png)

输入是：原始的 value、key、query

进入一个线形层，线形层把 value、key、query 投影到比较低的维度。然后再做一个 scaled dot product （图 2 左图）。

执行 h 次会得到 h 个输出，再把 h 个 输出向量全部合并 concat 在一起，最后做一次线性的投影 Linear，会回到我们的 multi-head attention。

为什么要做多头注意力机制呢？一个 dot product 的注意力里面，没有什么可以学的参数。具体函数就是内积，为了识别不一样的模式，希望有不一样的计算相似度的办法。

加性 attention 有一个权重可学，也许能学到一些内容。

本文的 dot-product attention，先投影到低维，投影的 w 是可以学习的。

multi-head attention 给 h 次机会去学习 不一样的投影的方法，使得在投影进去的度量空间里面能够去匹配不同模式需要的一些相似函数，然后把 h 个 heads 拼接起来，最后再做一次投影。

46:13

有点像 CNN 多个输出通道的感觉。

multi-head attention 具体公式

46:17

Multi-head 的输入还是 Q,K,V

![](https://i0.hdslb.com/bfs/note/16de3530bde46e56c3c631a7a0203b97502c54a7.png)

但是输出是 不同的头的输出的 concat 起来，再投影到一个 WO 里面。

每一个头 hi 是把 Q,K,V 通过 可以学习的 Wq, Wk, Wv 投影到 dv 上，再通过注意力函数，得到 headi。

本文采用 8 个 heads。因为有残差连接的存在使得输入和输出的维度至少是一样的。

投影维度 dv = dmodel / h = 512 / 8 = 64，每个 head 得到 64 维度，concat，再投影回 dmodel。

**作业：**非常多的小矩阵的乘法，通过一次矩阵乘法实现。

47:32

![](https://i0.hdslb.com/bfs/note/1f69bac44bc53940cfce2567de68c3fe314ed0d6.png)

3.2.3 Applications of attentions in our model

47:41

3 种 不一样的注意力层

![](https://i0.hdslb.com/bfs/note/56f47bfd7f3314063da43092b3368113f03393c6.png)

48:21

**encoder 的注意力层：**

i.e., 句子长度是 n，encoder 的输入是一个 n 个长为 d 的向量。

假设 pn 大小设成 1 了，每一个输入词对应的是一个长为 d 的向量。

图示

![](https://i0.hdslb.com/bfs/note/f783900ab47e077e93a9fe38e242d86e0cb074a6.png)

encoder 的注意力层，有三个输入，它分别表示的是 key、value 和 query。

![](https://i0.hdslb.com/bfs/note/9c7fd445b6c32f75c9196cacb18a4c893ba512b1.png)

一根线过来，它复制成了三下：同样一个东西，既 key 也作为 value 也作为 query，所以叫做**自注意力机制**。**key、value 和 query 其实就是一个东西**，就是自己本身。

![](https://i0.hdslb.com/bfs/note/297ee9220a1282227fd3a58f76aa63e1301a88b8.png)

输入了 n 个 query，每个 query 会得到一个输出，那么会有 n 个输出。

输出 是 value 加权和（权重是 query 和 key 的相似度），输出的维度 == d -- > 输入维度 == 输出维度

示意图

![](https://i0.hdslb.com/bfs/note/7b8662dfd552b6c5c7e65423524185b47e8cb99d.png)

绿色线代表权重，和自己的相似度最大、权重线最粗。

假设和最右侧向量 相似度比较高，权重也会高一些、绿色线会粗一些。

不考虑 multi-head 和 有投影的情况：

输出是 输入的加权和，其权重来自 每个向量与其它向量的相似度。

multi-head 和 有投影的情况：

学习 h 个不一样的距离空间，使得输出变化。

**decoder 的 masked multi-head attention**

![](https://i0.hdslb.com/bfs/note/48a403d6961d6b7d42670b0ae2346f25f1a28256.png)

输入复制 3 份。

![](https://i0.hdslb.com/bfs/note/a8a67265eaf9fbd11f21e9a779ff23949cd73f33.png)

![](https://i0.hdslb.com/bfs/note/54ac6afa6e910d6db46a74ebf98c4baf71549612.png)

masked 体现在，看不到 t 时刻以后的输入，黄圈内的绿色权重为 0

**decoder 的 multi-head attention**

51:39

不再是 self-attention。

key - value 来自 encoder 的输出。

![](https://i0.hdslb.com/bfs/note/0b681586b3d850eb8133c8e05c85f5d865a50422.png)

query 是来自 decoder 里 masked multi-head attention 的输出。

![](https://i0.hdslb.com/bfs/note/d08c3943ff2c4786f3e93534456b2284f10e8ab4.png)

图中红色方块：encoder 的输出 value 和 key 。

encoder 最后一层的输出： n 个 长为 d 的向量。

图中绿色方块：

decoder 的 masked multi-head attention + Add & Norm 的输出是 m 个 长为 d 的向量。

图中蓝色方块：decoder 的输出

根据 query 算 输出： value 的加权和 （权重 取决于 红色方块 和 绿色方块 的相似度）

第 3 个 attention 层，根据 query 去有效的提取 encoder 层输出

![](https://i0.hdslb.com/bfs/note/75979535f4a5f055aa08e107c9521f784f5cd977.png)

i.e.， Hello World --> 你好世界

53:23

计算 “好” 的时候，“好” 作为 query ，会跟 “hello” 向量更相近一点，给 “hello” 向量一个比较大的权重。

但是 "world" 跟后面的词相关， "world" 跟 当前的 query （“好” ）相关度没那么高。

在算 “好” 的时候，我会给 “hello” 向量一个比较大的权重。

在算 query “世” 的时候，会给第二个 "world" 向量，一个比较大的权重。

根据解码器的输入的不一样，会根据当前的 query 向量，去在编码器的输出里面去挑我（当前 query）感兴趣的东西。

attention：query 注意到 当前的 query 感兴趣的东西，对当前的 query 的不感兴趣的内容，可以忽略掉。 --> attention 作用：在 encoder 和 decoder 之间传递信息

**3.3 Position-wise Feed-Forward Networks**

作用在最后一个维度的 MLP

54:56

MLP: applied to each position separtely and identically.

**Point-wise**: 把一个 MLP 对每一个词 （position）作用一次，对每个词作用的是同样的 MLP

FFN： Linear + ReLU + Linear

单隐藏层的 MLP，中间 W1 扩维到 4 倍 2048，最后 W2 投影回到 512 维度大小，便于残差连接。

pytorch 实现：2 个线性层。pytorch 在输入是 3d 的时候，默认在最后一个维度做计算。

画图示例

56:44

最简单情况：没有残差连接、没有 layernorm、 attention 单头、没有投影。看和 RNN 区别

attention 对输入做一个加权和，加权和 进入 point-wise MLP。（画了多个红色方块 MLP， 是一个权重相同的 MLP）

point-wise MLP 对 每个输入的点 做计算，得到输出。

attention 作用：把整个序列里面的信息抓取出来，做一次汇聚 aggregation

![](https://i0.hdslb.com/bfs/note/886f0d7acaee0e34fa6986b54f87c21a4a556d1e.png)

图中红色填充块，已经就有了，序列中感兴趣的东西

以至于我在做投影，在做 MLP 的时候映射成我更想要的那个语义空间的时候，因为这个东西已经含有了我的序列信息，所以每个 MLP 只要在对每个点独立做就行了。

历史信息，因为这个地方序列信息已经被汇聚完成，所以 MLP 是可以分开做的，也就整这个 transformer 是如何抽取序列信息，然后把这些信息加工成我最后要的语义空间，向量的过程

对比 RNN 怎么做的。

图中 绿色 表示之前的信息

![](https://i0.hdslb.com/bfs/note/3e195c0c4a16a1e17b681bd37859ce7519ed1289.png)

RNN 跟 transformer **异：如何传递序列的信息**

RNN 是把上一个时刻的信息输出传入下一个时候做输入。Transformer 通过一个 attention 层，去全局的拿到整个序列里面信息，再用 MLP 做语义的转换。

RNN 跟 transformer **同：语义空间的转换 + 关注点**

用一个线性层 or 一个 MLP 来做语义空间的转换。

**关注点**：怎么有效的去使用序列的信息。

**3.4 Embeddings and Softmax**

60:43

embedding：将输入的一个词、词语 token 映射成 为一个长为 d 的向量。学习到的长为 d 的向量 来表示整个词、词语 token。

本文 d = 512

编码器、解码器、最后 softmax 之前的 3 个 embedding 共享权重。--> 训练更简单。

Note：权重 * sqrt(dmodel = 512) ，学 embedding 的时候，会把每一个向量的 L2 Norm 学的比较小。

i.e., 学成 1， 不论维度多大，最后的值都会 = 1。

维度大的化，学到的一些权重值就会变小，但之后还需要加上 positional encoding（不会随着维度的增加而变化）。

multiply weights by sqrt(dmodel) 使得 embedding 和 positional encosing 的 scale 差不多，可以做加法。

**3.5 Positional Encoding**

01:01:56

Why? attention 不会有时序信息。

output 是 value 的 加权和（权重是 query 和 key 之间的距离，和 **序列信息** 无关）。

根本不看 key - value 对在序列哪些地方。一句话把顺序任意打乱之后，attention 出来，结果都是一样的。

**顺序会变，但是值不会变，有问题！**

在处理时序数据的时候，一句话里面的词完全打乱，那么语义肯定会发生变化，但是 attention 不会处理这个情况。 --> 加入时序信息。

**How**：RNN 把上一时刻的输出 作为下一个时刻的输入，来传递时序信息。

**How**：attention 在输入里面加入时序信息 --> positional encoding

一个在位置 i 的词，会把 i 位置信息加入到输入里面。如位置 12345，12345。

计算机表示一个 32 位的整数：32 个 bit，每个 bit 上有不同的值来表示。

一个词在嵌入层表示成一个 512 维的向量，用另一个 512 维的向量来表示一个数字，位置信息 1 2 3 4 5 6 7 8......。

表示一个位置数字信息的值，怎么计算？

周期不一样的 sin 和 cos 函数计算 --> 任何一个值可以用一个长为 512 的向量来表示。

这个长为 512 、记录了时序信息的一个 positional encoding，+ 嵌入层相加 --> 完成 把时序信息加进数据。

**详细看图解释**：输入进来进入 embedding 层之后，那么对每个词都会拿到那个向量长为 512 的一个向量。positional encodding （这个词在句子中的位置），返回一个长为 512 的向量，表示这个位置，然后把 embeding 和 positional encodding 加起来就行了。

positional encodding 是 cos 和 sin 的一个函数，在 [-1, +1] 之间抖动的。所以 input embedding * sqrt(d) ，使得乘积后的每个数字也是在差不多的 [-1, +1] 数值区间。相加完成 --> 在输入里面添加时序信息。

完成 与 positional encoding 相加 之后的部分是顺序不变的。

不管怎么打乱输入序列的顺序，进入 layer 之后，输出那些值是不变的，最多是顺序发生了相应的变化。所以就直接把顺序信息直接加在数据值里。

**模型架构总结：**

01:05:24

内容不长、但弄清楚细节比较花时间。

**4.Why Self-attention**

01:05:46

模型长什么样，为什么要这样做？设计理念

解释 Table 1

![](https://i0.hdslb.com/bfs/note/c8f7444dd83d8e2f1239c8826e2952dbeeca19ad.png)

第一个当然是他们关注的自注意力，然后是循环层、卷积层。另外一个是他构造出来一个受限的自注意力，它的有三列作比较。第一列是说我的计算复杂度当然是越低越好。第二个是说我的顺序的计算越少越好。顺序的计算就是说你下一步计算必须要等前面多少步计算完成，再算一个 layer 的时候。你越标的，那么你的并行度就越高啊。

Complexity per Layer 越少越好

n 序列长度，d 向量长度；

self-attention: O(n^2 * d) 主要是矩阵乘法，并行度高

Q（n * d） K（m * d）--> Q * KT (n * m)

self-attention 自注意力，query、key 相同 --> m = n, 复杂度为 O(n^2 * d) 【其它的矩阵运算复杂度是一样的 O() 省略了常数项 k 】

recurrent: 序列长度为 n ，一个 dense layer * 一个长为 d 的输入

循环层是要我们知道，如果你的序列是长的 N 话，它就一个一个做运算，每个里面它的主要的计算就是一个 N 乘以 N 的矩阵，一个你就是一个 dance layer 然后再乘以你一个长为 D 的一个输入，所以它是一个 N 平方，然后要做 N 次，所以是 N 乘 D 平方。

然后你对比一下这两个东西是有一定区别的，真的取决你是 N 大还是 D 大。如果你 N 大的话，当然它贵点。你第一大的话是下面一个贵一点。实际上来说，你的第一这个地方是 52，你的 N 也差不多是几百的样子。现在当然是说比较大的模型话，第一可以做到 22048 甚至更大，你的 N 相对来说也会做得比较，也是几千的样子。所以你其实现在看起来这两个东西都差不多， N 和 D 的其实在差不多的数据上面。所以这两个都差不多。

convolutional: k 比较小 3 5；CNN 和 RNN 复杂度差不多；

self-attention(restricted): query 只跟 最近的 r 个邻居计算

Sequential Operations （下一步计算，必须要等前面多少步 计算完成） 越少，并行度越高

self-attention: O(1)，矩阵乘法的并行度高

recurrent: 一步一步做运算，当前时刻的词 需要等待前面所有时刻 计算完成，--> 一个成为 N 的一个序列化的操作，在并行上是比较吃亏的。我们之前提到过。另外一个是说你最初点的那个历史信息，需要到最后那一个点的话需要走过 N 步才能过去，所以它这个地方的最长是 on 所以大家会批评。

convolutional:

self-attention(restricted):

Maximum Path Length（一个信息从一个数据点走到另外一个数据点要走多少步）越短越好

任何两个 走多少步

self-attention: O(1)，一个 query 和所有的 key 做运算。输出是所有 value 的加权和。任何 query 和任意一个很远的 key-value pair，只要一次就能过来。

recurrent:

convolutional:

self-attention(restricted):

**总结**：实际使用 self-attention

前 3 个算法的时间复杂度差不多，

attention 需要更多的数据、

**7. 实验**

01:12:51

**5.1Training Data and Benchmarking**

WMT 2014 数据集

byte-pair encoding, BPE 提取词根 --> 处理一个词的多种变化 -ing -ed、37000 tokens（英语德语共享字典 --> encoder 和 decoder 用一个东西、模型更简单、Embedding 共享权重）

英语法语用了一个更大的数据集

**5.2 Hardware and Schedule**

01:14:16

8 P100 GPUs, 现在 Google 推荐用 TPUs (适合大的矩阵乘法)

0.4 seconds / batch, 100, 000 steps or 12 hours

big models: 1s / step, 300, 000 steps, 3.5 days

**5.3 Optimizer：没有可以调的 Adam**

01:15:33

Adam 训练器； beta2 常见为 0.99 or 0.999

学习率通过公式计算

dmodel ^ (-0.5) == 1 / sqrt(dmodel) --> **模型要学习的宽度越宽，学习率越低**

warm-up, 从一个小的值慢慢爬到一个高的值，到一定值之后，根据 step_num ^ 0.5 衰减

warmup-steps 4000 步

![](https://i0.hdslb.com/bfs/note/c751eaea75427c0c8618a96c451f631d11eccd2d.png)

学习率不用调，Adam 对学习率不那么敏感，dmodel 已经考虑在公式里， step_num 也是不错的 schedule。

**5.4 Regularization**

01:16:35

**Residual Dropout**

each sub-layer: 多头注意力 和 之后的 MLP，每一层的输出上，在进入残差连接之前和进入 layernorm 之前，使用 dropout，P drop = 0.1

10% 的元素 重置为 0， 剩下的值 * 1.1

**总结**：带权重的层，输出都使用了 dropout

01:17:38

**Label Smoothing**

**Inception V3**

0 - 1 标签，softmax 很难趋近为 1。很 soft，输出值很大，才会激活为 1。

正确的词，softmax 的输出为 0.1 即可，剩下的值是 0.9 / 字典大小

损失 perplexity，模型的困惑度（不确信度）、log（loss）做指数。因为正确的标签只需要给到 10 %。

模型不那么精确，可以提高 accuracy and BLEU score

01:19:09

**超参数的对比：只调 N 层数、dmodel 模型的宽、注意力的 head 数 h**

![](https://i0.hdslb.com/bfs/note/dfd9d4184f7c029f5fb4346bf9181b760aee6821.png)

N 是 堆多少层

dmodel 模型的宽度：是 一个 token 进来之后表示成 多长的向量

dff 是 MLP 中间隐藏层输出的大小

h 是 注意力层 head 的个数

dk, dv 分别是 key 和 value 的维度

Pdrop 是 dropout 的概率

Els 是 label smoothing 要学的正确的 label 值为多少

dmodel = h * dk

big model：模型宽度 * 2, dff * 2，

h * 2 --> dk 和 dv 不用变维度

模型更复杂 --> Pdrop = 0.3, train steps 300 K

模型可调的参数：多少层 N，多宽 dmodel, 模型的 head 数 h，剩下的参数按比例计算，便于后人的工作。

Table 4: 除了 MT，其它的任务也表现不错。

**8. 评论：写作、Transformer 模型、 attention 标题、未来**

01:21:47

**写作**：非常简洁、每句话在讲一件事情；没有太多的写作技巧、提出了 Transformer 模型、和 CNN、 RNN 的对比

建议：写作技巧 -- 将一个故事，有代入感，设计理念是什么、对整篇文章的思考是什么，不那么重要的放 appendix

**Transformer 模型出圈 --> 多模态**：像 CNN 对 CV 的作用，不仅仅应用在 NLP，在 CV、Video 上也有很好的应用。

启示：一个新的模型可以在 DL 上 通用。人的感知是多模态的、使得 Transformer 在文本、语音、视频抽取多维特征。

**对 Transformer 中 attention 的理解**：attention 只是起到 把整个序列的信息聚合起来 的作用，后面的 MLP 和 残差连接 是缺一不可的。去掉 MLP 和 残差连接，只有 attention，也什么都训练不出来。

**呼应标题 attention is all you need**：不是只有 attention 就行了。

attention 没有对数据的顺序建模，为什么 ko RNN 呢？

RNN 显示的建模了序列信息，理论应该比 attention 效果更好。

attention 用了更广泛的 inductive bias 归置偏置，使得 attention 没有用空间上的假设，取得和 CNN 一样、 甚至更好的结果。

代价：假设更加一般、对数据的抓取能力差，需要使用更多的数据、更大的模型 才能训练出一样的效果

注：[inductive bias](https://www.zhihu.com/question/264264203)

![](https://i0.hdslb.com/bfs/note/83de734919cd962be99064a32ce1e95a06416b6e.png)

CNN 的 inductive bias 应该是 locality 和 spatial invariance，即空间相近的 grid elements 有联系而远的没有，和空间不变性（kernel 权重共享）

RNN 的 inductive bias 是 sequentiality 和 time invariance，即序列顺序上的 timesteps 有联系，和时间变换的不变性（rnn 权重共享）

**attention 给研究者的鼓励**：在 CNN 和 RNN 之外，也有新的模型能够 打败它们。 有研究者在尝试 就用 MLP or 更简单的架构，在图片、文本上去的很好的效果。

未来 DL 领域会有更多的模型出现，更有意思~