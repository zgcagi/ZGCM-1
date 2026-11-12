<div align="center">
<img src="assets/zgcm1-logo.png" width="480" alt="ZGCM-1">

**A Fully Open and Extremely Efficient Foundation Model for Math and Agentic Search**

Zhongguancun Academy · Zhongguancun Institute of Artificial Intelligence

<p align="center">
  <a href="https://arxiv.org/abs/2609.13356">📄 Tech Report</a>
  &nbsp;·&nbsp;
  <a href="https://huggingface.co/zgcagi/ZGCM-1-7B"><img src="assets/icons/huggingface.svg" width="18" height="18" alt=""> Model</a>
  &nbsp;·&nbsp;
  <a href="https://huggingface.co/datasets/zgcagi/ZGCM-1-Data"><img src="assets/icons/huggingface.svg" width="18" height="18" alt=""> Data</a>
  &nbsp;·&nbsp;
  <a href="https://wandb.ai/zgcagi/zgcm-1-reports/reports/ZGCM-1-Training-and-Evaluation--VmlldzoxNzk1OTMxMg"><img src="assets/icons/wandb.svg" width="18" height="18" alt=""> Training Log</a>
</p>

<p align="center">
  <a href="#evaluation-results">📊 Results</a>
  &nbsp;·&nbsp;
  <a href="https://github.com/zgcagi/ACE">🎯 ACE</a>
  &nbsp;·&nbsp;
  <a href="#getting-started"><img src="assets/icons/github.svg" width="18" height="18" alt=""> Training Code</a>
  &nbsp;·&nbsp;
  <a href="#wechat-community">💬 WeChat Community</a>
</p>

</div>

## Introduction

**ZGCM-1** is a **7.39B-parameter dense language model trained from scratch**, built for mathematical reasoning and tool-assisted search. It combines deliberate internal thinking with active information gathering, supporting **256K-token context** and both thinking and direct-response modes in a single model.

The project brings together an efficient hybrid-attention architecture, FP8 training with Muon, progressive long-context mid-training, and general-agentic supervised fine-tuning. Researcher-directed AI agents contribute throughout development, from data curation and cluster operations to evaluation and deployment.

![ZGCM-1 overview: benchmark performance and key technical components](assets/zgcm-1-overview.png)


## Highlights

