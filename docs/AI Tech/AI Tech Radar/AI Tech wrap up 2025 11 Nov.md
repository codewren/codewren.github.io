# AI wrapup 2025 Nov

# GPT Teacher — 从 0 到 1 在 CPU 上训练可演示的小参数 GPT
https://github.com/helloworldtang/GPT_teacher-3.37M-cn

本项目面向课堂教学，[目标是让初学者用一台普通 CPU 电脑，在 45 分钟内从零跑通一个小参数的中文 GPT](https://mp.weixin.qq.com/s/gUgY_TIRSoEyzZ2YRVGB2w)：看清核心流程、跑通训练、得到“可用的中文回答”，并支持简单的推理演示。

## 项目收获

- 了解 GPT 的核心原理：分词 → 批处理 → 前向 → 损失 → 反向 → 优化 → 保存 → 推理（完整链路）
- 掌握高效小模型技术：RMSNorm、RoPE、权重共享、短序列、小词表、量化
- 学会仅用 CPU 训练：控制模型/数据规模、梯度累积、学习率预热与退火、禁用无关耗时特性
- 学会可用答案保障：目标对齐（忽略前缀）、推理首步约束与后处理、停止词、提示词规范化

## 代码结构

- `src/model.py`：GPT 核心（Embedding/自注意力/前馈/RMSNorm/RoPE/权重共享）
- `src/data.py`：分词器加载、指令和 LM 数据处理、批处理与目标构造
- `src/train.py`：CPU 训练主循环、评估、保存与动态量化、耗时统计
- `src/infer.py`：推理 CLI（温度、top-k、top-p、重复惩罚、停止词、输出清理）
- `src/build_tokenizer.py`：构建 HF ByteLevel BPE 中文分词器（带解码器，避免乱码）
- `config.yaml`：统一管理模型、训练、数据、分词器与保存路径
- `data/*.jsonl`：教学数据（`{"prompt": "...", "completion": "..."}`）

## 使用技术

- 模型结构：Decoder-only Transformer（因果自注意力）
- 归一化：RMSNorm（简洁高效）
- 位置编码：RoPE（相对位置，计算高效）
- 前馈：SiLU（现代 LLM 常用）
- 权重共享：词嵌入与输出层共享，降参数且表示一致
- 训练：AdamW、学习率线性预热+余弦退火、梯度裁剪
- 数据与分词：外置`jsonl`+HF ByteLevel BPE（显式设置 ByteLevel 解码器，避免乱码）
- 推理：温度/top-k/top-p 采样、重复惩罚、停止词、输出清理、提示规范化
- 量化：导出动态量化权重以加速 CPU 推理

## 如何仅用 CPU 训练

- 小模型+短序列：`n_layer=4, n_head=4, n_embd=256, seq_len=128`
- 小词表：HF ByteLevel BPE（带解码器）
- 梯度累积：`batch_size=16, micro_batch=4`（有效批 64）
- 学习率策略：线性预热 10%+余弦退火
- 资源控制：`torch.set_num_threads(os.cpu_count())`，DataLoader 禁用多进程（macOS 下更稳）
- 训练结束导出量化权重，演示更流畅

-------


# Foundations of Large Language Models
https://arxiv.org/abs/2501.09223

```
@misc{xiao2025foundationslargelanguagemodels,
      title={Foundations of Large Language Models}, 
      author={Tong Xiao and Jingbo Zhu},
      year={2025},
      eprint={2501.09223},
      archivePrefix={arXiv},
      primaryClass={cs.CL},
      url={https://arxiv.org/abs/2501.09223}, 
}
```