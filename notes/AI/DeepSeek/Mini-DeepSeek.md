
- 本节复现DeepSeek
- [【全网独家】手动复现DeepSeek v3！从零训练Mini DeepSeek v3！模型预训练+全量指令微调+DPO强化学习微调全流程实战_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1KtwueYE54?spm_id_from=333.788.videopod.sections&vd_source=d5dfd0b2a455103b5d899d64b40b2576)




# 基本情况介绍
---
- DeepSeek为自研MoE模型，671B参数，激活37B，在14.8Ttoken上进行了预训练

- 模型架构
	- 基础架构：基于Transformer框架，采用多头潜在注意力（MLA）和MoE混合专家框架。引入无辅助损失的负载均衡策略，减少负载均衡对性能的影响。
	- 多令牌预测MTP
- 基础设施
	- 计算集群：2048个H800，节点内通过NVLink和NVSwitch连接GPU，节点间IB互连
	- 训练框架：HAL-LLM框架、16路流水线并行、64路专家并行、Zero-1数据并行。DualPipe算法优化，提高训练效率，减少流水线旗袍，充分利用带宽，优化内存占用。
	- FP8训练：细粒度混合精度框架，采用tile-wise或block-wise分组量化策略，
	- 推理和部署：在H800
- 预训练：文本表达习惯
	- 数据构建：优化预训练语料库，增加数学和编程样本比例，扩展多语言覆盖，采用文档打包方法和FIM(*生成前后，预测中间*)策略，使用Byte-Level BPE分词器并优化相关设置。
		- 数据清洗
	- 超参数：61层，隐藏维度7168，采用AdamW优化器，设置学习率调度，批量大小调度等参数
	- 长上下文扩展：采用YaRN进行上下文扩展，分两部将上下文窗口从4K扩展到128K，模型在长上下文任务上表现良好
- 后训练：明白对话如何开展
	- SFT监督微调：1.5M实例的数据集，全量指令微调
	- RL强化学习：DPO强化学习，GRPO

# 训练流程概述
---
- 准备工作
	- 工具包
	- 分词器训练
- 预训练
	- pretrain
	- LMconfig
	- dataset
	- model
- 后训练
	- full_sft：全指令微调
	- dpo_train：DPO强化学习

---
---
---


# 准备工作
---
## 分词器
### 概念
- Tokenizer
- 将文本转换为一系列离散的标记，之后通过嵌入层将这些标记转换为向量输入模型，为embedding作准备
- 子词分词：将常见的词作为整体保留，将不常见的词进一步分解为子词的组合
	- 降低复杂性：使词汇表控制在合理的大小
	- 提高泛化能力：能够有效处理新词和变体
	- 提高效率：较小的词汇表可以减少嵌入矩阵大小，从而减少参数量和训练时间
- 分词器通过分析语料库中词频词形的变化，选择最适合的子词或词汇构建词汇表，确保词汇表足够小但能覆盖大部分文本
- 分词器会学习语义关系
- （out of vocabulary, OOV）处理未登录词：正确的拆解，保持语义一致性

### 实现
- 中文在字级别，英文在subword级别
1. Normalization：标准化，大写转化为小写
2. pre-tokenizaton：预处理，按特定标记初步分词，如空格、逗号、句号等。将串转化为列表
3. model：通过模型实现分词，如BPE、wordpiece、unigram
4. postprocessor：后处理，添加起止符

#### BPE/wordpiece
- 将频繁出现的单元合并成新的单元，构建一个子词级别的词汇表，初始设定词汇表的期望大小

- score=(freq_of_pair)/(freq_of_first_element×freq_of_second_element)

这个公示是非常抽象啊，那我们以huggingface上的例子来介绍是如何运行的：