- **Math and reasoning at 7B scale.** ZGCM-1 achieves 97.13% on MATH-500, 75.00% on AIME 2026, and 70.42% on HMMT 2025, with the best average rank across the report's 14 reasoning benchmarks among the seven compared 7B–8B models.
- **Search beyond model memory.** Multi-step tool use reaches 63.09% on WebWalkerQA, 19.43% on BrowseComp, and 62.00% on Binary Function Search.
- **Efficient long context.** Gated sliding-window and global attention deliver 3.94× training throughput at 256K compared with full attention in the report's architecture experiments. The report estimates an approximately 4.2× improvement in 16K pretraining time-to-loss from combined architecture, precision, optimizer, and normalization gains.
- **An open research recipe.** This repository provides data-processing, pretraining, mid-training, and SFT workflows, with stage-specific configurations and runtime documentation. Explore the [model and data](#resources), or get started with the [training workflows](#getting-started).

## Evaluation Results

The following results are from the technical report, using the **256K SFT checkpoint in thinking mode**. Non-agentic evaluations use temperature 1.0, top-p 1.0, and mean pass@1 over 32 runs unless otherwise specified.

### Selected reasoning benchmarks

![ZGCM-1 per-benchmark ranks across 14 reasoning benchmarks compared with six other 7B–8B models](assets/zgcm-1-reasoning-ranks.png)


| Benchmark (%) | ZGCM-1 | DeepSeek-R1-0528-Qwen3-8B | MiniCPM4.1-8B | Qwen3-8B | Olmo 3 7B Think |
| --- | ---: | ---: | ---: | ---: | ---: |
| MATH-500 | **97.13** | 96.32 | 95.60 | 96.20 | 95.10 |
| AIME 2024 | 80.62 | **83.33** | **83.33** | 80.00 | 71.60 |
| AIME 2025 | 73.33 | **75.21** | 73.33 | 63.33 | 64.60 |
| AIME 2026 | **75.00** | 69.17 | 71.67 | 66.67 | 66.16 |
| HMMT 2025 | **70.42** | 61.50 | 52.50 | 43.33 | 43.89 |
| HMMT 2026 | **59.48** | 51.52 | 46.21 | 45.45 | 43.94 |

*Selected rows and models from Table 2; bold marks the best score in each displayed row. The full evaluation covers 20 benchmarks, including code, knowledge, and instruction following.*

### Agentic search

| Benchmark | ZGCM-1 (%) | Setting |
| --- | ---: | --- |
| WebWalkerQA | 63.09 | Web search and page reading |
| BrowseComp | 19.43 | Web search and page reading |
| GAIA (text-only) | 42.52 | Web search and page reading |
| Binary Function Search | 62.00 | 31/50 exact function-entry matches using Ghidra tools |

*Source: Tables 3–4. Web research allows up to 64 search-and-read steps. Binary Function Search uses a separate protocol on 50 tasks from 10 held-out projects.*

## Architecture and Training

![ZGCM-1 hybrid attention architecture with gated sliding-window GQA and global attention](assets/zgcm-1-architecture.png)


| Model specification | ZGCM-1 |
| --- | --- |
| Architecture | Decoder-only dense Transformer |
| Parameters | 7.39B |
| Layers / hidden size | 32 / 4,096 |
| Attention | 27 gated sliding-window layers + 5 global layers |
| Local window / GQA heads | 128 tokens / 32 query heads, 8 KV heads |
| Maximum context | 262,144 tokens (256K) |

The training recipe described in the report has three main stages:

1. **Pretraining:** approximately 4.19T tokens, combining curriculum-based data mixing with hybrid FP8 precision and Muon optimization.
2. **Mid-training:** approximately 600B tokens with context extended from **16K → 64K → 256K**. Interaction traces are reformulated as Markov Decision Process (MDP) state-action transitions to supervise individual decisions.
3. **Supervised fine-tuning:** joint general and agentic training with mixed thinking/direct-response examples, execution-verified trajectories, and assistant-only loss. The report also explores mixed RL for mathematics and code.

## Resources

| Resource | Location |
| --- | --- |
| Model weights | [ZGCM-1 model weights](https://huggingface.co/zgcagi/ZGCM-1-7B) |
| Data | [zgcagi/ZGCM-1-Data](https://huggingface.co/datasets/zgcagi/ZGCM-1-Data) |
| Data and training workflows | Stage directories below |

For model and dataset usage, see the corresponding Hugging Face cards. Model specifications, evaluation details, and figures are presented in *ZGCM-1: A Fully Open and Extremely Efficient Foundation Model for Math and Agentic Search*.

## Getting Started

Choose the workflow you want to reproduce and follow its setup instructions. Each top-level directory contains its own code, configurations, and runtime documentation.

| Directory | Area | Runtime documentation |
| --- | --- | --- |
| [`data-process/`](data-process/) | AI-native data governance | [`data-process/README.md`](data-process/README.md) |
| [`pretrain/`](pretrain/) | Pretraining | [`pretrain/README.md`](pretrain/README.md) |
| [`midtrain/`](midtrain/) | Mid-training | [`midtrain/README.md`](midtrain/README.md) |
| [`sft/`](sft/) | Supervised fine-tuning | [`sft/README.md`](sft/README.md) |
| [`rl/`](rl/) | Reinforcement learning | [`rl/README.md`](rl/README.md) |

Enter the relevant directory and follow its README to prepare the environment, data, tokenizer, and checkpoints, then launch the workflow with the provided configuration.

## WeChat Community

Scan the QR code to join the ZGCM-1 community group. Click the image to open it
at full size. If the code has expired, open an Issue and ask the maintainers for
the latest one.

<p align="center">
  <a href="assets/zgcm-1-wechat-group-4.jpg">
    <img src="assets/zgcm-1-wechat-group-4.jpg" width="360" alt="ZGCM-1 WeChat Group 4 QR code">
  </a>
</p>

## License

The repository is distributed under the [MIT License](LICENSE). See the stage directories for bundled third-party licenses and notices, and the model and dataset cards for their respective terms.

## Citation

If you find ZGCM-1 useful in your research, please cite our technical report:

```bibtex
@misc{zgcm1,
  title={ZGCM-1: A Fully Open and Extremely Efficient Foundation Model for Math and Agentic Search},
  author={Jiyan He and Guang Liang and Hao Liu and Haoxiang Guan and Jinbo Sun and Junyi Guo and Wenjun Feng and Yantai Xie and Yifei Shen and Bin Shao and Chuyang Wei and Kai Chen and Kexin Zhou and Minghang Zhu and Shuxin Zheng and Tie-Yan Liu and Taine Zhao and Wenhui Zhu and Xueyin Xu and Xiaoqing Zhang and Yatao Li and Yuxuan Ren},
  year={2026},
  eprint={2609.13356},
  archivePrefix={arXiv},
  primaryClass={cs.AI},
  url={https://arxiv.org/abs/2609.13356}
}
```
