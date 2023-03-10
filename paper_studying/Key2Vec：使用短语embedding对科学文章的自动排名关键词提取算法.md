# 摘要

关键词提取是自然语言处理中的一项基本任务，它有助于将文档映射到一组有代表性的短语。

在本文中，提出了一种无监督技术(Key2Vec)，它利用短语embedding来对从科学文章中提取的关键词进行排名。具体来说，我们提出了一种有效的处理文本文档的方法，用于训练多词短语embedding，用于科学文章的主题表示，并使用主题加权PageRank对从中提取的关键短语进行排名。对基准数据集进行评估，产生最先进的结果。

# 一、介绍

关键字是单个或多词的语言单位，代表文档的突出方面。一些主要的挑战是要处理的文档长度不同，结构不一致，以及在不同领域表现良好的开发策略。

随着深度学习技术应用于自然语言处理(NLP)的最新进展，趋势是将单词表示为密集实值向量，通常称为词嵌入。嵌入向量，是为了保持词之间语义和句法的相似性。一些最流行的训练词嵌入的方法是Word2Vec，Glove和Fasttext。

在科学文章上尝试特定于领域的嵌入。

**本文特点：使用多词短语嵌入来构建给定文档的主题表示，并为短语分配主题权重。**

表1显示从样本研究摘要中使用Key2Vec提取的排名关键短语。

![1671327131873](D:%5CTypora%5Cuser-image%5C1671327131873.png)

# 二、方法论

> 主要步骤：

1. 候选词选择；
2. 候选词评分；
3. 候选词排名。

所有的步骤都取决于文本处理步骤的选择，以及我们在大量科学文章语料库上训练的短语嵌入模型。接下来我们将解释它们，并详细描述它们的实现。

## 1、文本处理

使用Spacy1包，将文本文档分解为句子，将句子标记为uniggram标记，以及从中识别名词短语和命名实体。在此过程中，如果在句子的特定偏移处检测到命名实体，则不考虑在同一偏移处出现的名词短语。

过滤掉以下内容：

- 完全是数字的名词短语和命名实体(命名实体就是那些人名、机构名等)。
- 属于以下类别的命名实体将被过滤掉：DATE、TIME、PERCENT、MONEY、QUANTITY、ORDINAL、CARDINAL。
- 删除标准停止词。
- 除“-”外，标点符号被删除。
- 如果普通形容词和转述动词作为名词短语/命名实体的第一个或最后一个标记出现，则删除它们。
- 限定词从名词短语/命名实体的第一个标记中删除。
- 名词短语/命名实体的第一个或最后一个标记属于下列词性：感叹词、助词、协调连词、连词、数字、代词、从属连词、标点、符号、小品词、其它，被删除。
- 名词短语/命名实体的开始和结束标记如果属于英语停止词和功能词的标准列表，将被删除。

并使用人工制作的正则表达式来清理在上述数据清理步骤之后获得的最终令牌列表：

- 去掉前面/后面的垃圾字符。
- 处理悬空/向后括号。我们不允许'('或')'不带另一个出现。
- 处理奇怪分隔的连字符。
- 处理奇怪分隔的撇号词。
- 规范化的空白。

由此产生的字母符号和多词短语将按照它们在原句子中出现的顺序合并。图1展示了文本处理管道如何在示例句上工作，用于准备作为嵌入算法输入的训练样本。

![1671331141442](D:%5CTypora%5Cuser-image%5C1671331141442.png)

## 2、训练短语Embedding模型

该方法在很大程度上依赖于底层嵌入。使用Fasttext直接训练多词短语嵌入，而不是先训练非字母单词的嵌入模型然后结合它们的密集向量来获得多词短语的向量。训练词汇包括单字和多字短语。

底层嵌入模型的主要目的是捕获由单个词和多词短语组成的文本单元之间的语义和句法相似性。Fasttext可以捕获单词之间的语义和形态相似性。

> 使用负抽样训练一个Fasttext - skipgram模型，上下文窗口大小为5，维数为100，epoch数设置为10。

**候选词选择：**此步骤有助于从可以从文档中提取的所有短语集中选择候选关键字，并且通常用于大多数自动排名关键字提取系统。我们将给定的文档分成句子，并提取前面描述的名词短语和命名实体。作为这一步的输出，我们就文档$d_i$得到一组唯一的短语($C_{d_i}=\{c_1,\dots,c_n \}_{d_i}$)，用于后续两步的评分和排名。

**候选词评分：**在这一步中，我们将主题向量($\hat{τ_{d_i}}$)分配给文档($d_i$)。主题向量可以根据正在处理的文档类型和我们希望在最终结果中获得的关键字类型进行调优。

用余弦距离($\frac{1}{1-cos(c_j^{d_i},c_k^{d_i})}$)来表示候选词与主题向量的相似度。

**候选词排名：**为了对候选关键词进行最终排名，我们使用加权个性化PageRank算法。

# 三、实验和结果

![1671415634547](D:%5CTypora%5Cuser-image%5C1671415634547.png)

Key2Vec在指标上的性能如表2所示。

表3和表4显示了Key2Vec与一些最先进的系统的比较，这些系统分别在Inspec和SemEval 2010数据集上表现最佳。

# 四、结论

在本文中，提出了一个自动提取和排序科学文章关键字的框架。展示了一种训练短语嵌入的有效方法，并证明了它在构建科学文章的主题表示和为候选关键词分配主题权重方面的有效性。还引入了主题加权PageRank来对候选关键字进行排名。