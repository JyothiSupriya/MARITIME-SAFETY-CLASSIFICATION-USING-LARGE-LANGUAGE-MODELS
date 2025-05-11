# MARITIME-SAFETY-CLASSIFICATION-USING-LARGE-LANGUAGE-MODELS
A research-based evaluation of nine locally hosted Large Language Models (LLMs) for classifying maritime safety near-miss incidents into 50 expert-defined categories, using 5,000 binary decisions across 100 real-world narratives.

---

## 📖 Project Overview

This repository contains all code, data and analysis for my Master’s thesis in which we evaluate nine locally-hosted Large Language Models (LLMs) on a zero-shot binary classification task over maritime “near-miss” incident narratives. We ask 50 human-defined yes/no questions of 100 sampled records (5000 Record–Question pairs), run each model three times, aggregate by majority vote, and compare against human annotations using:

- Confusion matrices  
- Precision, recall & F₁-scores  
- Cohen’s κ (agreement beyond chance)  
- McNemar’s test (statistical significance between model pairs)  

---

- **Feasibility** of zero-shot LLM classification in a safety-critical domain  
- **Consistency** of model outputs across repeated runs  
- **Alignment** of LLM decisions with human annotations  
- **Statistical rigor** via confusion matrices, F₁-scores, Cohen’s κ and McNemar’s tests

## 🎯 Motivation

- **Manual bottleneck**: Traditional analysis of incident reports is labor-intensive.
- **Domain need**: Near-miss reporting underpins maritime safety improvements—automated categorization can accelerate insights.  
- **Research gaps**: Prior work rarely integrates multiple statistical metrics or examines run-to-run variability in LLM outputs. This study bridges those gaps.

---
## 🗂️ Data & Preprocessing

1. **Source Corpus**  
   – 100000-entry Maritime Safety Near-Miss Dataset (To protect sensitive operational details, the full text of each narrative is withheld—only masked Record_IDs ).  
2. **Sample Selection**  
   – Randomly sampled **100 incident descriptions** (“Incident Description After Resolution” only).  
3. **Topic Taxonomy**  
   – Manually defined **50 generic safety topics** (e.g. “faulty equipment condition,” “communication failure,” etc.).  
4. **Record–Question Matrix**  
   – Paired each of the 100 narratives with all 50 topics → **5 000 binary decision points**.  
5. **Text Cleaning**  
   – Removed special characters/extra whitespace, preserved original phrasing (no stemming/lemmatization).  
6. **Human Annotation**  
   – Human labeled every pairing as **Yes/No** for ground truth.


## 🤖 Models Evaluated

All models were used in zero-shot mode (no fine-tuning), accessed via `ollama run <model>`:

| Model                  | Size         | Source / Notes                             |
|------------------------|--------------|--------------------------------------------|
| **GEMMA 7B**           | 7 B params   | Google open-source, reasoning-optimized    |
| **LLaMA 3.2**          | 3 B params   | Meta AI, instruction-following improvements|
| **MISTRAL 7B**         | 7 B params   | Mistral AI, performance-tuned              |
| **DeepSeek 14B**       | 14 B params  | Technical & scientific focus               |
| **PHI 4**              | 14 B params  | Microsoft research model                   |
| **Qwen 32B**           | 32 B params  | Large knowledge representation             |
| **GEMMA 27B**          | 27 B params  | Google, advanced reasoning                 |
| **DeepSeek 32B**       | 32 B params  | Largest DeepSeek variant                   |
| **LLaMA 3.3 70B**      | 70 B params  | Meta AI, highest capacity in LLaMA family  | 

---

## 🔬 Methodology & Pipeline

1. **Prompting & Parsing**  
   - Notebook: `Multi_LLM_Binary_Evaluator.ipynb`  
   - For each (Record, Question) pair and each model:  
     - Prompt:  
       ```
       You are a safety expert. Evaluate the following record and answer 
       the question with only one word: "Yes" or "No".
       ```
     - Run **three independent times** to capture stochastic variation.  
     - Parse each raw output to `"Yes"` or `"No"` (error/fallback on parse failures).  
