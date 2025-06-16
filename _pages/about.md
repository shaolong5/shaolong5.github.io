---
permalink: /
title: "Energy Efficient Protein Language Models"
author_profile: true
redirect_from: 
  - /about/
  - /about.html
---

---

summary: How compact language models like Llama-3 and Phi-3, combined with LoRA, can generate high-quality protein sequences at a fraction of the energy cost.
<br>Author: Shaolong Shi<br>Date: 2025-06-16

---

## Introduction

Protein design is one of the most exciting applications of large language models. With the success of AlphaFold and ESMFold in protein structure prediction, attention is turning toward *generation*: Can we design functional proteins from scratch using language models? Traditionally, the models that power this field—like ProGen2 or ESM-3—are massive, power-hungry, and difficult to deploy.

This post explores how two relatively small models—Llama-3-8B and Phi-3-mini—can be adapted using **LoRA (Low-Rank Adaptation)** to generate high-quality protein sequences. Even more excitingly, these models were run on **energy-efficient hardware**, showing that protein generation can be both accurate and sustainable.

## Why Energy Efficiency Matters

Training and running large protein models consumes enormous GPU hours. While feasible for big labs, this creates a barrier for smaller institutions and increases the environmental cost of AI research. Our goal is to reduce both training cost and inference power consumption—without sacrificing performance.

We used LoRA to fine-tune pretrained models on protein data, reducing trainable parameters to just 4%. These models were then evaluated not only on traditional benchmarks like TM-Score and pLDDT but also on energy metrics like TPS/W (tokens per second per watt).

In high-throughput biological scenarios—such as enzyme screening or rapid synthetic design—power-efficient inference becomes crucial. Having the ability to run a compact model on a mobile or edge device, or even an embedded RISC-V chip, opens up new use cases in resource-constrained environments (e.g., portable biosensors or field labs).

## Model Setup

We tested two models:

- **Llama-3-8B**: A recent open-source foundational model from Meta.
- **Phi-3-mini**: A lightweight model designed for instruction-following.

Both were fine-tuned using LoRA on protein sequences and property-conditioned prompts. Fine-tuning was done in two stages:

1. **Unconditional generation** ("make me a protein").
2. **Controllable generation** ("make me a SAM-MT enzyme").

We followed continual learning—starting from the pre-trained NLP weights and incrementally adapting to the protein domain. This retained language-understanding capabilities, allowing natural prompt-driven protein generation.

All training was performed on a single NVIDIA A100 GPU using 16-bit precision, and deployment was tested on both the A100 and **Esperanto’s ET-SoC-1 chip**, a low-power RISC-V SoC designed for on-device AI.

## Dataset: Small But Rich

We used \~2M sequences from UniRef50 for unconditional training, and \~800K sequences across 10 protein classes for controllable generation. These classes include:

- 6 enzyme types (e.g., Transferase, Ligase, Oxidoreductase)
- 4 structural protein superfamilies (e.g., SAM-MT, TPHD)

Each class presents distinct sequence lengths, complexity, and structure. For example, ligases tend to have longer sequences (\~465 amino acids), while TPHD proteins are shorter (\~170).

| Class          | # Sequences | Avg Length |
| -------------- | ----------- | ---------- |
| SAM-MT         | 195,820     | 188        |
| TRX            | 129,686     | 159        |
| TPHD           | 141,034     | 170        |
| CheY           | 133,126     | 152        |
| Oxidoreductase | 23,901      | 343        |
| Transferase    | 65,899      | 336        |
| Hydrolase      | 35,758      | 323        |
| Lyase          | 18,550      | 325        |
| Isomerase      | 12,151      | 342        |
| Ligase         | 24,010      | 465        |

We chose these classes to evaluate both structural consistency and biological diversity.

## Evaluation: Structure First

Unlike text generation, proteins need to fold into viable 3D structures. We used the following tools:

- **ESMFold**: Predicts 3D structure and computes pLDDT (confidence score).
- **Foldseek**: Aligns generated structures to known proteins, reporting TM-Score (similarity) and RMSD (distance).

For controllable generation, we filtered results to retain only high-confidence predictions (pLDDT > 60). This ensured structural validity before scoring with TM-Score.

On average, Llama-3 generated longer, more ordered sequences, while Phi-3 focused better on shorter, property-aligned designs.

## Results

### 🎯 Unconditional Generation

- **Llama-3 (A100)**: `pLDDT = 69.75 ± 12.74`
- **Phi-3 (A100)**: `pLDDT = 60.41 ± 4.56`

These scores exceeded those of ProGen2 and ProLLaMA, with the Llama-3 variant approaching the lower bound of natural protein stability.

Llama-3 also achieved better RMSD metrics on AFDB alignments, indicating tighter structural similarity despite no prompt-based conditioning.

### 🎯 Controllable Generation (TM-Score)

| Class     | Llama-3 | Phi-3 |
| --------- | ------- | ----- |
| TPHD      | 0.95    | 0.80  |
| TRX       | 0.97    | 0.86  |
| SAM-MT    | 0.83    | 0.79  |
| Isomerase | 0.81    | 0.86  |
| Hydrolase | 0.79    | 0.81  |

Phi-3 occasionally outperformed Llama-3 in classes with shorter or simpler structures, suggesting better overfitting under smaller capacity.

Classes like Oxidoreductase and Ligase remained more difficult due to long sequence lengths and inherent variability.

### ⚡ Inference Efficiency

| Metric    | Llama-3 A100 | Llama-3 ET-SoC | Phi-3 A100 | Phi-3 ET-SoC |
| --------- | ------------ | -------------- | ---------- | ------------ |
| Power (W) | 300          | 25             | 300        | 25           |
| TPS       | 36           | 5              | 36         | 10           |
| TPS/W     | 0.12         | 0.20           | 0.12       | **0.40**     |

The energy efficiency of ET-SoC-1 is striking: 3x better TPS/W for Phi-3 compared to A100, despite reduced absolute speed.

This makes the model viable for sustainable deployments in low-power environments (e.g., portable sequencing stations or educational use).

## Web Demo

We built a [Gradio ](https://huggingface.co/Esperanto/Protein-Phi-3-mini)[**demo**](https://huggingface.co/Esperanto/Protein-Phi-3-mini) that allows users to:

- Choose a starting amino acid motif (e.g., `MGG...`)
- Select a property (e.g., `Hydrolase`)
- Generate & visualize the structure in real-time

The UI also reports performance metrics, including generation time and pLDDT scores.

## Takeaways

- ✅ **LoRA works well for proteins**: Only 4% of parameters fine-tuned, yet strong generalization
- ⚡ **ET-SoC-1 is energy efficient**: 25W power budget vs. 300W for A100
- 🧬 **Controllable generation is feasible**: Prompt conditioning enables property-specific design
- 🧠 **Small models can learn protein language**: Even Phi-3 performs surprisingly well

## What's Next?

We see several directions for future work:

- 🧪 **Experimental synthesis**: Collaborate with wet labs to synthesize top sequences
- 🔬 **Multi-modal input**: Add structure + function constraints during training
- 🧱 **Robustness testing**: Evaluate mutation resistance and stability under perturbation
- 🌍 **Open access tools**: Make sustainable protein generation accessible to all

By lowering the cost of protein design, we open the door to more inclusive, scalable biology.

---

*Code and models available on **[Hugging Face](https://huggingface.co/Esperanto)**.*

*Paper: Shah & Jayaratnam (2024), "Energy Efficient Protein Language Models," arXiv:2411.05966*