> **1.**假设我们有如下的一个语料库统计结果：("hug", 10), ("pug", 5), ("pun", 12), ("bun", 4), ("hugs", 5)  
> **2.**那按照第一步字符切分则为：("h" "##u" "##g", 10), ("p" "##u" "##g", 5), ("p" "##u" "##n", 12), ("b" "##u" "##n", 4), ("h" "##u" "##g" "##s", 5)  
> **3.**得到最初的词汇表vocab： ["b", "h", "p", "##g", "##n", "##s", "##u"] 对应的出现的次数是：4，15，17，20，16，5，36  
> **4.**我们以##u和##g为例，这一对在所有的语料中接连出现了20次，##u出现了36次，##g出现了20次  
> socre为：20/(36*20) = 1/36 这个分数很低，如果想选出高的分数，那么需要降低这两个出现的次数，我们**5.** 可以看到##g，##s的分数为：5/(20*5) = 1/20,这个对的分数高，所以这个对应该合并成##gs  
> **6.** 此时vocab更新为： ["b", "h", "p", "##g", "##n", "##s", "##u", "##gs"]  
> **7.** 根据vovab就可以统计语料为：("h" "##u" "##g", 10), ("p" "##u" "##g", 5), ("p" "##u" "##n", 12), ("b" "##u" "##n", 4), ("h" "##u" "##gs", 5)  
> **8.** 这样又可以循环一遍找到哪两个对应该合并，直到vocab size达到我们想要的需求，或者分数到达一个阈值。



### 训练分词器model
- 分词器的训练与神经网络不同。神经网络的权重是自然随机化的，即不同的种子会得到不同的结果。但分词器的训练是一个统计学过程，是确定性的，同一语料库必然得到相同的结果。
#### 从旧的tokenizer训练新的
- huggingface的transformers库中有可以训练与现有标记器相同特征的转换器：`AutoTokenizer.train_new_from_iterator()`
#### 从语料库训练
- 导包
	- tokennizers
	- datasets
	- transformers-AutoTokenizer
- 加载数据集
	- 实现读取数据集的迭代器函数
-  初始化分词器--BPE初始化
	- model.BPE
```Python
# 初始化tokenizer
tokenizer = Tokenizer(models.BPE())  
tokenizer.pre_tokenizer = pre_tokenizers.ByteLevel(add_prefix_space=False)

# 定义特殊token
special_tokens = ["<unk>", "<s>", "</s>"]

# 设置训练器并添加特殊token
trainer = trainers.BpeTrainer(
    vocab_size=6400,
    special_tokens=special_tokens,  # 确保这三个token被包含
    show_progress=True,
    initial_alphabet=pre_tokenizers.ByteLevel.alphabet()
)

print("分词器初始化成功，准备训练。")
```
- 训练分词器
```Python
# 读取文本数据
texts = read_texts_from_jsonl(data_path)

# 训练tokenizer
tokenizer.train_from_iterator(texts, trainer=trainer)

print("分词器训练完成！")
```
- 保存为json并设置解码器decoder

# 预训练
---
## 前言
- 在大规模无监督数据上进行初步模型训练，使模型学习到通用的语言模式、知识表征和统计特征
- 通过预训练，模型可以在后续的特定任务上（如分类、生成、翻译等）以更快的速度和更高的准确性进行微调（Fine-tuning）。
- 预训练的目标是使模型能够有效捕捉数据中的规律，学会高效的特征表示，从而为后续的任务奠定基础。这一阶段通常涉及训练深度神经网络，如Transformer架构，通过处理大规模文本数据，模型能够学会上下文依赖关系、词汇语义关系等复杂的语言特征。

### 数据
- 预训练数据集
	- 数据规模
	- 数据多样性
	- 数据质量
	- 领域相关性



### 防止遗忘
- 微调：通过额外的训练使模型对某一部分数据更敏感
- 知识蒸馏：将大模型的知识浓缩到小模型中
- 数据采样
- 加权训练
- 对比学习
- 连续学习
- 数据增强
- 生成对抗


- 开启、打断与中继
- 原版：260专家/20~30专家
- 4专家/2专家

## 数据集处理
- ==采用GitHub上开源的序列猴子开源数据集，4.3G的CSV==
- 数据清洗
## 文件目录
- LMconfig.py
	- 超参数
- model.py
	- 模型架构
- dataset.py
	- 数据读取

## model
- RMSNorm：均方根