2. **Majority-Vote Aggregation & Masking**  
   - For each model, take the majority label across three runs.  
   - Replace confidential narratives with `Record_1…Record_100` IDs.  
   - Commit only `Masked_Majority_Vote_Responses.xlsx` (5000 rows × 12 columns).  
3. **Statistical Evaluation & Visualization**  
   - Notebook: `Evaluation_Metrics.ipynb`  
   - Load masked Excel → for each model vs. human:  
     - Compute confusion matrix, precision, recall, F₁-score.  
     - Calculate Cohen’s κ for inter-rater agreement.  
     - Perform McNemar’s test on paired yes/no decisions.  
   - Generate figures:  
     1. **F₁-Scores by Model & Class** (No vs. Yes)  
     2. **Aggregated Confusion Matrices**  
     3. **Cohen’s κ Bar Chart**  
     4. **McNemar’s p-value Heatmap**

---
---

## 🛠 Prerequisites

See [Prerequisites.md](Prerequisites.md) for system-level setup:  
- Install Ollama CLI  
- Start the Ollama server  
- Pull each LLM (Gemma-7B, LLaMA-3.2, Mistral-7B, DeepSeek-14B, Phi-4, Qwen-32B, Gemma-27B, DeepSeek-32B, LLaMA-3.3-70B)


## 🗂️ Repository Contents

| File                                      | Description                                                                                 |
|-------------------------------------------|---------------------------------------------------------------------------------------------|
| `Multi_LLM_Binary_Evaluator.ipynb`        | Prompts all nine LLMs, parses three-run outputs, saves raw “Yes/No” labels per run.         |
| `Masked_Majority_Vote_Responses.xlsx`     | Dataset with 5000 rows:  
|                                           | • `Record_ID` (Record_1…Record_100)  
|                                           | • `Question` (50 predefined safety topics)  
|                                           | • `LLM Response_<model>` (final majority-vote label)  
|                                           | • `Human Response` (Human ground truth)                                                     |
| `Evaluation_Metrics.ipynb`                | Loads the masked Excel, computes & visualizes: confusion matrices, F₁, Cohen’s κ, McNemar.  |
| `Prerequisites.md`                        | System-level setup: Ollama CLI install, server & model pulls.                               |
| `Requirements.txt`                        | Python dependencies (`pandas`, `scikit-learn`, `statsmodels`, `tqdm`, `matplotlib`, etc.)   |
| `README.md`                               | This document.                                                                              |

---

## 📈 Key Results

- **Overall accuracy** spanned **83%** (LLaMA 3B) to **95.5%** (LLaMA 70B).  
- **Top performers**:  
  - **LLaMA 70B**: F₁-Yes = 0.65, Cohen’s κ = 0.625 (best)  
  - **PHI 4**:   F₁-Yes = 0.59, κ = 0.566 (second)  
- **Lowest**:  
  - **LLaMA 3.2**: F₁-Yes = 0.61, κ = 0.272  
- **Class imbalance**: “No” class (4 778/5 000) achieved much higher F₁ than “Yes” (222/5 000), highlighting detection challenges for rare positive cases.  
- **Significance testing** (McNemar’s): most model pairs differed significantly, except:  
  - MISTRAL 7B vs DeepSeek 14B (p = 1.000)  
  - GEMMA 7B vs GEMMA 27B (p = 0.314)  
  - GEMMA 7B vs DeepSeek 32B (p = 0.477)  
  - DeepSeek 32B vs GEMMA 27B (p = 0.097)

---

## 🔮 Conclusions & Future Work

- **Larger models** generally outperform smaller ones in both accuracy and agreement with human annotation.  
- **Minority detection** remains the hardest challenge; future work could explore class-aware prompting, data augmentation, or fine-tuning.  
- **Prompt engineering** and **ensemble methods** may further boost reliability in safety-critical contexts.  
- **Human-AI collaboration frameworks** should guide when to defer ambiguous cases to experts.

