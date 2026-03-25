---
title: Paper Review - OneTrans
date: 2026-03-22 14:57:10
tags: Recommendation, Ranking
categories: 
    - Paper Review
mathjax: true
cover: OneTransStack.png
---
# Paper Overview
| Dimension | Description |
|:---------|:--------|
| Title | [OneTrans: Unified Feature Interaction and Sequence Modeling with One Transformer in Industrial Recommender](https://arxiv.org/pdf/2510.26104) |
| Authors | Zhaoqi Zhang et al. (ByteDance) |
| Year | 2026 |
| Domain | RecSys |
| Task | Ranking (CTR / CVR prediction) |
| Keywords | - unified Transformer <br> - sequence modeling <br> - feature interaction modeling <br> - tokenization <br> - shared representation |

# TL; DR
## Problem
Prior work follows an **encode-then-interaction** paradigm that separate sequence modeling from feature interactions. <br>
This leads to:
- limited cross-feature interaction during sequence modeling
- suboptimal representation learning due to decoupled processing
- fragmented execution and higher latency
## Core idea
OneTrans unifies sequential and non-sequential features into a **single token space**, enabling joint modeling via a shared Transformer. <br>
**Key Contributions**
1. A **unified framework** for joint sequence modeling and feature interaction.
2. **Mixed parameterization** for recommender-specific customization.
3. Efficiency improvement via **pyramid strategy** and **cross-request KV caching**.
4. Empirical evidence of scaling behavior (log-linear performance gains with increased model size).
## Why it matters
- Enables richer interactions and joint representation learning across all features. 
- Enables end-to-end optimization.
- Aligns RecSys models with LLM scaling trends, making them more compatile with LLM-style architectures and optimizations.

# Problem Definition
| Dimension | Description |
|:---------|:--------|
| Input | - Sequential features: user behavior/action sequence <br> - Non-sequential features: <br> &nbsp;&nbsp;&nbsp; -- numerical features: CTR, price, dwelltime stats, etc. <br> &nbsp;&nbsp;&nbsp; -- categorical featurfes: user id, target item id, surface, device, etc. |
| Output | Multi-task output for CTR and CVR prediction |
| Objective | Combined cross-entropy loss for CTR and CVR |

# Model
{% asset_img Architecture_OneTrans.png %}
## Pipeline Overview
- Convert all inputs (tabular + sequences) into tokens
- Align them into a unified sequence
- Use a Transformer with **mixed parameterization**:
  - shared weights for sequential tokens
  - token-specific weights for non-sequential features
- Improve efficiency via **pyramid pruning**

## Tokenization
| Terminology | Description | Comments |
|:---------|:--------|:--------|
| $L_{NS}$ | number of non-sequential NS tokens | predefined hyperparameter |
| d | hidden dimension of all tokens | predefined hyperparameter |
| $L_S$ | number of sequential S tokens | $L_S = \sum_iL_i + L_{SEP}$ <br> where, <br> &nbsp;&nbsp;&nbsp; - $L_i$: number of events of behavior type $i$ <br> &nbsp;&nbsp;&nbsp; - $L_{SEP} = 0$ for time-aware merge |
| $L_i$ | **number of events in behavior type $i$** | |
| $e_{ij}$ | raw embedding of $j^{th}$ event in behavior type $i$ <br> &nbsp;&nbsp;&nbsp; - $i \in [1,n]$: behavior/action type <br> &nbsp;&nbsp;&nbsp; - $j \in [1,L_i]$: event index within that behavior

### Non-sequential Tokens (NS)
**Step1: Embed categorical features**
Each categorical features is mapped to an embedding:
Example:
- user id → 64 embedding
- itemid → 128 embedding
- device type → 6 embedding

Numerical features remain as raw scalars:
- CTR → 1
- price → 1

**Step2: Concatenate all features**
Concatenate numerical features and categorical embeddings into a single vector.
Example:
- total dimension: 
    $$
    D = 64 + 128 + 6 + 1 + 1 = 200
    $$

**Step3: Project to hidden space via MLP**
Apply an MLP to map the concatenated vector into a higher-dimensional representation
    $$
    MLP(concat(NS)) \in R^{L_{NS} \times d}
    $$

**Step4: Split into tokens**
$$
Token_{NS} = [t_1, t_2, ..., t_{L_{NS}}], \quad t_i \in R^d
$$
Example:
- $Token_{NS} = [R^{128}, R^{128}, R^{128}, R^{128}]$
---

### Sequential Tokens (S)
**Step1: Behavior-specific sequences**
Each behavior type forms its own sequence:
$$
S_i = [e_{i1}, e_{i2}, ..., e_{iL_i}] \in R ^{L_i × d_i}
$$

Examples: 
- click event: $e_{click} = concat(emb_{itemid}, emb_{item category}, dwelltime, click position)$
- purchase event: $e_{purchase} = concat(emb_{itemid}, price, discount, emb_{payment method}, quantity)$

**Step2: Per-behavior projection**
$$
\hat{S_i} = [MLP_i(e_{i1}), MLP_i(e_{i2}), ..., MLP_i(e_{iL_i})] \in R ^ L_i × d
$$
- Each behavior type has its own projection $MLP_i$
- Output dimension is predefined $d$

**Step3: Time-aware merge**
$$
Token_{S} = [token_{ij}]
$$
$$
token_{ij} = MLP_i(e_{ij}) + ActionTypeEmb(i) + TimeEmb(t_{ij})
$$
- $i$: behavior type
- $j$: event index
- Add:
    - action type embedding
    - timestamp embedding
---

### Unified Token Space
Final input to Transformer:
$$
X^{0} = [Token_S; Token_{NS}] \in R^{(L_S + L_{NS} \times d)}
$$

## Transformer Modeling
### OneTrans Block
$$
Z^{(n)} = MixedMHA(RMSNorm(X^{(n-1)})) + X^{(n-1)}
$$
$$
X^{(n)} = MixedFFN(RMSNorm(Z^{(n)})) + Z^{(n)}
$$
### Key Design: Mixed Parameterization
- S tokens:
    - Share Q/K/V and FFN parameters
- NS tokens:
    - Token-specific parameters
### Causal Attention
- Tokens attend only to previous tokens
- NS tokens appended affter S tokens

Implication:
- S tokens unaffected by NS tokens
- Enable KV caching for S tokens

### Efficiency: Pyramid Stack
- Only recent S tokens issue queries
- Key/values still from full history

Benefits:
- Long sequence compression
- Reduced complexity

## Training Objectives
$$
CTR: P(click | NS, S)
$$
$$
CVR: P(conv | click, NS, S)
$$

# Takeaways
