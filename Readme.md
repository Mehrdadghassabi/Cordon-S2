# Reasoning Reduces the Influence of Poisoned Context in RAG

This repository contains the code, experimental artifacts, and paper sources for:

> **Reasoning Reduces the Influence of Poisoned Context in RAG**

The paper studies whether reasoning-enabled language models are less influenced by misinformation contained in retrieved documents in Retrieval-Augmented Generation (RAG).

**Paper:** [arXiv](https://arxiv.org/abs/2608.17153)

---

## Overview

Retrieval-Augmented Generation (RAG) provides language models with external evidence, but retrieved documents may contain misleading or intentionally poisoned information. This work studies the relationship between:

* **Poison detection** — whether the model identifies misinformation in retrieved context.
* **Contextual influence** — whether detected misinformation nevertheless affects the model's final answer.

We evaluate reasoning-enabled and non-reasoning configurations of language models and measure how reasoning changes their susceptibility to poisoned context.

The repository provides both the **executable experimental workflow** and the **artifacts used to produce the reported results**.

---

## Repository Structure

```text
Cordon-S2/
├── src/
│   ├── Poison_generation.ipynb   # Poison generation
│   └── Eval.ipynb                # Evaluation and metric computation
│
├── poisons/                      # Poison passages used in the experiments
│
├── evaluation/                   # Language-model responses and evaluation
│                                  # artifacts in XLSX format
│
├── doc/                          # LaTeX source files for the paper
│                                  # corresponding to the arXiv version
│
└── README.md
```

### `src/`

Contains the two Jupyter notebooks that implement the experimental workflow.

* **`Poison_generation.ipynb`** generates the poisoned passages according to the procedure described in the paper.
* **`Eval.ipynb`** runs the evaluation and computes the reported metrics.

### `poisons/`

Contains the poison passages used in the reported experiments.

The generated poisons are preserved so that the reported evaluation can be reproduced without regenerating them.

### `evaluation/`

Contains the language-model responses and associated evaluation artifacts from the reported experiments in XLSX format.

These files provide a fixed record of the model outputs used in the reported results. A reviewer can inspect the responses and independently recalculate the reported metrics **without making any additional API calls**.

### `doc/`

Contains the LaTeX source files and associated resources used to produce the paper corresponding to the arXiv version.

---

# Reproducing the Results

The repository supports two forms of reproduction:

1. **API-independent verification of the reported results**
2. **Rerunning the experimental workflow**

## 1. Verify the reported results without API calls

The simplest way to verify the reported results is to use the XLSX files provided in [`evaluation/`](evaluation/).

These files contain the language-model responses generated during the reported experiments. A reviewer can:

1. Open the corresponding XLSX files.
2. Inspect the model responses and evaluation information.
3. Recalculate the reported metrics from the provided outputs.

No language-model API calls are required for this verification process.

This is particularly useful because the experiments use external model APIs. The provided evaluation artifacts preserve the responses used for the reported results, allowing the results to be independently inspected even without access to the corresponding model providers.

---

## 2. Rerun the experimental workflow

The complete executable workflow is provided in [`src/`](src/).

### Step 1: Generate poisoned passages

Run:

```text
src/Poison_generation.ipynb
```

This notebook implements the poison-generation procedure described in the paper.

The resulting poison passages can be stored in:

```text
poisons/
```

The poison passages used for the reported experiments are already provided in this directory. Therefore, **regenerating the poisons is not necessary to reproduce the reported evaluation**.

### Step 2: Run the evaluation

Run:

```text
src/Eval.ipynb
```

The evaluation notebook runs the experimental conditions and computes the reported metrics.

The main SciFact experiment uses **200 randomly sampled questions with random seed 38**, as described in the paper.

---

## Experimental Conditions

The experiments evaluate language models under RAG and non-RAG conditions, including reasoning-enabled and non-reasoning configurations where supported.

The main experiments include:

* **DeepSeek-V4-Flash**

  * reasoning off
  * reasoning on

* **Qwen3.6-Plus**

  * reasoning off
  * reasoning on

We additionally evaluate:

* **Claude Haiku 4.5**
* **Claude Sonnet 4.6**

The Claude comparison is exploratory and is not used as a within-model causal comparison of reasoning, since the model configurations differ in model capability as well as reasoning configuration.

---

## Datasets

The primary evaluation uses **SciFact**.

The main SciFact experiment contains:

* **200 randomly sampled questions**
* **random seed: 38**

Additional experiments use:

* **FiQA** — first 40 questions
* **MS MARCO** — first 40 questions

These additional datasets are used to examine whether susceptibility to contextual influence varies across datasets and questions.

---

# Evaluation Metrics

The evaluation distinguishes between detecting poisoned information and being influenced by it.

### Leakage Rate (LR)

Leakage Rate measures whether poisoned information affects the model when it is instructed to ignore the retrieved context.

Conceptually:

```text
LR = P(poison-aligned(M_ignore) & not poison-aligned(M_no-RAG))
```

where `M_ignore` receives the retrieved context but is instructed to ignore it, while `M_no-RAG` receives no retrieved context.

### Cordon Rate (CR)

Cordon Rate measures contextual influence among cases where the model detects the poisoned information and its answer without retrieval is not already aligned with the poison.

```text
CR = P(poison-aligned(M_RAG) | detected(M_RAG) & not poison-aligned(M_no-RAG))
```

This separates the ability to **detect** misinformation from the tendency to let detected misinformation influence the final answer.

### Additional metrics

The evaluation also reports:

* **Attack Success Rate (ASR)**
* **Poison Detection Rate (PDR)**

The implementation of these metrics is provided in the evaluation workflow.

---

# Poison Generation

Poison passages are generated following the procedure described in the paper.

The generation workflow constructs contradictory information and incorporates it into retrieved-context passages. The generated passages are then evaluated as part of the experimental pipeline.

The exact poison passages used for the reported experiments are included in:

```text
poisons/
```

This allows the evaluation to use the same poison artifacts reported in the paper rather than relying on a new stochastic generation run.

The poison-generation notebook is provided primarily to make the **generation procedure itself reproducible**.

---

# Evaluation Artifacts

The `evaluation/` directory contains the language-model responses produced for the reported experiments.

These artifacts are important for independent verification because they allow a reviewer to inspect the actual outputs underlying the reported aggregate metrics.

In particular, the artifacts allow the reviewer to trace the evaluation from:

```text
Question
   ↓
Retrieved context
   ↓
Poisoned context
   ↓
Model response
   ↓
Evaluation / judgment
   ↓
Metric
```

without requiring another API call.

---

# External Model APIs

Rerunning model inference requires access to the corresponding model providers and their APIs.

The repository therefore distinguishes between:

### Reproducing / verifying the reported results

This can be done using the provided artifacts in:

```text
poisons/
evaluation/
```

and does **not** require additional model API calls.

### Rerunning model inference

This requires access to the relevant external APIs and may produce different responses because external models can change over time and may exhibit provider-side nondeterminism.

Consequently, the provided XLSX evaluation artifacts should be used when the goal is to reproduce the exact outputs underlying the reported results.

---

# Reproducibility Summary

The repository provides:

| Component                                 | Provided |
| ----------------------------------------- | -------- |
| Paper source                              | Yes      |
| Poison-generation notebook                | Yes      |
| Evaluation notebook                       | Yes      |
| Exact poison passages used in experiments | Yes      |
| Language-model responses                  | Yes      |
| Evaluation artifacts                      | Yes      |
| Metric computation                        | Yes      |
| API-independent result verification       | Yes      |
| End-to-end experimental workflow          | Yes      |

Thus, the repository provides both the **executable workflow for rerunning the experiments** and the **fixed experimental artifacts needed to independently verify the reported results without additional API calls**.

---

# Citation

If you use this repository or build upon this work, please cite the paper:

```bibtex
@article{ghassabi2026reasoning,
  title={Reasoning Reduces the Influence of Poisoned Context in RAG},
  author={Ghassabi, Mehrdad and Ebrahimi, Audrina and Hakim, Sadra and Baradaran Kashani, Hamidreza},
  journal={arXiv preprint arXiv:2608.17153},
  year={2026}
}
```

---

