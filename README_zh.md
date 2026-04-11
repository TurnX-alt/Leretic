[English](README.md) | [简体中文](README_zh.md)

<img width="128" height="128" align="right" alt="Logo" src="https://github.com/user-attachments/assets/df5f2840-2f92-4991-aa57-252747d7182e" />

# Heretic: 全自动去除语言模型审查<br><br>[![Discord](https://img.shields.io/discord/1447831134212984903?color=5865F2&label=discord&labelColor=black&logo=discord&logoColor=white&style=for-the-badge)](https://discord.gg/gdXc48gSyT) [![Follow us on Hugging Face](https://huggingface.co/datasets/huggingface/badges/resolve/main/follow-us-on-hf-md-dark.svg)](https://huggingface.co/heretic-org)

[![#1 Repository of the Day](https://trendshift.io/api/badge/repositories/20538)](https://trendshift.io/repositories/20538)

Heretic 是一款可以在无需昂贵的后训练（post-training）的情况下，移除基于 Transformer 架构的语言模型的审查（也称为“安全对齐”）的工具。
它结合了方向性消融（directional ablation，即“abliteration”）的高级实现（[Arditi et al. 2024](https://arxiv.org/abs/2406.11717)，Lai 2025（[1](https://huggingface.co/blog/grimjim/projected-abliteration)，[2](https://huggingface.co/blog/grimjim/norm-preserving-biprojected-abliteration)）），以及由 [Optuna](https://optuna.org/) 驱动的基于 TPE 的参数优化器。

这种方法使得 Heretic 能够**完全自动地工作。** Heretic 通过同时最小化模型拒绝回答的次数以及与原始模型的 KL 散度，来寻找高质量的消融参数。这会产生一个去除了审查的模型，同时尽可能地保留原始模型的智能。使用 Heretic 不需要深入了解 Transformer 的内部结构。实际上，任何懂得如何运行命令行程序的人都可以使用 Heretic 对语言模型进行去审查。

<img width="650" height="715" alt="Screenshot" src="https://github.com/user-attachments/assets/d71a5efa-d6be-4705-a817-63332afb2d15" />

&nbsp;

在默认配置下无监督运行，Heretic 能够产生质量可与人类专家手动创建的消融模型相媲美的去审查模型：

| 模型 | 对于“有害”提示的拒绝次数 | 对于“无害”提示与原始模型的 KL 散度 |
| :--- | ---: | ---: |
| [google/gemma-3-12b-it](https://huggingface.co/google/gemma-3-12b-it) (原版) | 97/100 | 0 *(根据定义)* |
| [mlabonne/gemma-3-12b-it-abliterated-v2](https://huggingface.co/mlabonne/gemma-3-12b-it-abliterated-v2) | 3/100 | 1.04 |
| [huihui-ai/gemma-3-12b-it-abliterated](https://huggingface.co/huihui-ai/gemma-3-12b-it-abliterated) | 3/100 | 0.45 |
| **[p-e-w/gemma-3-12b-it-heretic](https://huggingface.co/p-e-w/gemma-3-12b-it-heretic) (本文方法)** | **3/100** | **0.16** |

Heretic 生成的版本没有任何人工参与，实现了与其他消融模型相同水平的拒绝抑制，但具有低得多的 KL 散度，这表明对原始模型能力的损害更小。
*(你可以使用 Heretic 的内置评估功能重现这些数字，例如 `heretic --model google/gemma-3-12b-it --evaluate-model p-e-w/gemma-3-12b-it-heretic`。请注意，确切的值可能取决于平台和硬件。上表是在 RTX 5090 上使用 PyTorch 2.8 编制的。)*

当然，数学指标和自动化基准测试永远无法说明全部情况，也无法替代人类评估。使用 Heretic 生成的模型受到了用户的好评（链接和强调为后加）：

> “之前我持怀疑态度，但我刚刚下载了 [**GPT-OSS 20B Heretic**](https://huggingface.co/p-e-w/gpt-oss-20b-heretic) 模型，天哪。它对敏感话题给出了格式正确的长篇回复，使用了你所期望的未经审查的模型的纯正的未审查词汇，还能生成包含细节等的 Markdown 格式表格。看起来这是迄今为止该模型最好的消融版本……”
> [*(评论链接)*](https://old.reddit.com/r/LocalLLaMA/comments/1oymku1/heretic_fully_automatic_censorship_removal_for/np6tba6/)

> “[**Heretic GPT 20b**](https://huggingface.co/p-e-w/gpt-oss-20b-heretic) 似乎是我尝试过的最好的未审查模型。它没有破坏模型的智能，而且能够正常回答通常会被基础模型拒绝的提示词。”
> [*(评论链接)*](https://old.reddit.com/r/LocalLLaMA/comments/1oymku1/heretic_fully_automatic_censorship_removal_for/npe9jng/)

> “[[**Qwen3-4B-Instruct-2507-heretic**](https://huggingface.co/p-e-w/Qwen3-4B-Instruct-2507-heretic)] 是一直以来我能够用 16GB 显存运行的最好的未量化的消融模型。”
> [*(评论链接)*](https://old.reddit.com/r/LocalLLaMA/comments/1phjxca/im_calling_these_people_out_right_now/nt06tji/)

Heretic 支持大多数密集模型，包括许多多模态模型，以及几种不同的 MoE 架构。它尚未支持 SSM/混合模型、具有非均匀层的模型以及某些新型的注意力系统。

你可以在 [Hugging Face 上](https://huggingface.co/collections/p-e-w/the-bestiary) 找到一小部分使用 Heretic 去审查的模型，此外社区还创建并发布了[远超 1000 个](https://huggingface.co/models?other=heretic) Heretic 模型。


## 用法

准备一个安装了适合你硬件的 PyTorch 2.2+ 的 Python 3.10+ 环境。然后运行：

```
pip install -U heretic-llm
heretic Qwen/Qwen3-4B-Instruct-2507
```

将 `Qwen/Qwen3-4B-Instruct-2507` 替换为你想要去审查的任何模型。

这个过程是完全自动的，不需要配置；然而，Heretic 有多种配置参数，可以通过更改来实现更多的控制。运行 `heretic --help` 查看可用的命令行选项，或者如果你更喜欢使用配置文件，请查看 [`config.default.toml`](config.default.toml)。

在程序运行开始时，Heretic 会对系统进行基准测试，以确定最佳的批大小（batch size），从而充分利用可用的硬件。
在 RTX 3090 上，使用默认配置，对 Llama-3.1-8B-Instruct 进行去审查大约需要 45 分钟。请注意，Heretic 支持使用 bitsandbytes 进行模型量化，这可以大幅减少处理模型所需的显存（VRAM）。将 `quantization` 选项设置为 `bnb_4bit` 以启用量化。

在 Heretic 完成对模型的去审查后，你可以选择保存该模型、将其上传到 Hugging Face、与其进行对话以测试其效果，或这些操作的任意组合。


## 研究功能

除了其主要的功能移除模型审查之外，Heretic 还提供了旨在支持对模型内部语义（可解释性）进行研究的功能。为了使用这些功能，你需要使用可选的 `research` 额外包来安装 Heretic：

```
pip install -U heretic-llm[research]
```

这使你能够使用以下功能：

### 通过传递 `--plot-residuals` 生成残差向量图

当使用此标志运行时，Heretic 将：

1. 对于“有害”和“无害”提示，计算每个 Transformer 层的第一个输出 token 的残差向量（隐藏状态）。
2. 从残差空间到 2D 空间执行 [PaCMAP 投影](https://github.com/YingfanWang/PaCMAP)。
3. 通过其几何中位数左右对齐“有害”/“无害”残差的投影，以使连续层的投影更加相似。此外，针对每个新层，PaCMAP 均使用前一层的投影进行初始化，从而尽量减少破坏性的转换。
4. 绘制投影的散点图，为每层生成一张 PNG 图像。
5. 生成一个动画，以动画 GIF 的形式显示残差在各层之间是如何转换的。

<img width="800" height="600" alt="Plot of residual vectors" src="https://github.com/user-attachments/assets/981aa6ed-5ab9-48f0-9abf-2b1a2c430295" />

请参阅[配置文件](config.default.toml)，了解允许你控制生成的图的各个方面的选项。

请注意，PaCMAP 是一项在 CPU 上执行的昂贵操作。对于较大的模型，计算所有层的投影可能需要一个小时或更长时间。

### 通过传递 `--print-residual-geometry` 打印关于残差几何的细节

如果你对如何定量分析“有害”和“无害”提示的残差向量之间的关系感兴趣，此标志将为你提供下表，其中包含许多有助于理解上述关系的指标（此处以 [gemma-3-270m-it](https://huggingface.co/google/gemma-3-270m-it) 为例）：

```
┏━━━━━━━┳━━━━━━━━┳━━━━━━━━━━┳━━━━━━━━━┳━━━━━━━━━━┳━━━━━━━━━┳━━━━━━━━━━┳━━━━━━━━━━┳━━━━━━━━━━┳━━━━━━━━━━┳━━━━━━━━━━┳━━━━━━━━━┳━━━━━━━━━┳━━━━━━━━┓
┃ Layer ┃ S(g,b) ┃ S(g*,b*) ┃  S(g,r) ┃ S(g*,r*) ┃  S(b,r) ┃ S(b*,r*) ┃      |g| ┃     |g*| ┃      |b| ┃     |b*| ┃     |r| ┃    |r*| ┃   Silh ┃
┡━━━━━━━╇━━━━━━━━╇━━━━━━━━━━╇━━━━━━━━━╇━━━━━━━━━━╇━━━━━━━━━╇━━━━━━━━━━╇━━━━━━━━━━╇━━━━━━━━━━╇━━━━━━━━━━╇━━━━━━━━━━╇━━━━━━━━━╇━━━━━━━━━╇━━━━━━━━┩
│     1 │ 1.0000 │   1.0000 │ -0.4311 │  -0.4906 │ -0.4254 │  -0.4847 │   170.29 │   170.49 │   169.78 │   169.85 │    1.19 │    1.31 │ 0.0480 │
│     2 │ 1.0000 │   1.0000 │  0.4297 │   0.4465 │  0.4365 │   0.4524 │   768.55 │   768.77 │   771.32 │   771.36 │    6.39 │    5.76 │ 0.0745 │
│     3 │ 0.9999 │   1.0000 │ -0.5699 │  -0.5577 │ -0.5614 │  -0.5498 │  1020.98 │  1021.13 │  1013.80 │  1014.71 │   12.70 │   11.60 │ 0.0920 │
│     4 │ 0.9999 │   1.0000 │  0.6582 │   0.6553 │  0.6659 │   0.6627 │  1356.39 │  1356.20 │  1368.71 │  1367.95 │   18.62 │   17.84 │ 0.0957 │
│     5 │ 0.9987 │   0.9990 │ -0.6880 │  -0.6761 │ -0.6497 │  -0.6418 │   766.54 │   762.25 │   731.75 │   732.42 │   51.97 │   45.24 │ 0.1018 │
│     6 │ 0.9998 │   0.9998 │ -0.1983 │  -0.2312 │ -0.1811 │  -0.2141 │  2417.35 │  2421.08 │  2409.18 │  2411.40 │   43.06 │   43.47 │ 0.0900 │
│     7 │ 0.9998 │   0.9997 │ -0.5258 │  -0.5746 │ -0.5072 │  -0.5560 │  3444.92 │  3474.99 │  3400.01 │  3421.63 │   86.94 │   94.38 │ 0.0492 │
│     8 │ 0.9990 │   0.9991 │  0.8235 │   0.8312 │  0.8479 │   0.8542 │  4596.54 │  4615.62 │  4918.32 │  4934.20 │  384.87 │  377.87 │ 0.2278 │
│     9 │ 0.9992 │   0.9992 │  0.5335 │   0.5441 │  0.5678 │   0.5780 │  5322.30 │  5316.96 │  5468.65 │  5466.98 │  265.68 │  267.28 │ 0.1318 │
│    10 │ 0.9974 │   0.9973 │  0.8189 │   0.8250 │  0.8579 │   0.8644 │  5328.81 │  5325.63 │  5953.35 │  5985.15 │  743.95 │  779.74 │ 0.2863 │
│    11 │ 0.9977 │   0.9978 │  0.4262 │   0.4045 │  0.4862 │   0.4645 │  9644.02 │  9674.06 │  9983.47 │  9990.28 │  743.28 │  726.99 │ 0.1576 │
│    12 │ 0.9904 │   0.9907 │  0.4384 │   0.4077 │  0.5586 │   0.5283 │ 10257.40 │ 10368.50 │ 11114.51 │ 11151.21 │ 1711.18 │ 1664.69 │ 0.1890 │
│    13 │ 0.9867 │   0.9874 │  0.4007 │   0.3680 │  0.5444 │   0.5103 │ 12305.12 │ 12423.75 │ 13440.31 │ 13432.47 │ 2386.43 │ 2282.47 │ 0.1293 │
│    14 │ 0.9921 │   0.9922 │  0.3198 │   0.2682 │  0.4364 │   0.3859 │ 16929.16 │ 17080.37 │ 17826.97 │ 17836.03 │ 2365.23 │ 2301.87 │ 0.1282 │
│    15 │ 0.9846 │   0.9850 │  0.1198 │   0.0963 │  0.2913 │   0.2663 │ 16858.58 │ 16949.44 │ 17496.00 │ 17502.88 │ 3077.08 │ 3029.60 │ 0.1611 │
│    16 │ 0.9686 │   0.9689 │ -0.0029 │  -0.0254 │  0.2457 │   0.2226 │ 18912.77 │ 19074.86 │ 19510.56 │ 19559.62 │ 4848.35 │ 4839.75 │ 0.1516 │
│    17 │ 0.9782 │   0.9784 │ -0.0174 │  -0.0381 │  0.1908 │   0.1694 │ 27098.09 │ 27273.00 │ 27601.12 │ 27653.12 │ 5738.19 │ 5724.21 │ 0.1641 │
│    18 │ 0.9184 │   0.9196 │  0.1343 │   0.1430 │  0.5155 │   0.5204 │   190.16 │   190.35 │   219.91 │   220.62 │   87.82 │   87.59 │ 0.1855 │
└───────┴────────┴──────────┴─────────┴──────────┴─────────┴──────────┴──────────┴──────────┴──────────┴──────────┴─────────┴─────────┴────────┘
g = 良好提示的残差向量平均值
g* = 良好提示的残差向量几何中位数
b = 糟糕提示的残差向量平均值
b* = 糟糕提示的残差向量几何中位数
r = 针对平均值的拒绝方向（即 b - g）
r* = 针对几何中位数的拒绝方向（即 b* - g*）
S(x,y) = x 和 y 的余弦相似度
|x| = x 的 L2 范数
Silh = 良好/糟糕聚类的残差平均轮廓系数
```


## Heretic 是如何工作的

Heretic 实现了一种参数化的方向性消融变体。对于每个受支持的 Transformer 组件（目前是注意力输出投影和 MLP 下投影），它会识别每个 Transformer 层中的相关矩阵，并相对于相关的“拒绝方向”将它们正交化，从而抑制该方向在与该矩阵相乘的结果中的表达。

每层的拒绝方向计算为“有害”和“无害”示例提示的第一个 token 残差之间的均值差。

消融过程由几个可优化的参数控制：

* `direction_index`：要么是拒绝方向的索引，要么是特殊值 `per layer`，表示每层应使用与该层关联的拒绝方向进行消融。
* `max_weight`、`max_weight_position`、`min_weight` 和 `min_weight_distance`：对于每个组件，这些参数描述了层级上消融权重核（kernel）的形状和位置。下图说明了这一点：

<img width="800" height="500" alt="Explanation" src="https://github.com/user-attachments/assets/82e4b84e-5a82-4faf-b918-ac642f9e4892" />

&nbsp;

与现有的消融系统相比，Heretic 的主要创新在于：

* 消融权重核的形状非常灵活，结合自动参数优化，可以改善合规性/质量之间的权衡。Maxime Labonne 之前在 [gemma-3-12b-it-abliterated-v2](https://huggingface.co/mlabonne/gemma-3-12b-it-abliterated-v2) 中探索了非恒定消融权重。
* 拒绝方向索引是一个浮点数而不是整数。对于非整数值，将对两个最近的拒绝方向向量进行线性插值。这解锁了均值差计算所识别方向之外的大量额外方向空间，通常能使优化过程找到比属于任何单个层的方向更好的方向。
* 针对每个组件分别选择消融参数。我发现 MLP 干预往往比注意力干预对模型更具破坏性，因此使用不同的消融权重可以挤出一些额外的性能。


## 先前技术

我知道以下公开可用的消融技术实现：

* [AutoAbliteration](https://huggingface.co/posts/mlabonne/714992455492422)
* [abliterator.py](https://github.com/FailSpy/abliterator)
* [wassname's Abliterator](https://github.com/wassname/abliterator)
* [ErisForge](https://github.com/Tsadoq/ErisForge)
* [Removing refusals with HF Transformers](https://github.com/Sumandora/remove-refusals-with-transformers)
* [deccp](https://github.com/AUGMXNT/deccp)

请注意，Heretic 是从头开始编写的，没有重用这些项目中任何一个的代码。


## 致谢

Heretic 的开发受到了以下内容的启发：

* [原始的消融论文（Arditi et al. 2024）](https://arxiv.org/abs/2406.11717)
* [Maxime Labonne 关于消融的文章](https://huggingface.co/blog/mlabonne/abliteration)，以及他自己的消融模型卡片中的一些细节（见上文）
* Jim Lai 描述[“投影消融”（projected abliteration）](https://huggingface.co/blog/grimjim/projected-abliteration)和[“保范数双重投影消融”（norm-preserving biprojected abliteration）](https://huggingface.co/blog/grimjim/norm-preserving-biprojected-abliteration)的文章


## 引用

如果你在研究中使用了 Heretic，请使用以下 BibTeX 条目进行引用：

```bibtex
@misc{heretic,
  author = {Weidmann, Philipp Emanuel},
  title = {Heretic: Fully automatic censorship removal for language models},
  year = {2025},
  publisher = {GitHub},
  journal = {GitHub repository},
  howpublished = {\url{https://github.com/p-e-w/heretic}}
}
```


## 许可证

Copyright &copy; 2025-2026  Philipp Emanuel Weidmann (<pew@worldwidemann.com>) + contributors

本程序是自由软件：你可以根据自由软件基金会发布的 GNU Affero 通用公共许可证的条款，即该许可证的第 3 版或（由你选择）任何更高版本，来重新分发和/或修改它。

分发本程序是希望它会有用，但没有任何保证；甚至没有关于适销性或针对特定用途的适用性的隐含保证。有关更多详细信息，请参阅 GNU Affero 通用公共许可证。

你应该已经收到了一份与本程序一起的 GNU Affero 通用公共许可证副本。如果没有，请参阅 <https://www.gnu.org/licenses/>。

**通过为该项目做出贡献，你同意在相同的许可证下发布你的贡献。**

## 在 LM Studio 中运行 google/gemma-4-31b

LM Studio 是一个强大而友好的桌面应用程序，允许你直接在本地计算机上实验和开发大语言模型。LM Studio 支持 GGUF (llama.cpp) 和 MLX 格式的 Gemma 模型，可在本地机器上实现快速高效的推理。以下是关于如何在 LM Studio 中运行 `google/gemma-4-31b` 的说明。

### 1. 下载并安装 LM Studio

从 [LM Studio 官方网站](https://lmstudio.ai/download) 下载适用于 macOS、Windows 或 Linux 的安装程序，完成下载并运行安装程序。

### 2. 下载 Gemma 4 31B 模型

由于其出色的多模态能力（处理文本、图像等），Gemma 4 模型非常受本地 LLM 用户的欢迎。
要在 LM Studio 中下载并运行：

1. 打开 LM Studio 应用程序。
2. 在 Mac 上按 `Cmd + Shift + M`，或在 PC 上按 `Ctrl + Shift + M` 打开模型下载器。
3. 搜索 "gemma-4-31b"。
4. 从搜索结果中选择看起来合适的变体。LM Studio 会根据你的硬件建议合适的版本（例如量化后的 GGUF 版本）。
5. 点击 **Download**（下载）。

### 3. 加载并运行模型

下载完成后：
1. 前往左侧边栏的对话界面（Chat）。
2. 在顶部的模型选择下拉菜单中，选择你刚刚下载的 `gemma-4-31b` 模型。
3. 根据你的硬件情况（如内存和显存），在右侧面板适当调整上下文长度（Context Length）和硬件设置（Hardware Settings）。
4. 加载完成后，即可开始与 Gemma 4 31B 模型进行本地对话！

### 高级：通过 API 提供模型服务

如果你希望将模型集成到你自己的代码中，可以通过 LM Studio 的本地服务器来提供服务。
在 LM Studio 应用程序中，导航到 **Developer**（开发者）选项卡，并启动本地服务器（Local Server）。启动后，你可以使用 LM Studio 提供的类似 OpenAI 的 REST API 来以编程方式调用 `gemma-4-31b` 模型。更多详情请参阅 [LM Studio 开发者文档](https://lmstudio.ai/docs/developer)。
