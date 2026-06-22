# 15 分钟在 AMD ROCm 上部署通用 Physical AI

> 📍 **新手？** 请查看 [**RUNNING_THIS_TUTORIAL.md**](RUNNING_THIS_TUTORIAL.md)，了解在哪里以及如何运行本教程 —— 如何获取免费的 AMD GPU 算力（compute hours）并在 Radeon Cloud 上启动 notebook。

![Preface](images/preface.png)

*π0.5 是用于快速配置和首次推理（first inference）的示例策略*

[![ROCm](https://img.shields.io/badge/ROCm-7.x-00A3E0?logo=amd)](https://rocm.docs.amd.com/)
[![PyTorch](https://img.shields.io/badge/PyTorch-2.9+-EE4C2C?logo=pytorch)](https://pytorch.org/)
[![Python](https://img.shields.io/badge/Python-3.12+-3776AB?logo=python)](https://www.python.org/)
[![Notebook](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter)](https://jupyter.org/)
[![LeRobot](https://img.shields.io/badge/LeRobot-pi05-111111)](https://github.com/huggingface/lerobot)

本教程旨在快速让一个通用 Physical AI 模型在 AMD ROCm 上跑起来。
它以 π0.5 作为示例策略（policy），随后验证 GPU 运行时（runtime）、运行一次合成推理（synthetic inference），并为自定义数据集准备一条 finetuning 命令。

## 为什么要做这个

目标是让 world-model / vision-language-action 工作流在 ROCm 上可用，而不把 notebook 变成一个脆弱的一次性 demo。

实际上，这意味着 notebook 只覆盖对真正部署 Physical AI 模型至关重要的部分：

- 在 AMD 硬件上部署通用 Physical AI / VLA 模型
- 验证 ROCm、BF16 和 PyTorch/HIP 是否正确配置
- 在本地加载 π0.5 权重（weights）作为示例 checkpoint
- 在 finetuning 之前证明前向传播（forward pass）可以工作
- 让 finetuning 步骤保持显式且需主动开启（opt-in）

## Notebook 做了什么

notebook 有意构建为一个带 checkpoint 的逐步演练（walkthrough）：

1. 验证 ROCm 和 Python
2. 安装 LeRobot 及配套包
3. 下载或验证 π0.5 权重
4. 运行一次 dummy 推理
5. 打印一条 finetuning 命令模板
6. 汇总 `tunableop_results0.csv` 作为验证产物（validation artifact）

训练**不会**自动开始。只有当你设置 `run_training = True` 时，notebook 才会启动 finetuning。

## 文件

- [`notebooks/pi0.5_rocm_tutorial.ipynb`](./notebooks/pi0.5_rocm_tutorial.ipynb) - 端到端 ROCm 教程 notebook
- [`notebooks/tunableop_results0.csv`](./notebooks/tunableop_results0.csv) - 由 TunableOp/ROCm 调优产生的后端验证产物（backend validation artifact）

## 预期输出

当 notebook 正常工作时，你应当看到：

- `torch.cuda.is_available()` 返回 `True`
- ROCm 报告一块 gfx1100 级别的 AMD GPU
- BF16 支持已启用
- `lerobot[pi]` 成功导入
- `PI05Policy` 从本地权重加载
- 一次 dummy 前向传播返回形如 `torch.Size([1, 50, 32])` 的 action chunk
- 一条 finetuning 命令打印到屏幕上，而不会自动启动

对于 TunableOp CSV，预期内容为：

- 描述 `PT_VERSION`、`HIP_VERSION`、`HIPBLASLT_VERSION` 和 `GCN_ARCH_NAME` 的元数据字段
- 用于 TunableOp/GEMM 验证的 benchmark 行
- 可汇总为区间和均值的数值计时（timing）值

## 产物的含义

`tunableop_results0.csv` 可作为后端的健全性检查（sanity check），但它本身并不是 finetuning 的结果。

将它视为以下事项的证据：

- ROCm 调优已运行
- 后端已发出 benchmark 测量值
- 环境能够完成一组有代表性的 GEMM/TunableOp 检查

除非 notebook 同时以 `run_training = True` 运行训练单元（training cell），否则不要把它当作 finetuning 已完成的证明。

## 快速开始（Quick Start）

1. 在 Jupyter 中打开 [`notebooks/pi0.5_rocm_tutorial.ipynb`](./notebooks/pi0.5_rocm_tutorial.ipynb)。
2. 按顺序运行各单元（cells）。
3. 确认环境、依赖安装和权重加载步骤通过。
4. 运行 dummy 推理单元。
5. 用你的数据集编辑 finetuning 模板，然后再启用训练。

## 备注

- 本 notebook 面向 RDNA3 级别硬件上的 AMD ROCm。
- 本教程使用 `cuda` API，因为在 ROCm 上 PyTorch 通过同一套接口暴露 HIP。
- CSV 文件可通过运行 notebook 或相关调优步骤重新生成；应将其视为产物（artifact），而非手工编辑的报告。

## 许可证（License）

请遵循上游模型以及 notebook 中所用各软件包的许可条款。如果你公开发布本仓库，请单独核实模型权重的再分发（redistribution）条款。
