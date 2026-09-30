# XChoice: Explainable Evaluation of AI–Human Alignment in LLM-based Constrained Choice Decision Making

<!-- TODO: confirm the official venue name and year, then replace the badge text below -->
**Paper:** *XChoice: Explainable Evaluation of AI–Human Alignment in LLM-based Constrained Choice Decision Making* (accepted at AACL)
**Authors:** Weihong Qi, Fan Huang, Rasika Muralidharan, Jisun An, Haewoon Kwak (Indiana University Bloomington)

> **Status:** This repository accompanies the camera-ready version of the paper. Code is being cleaned up and will be released here incrementally. See [Release plan](#release-plan). ATUS data are not redistributed here; see [Data](#data) for how to obtain them from IPUMS.

---

## Overview

Most evaluations of LLM–human alignment in decision tasks check whether a model's choices *match* human choices. XChoice asks a different question: **does the model make the same trade-offs, for the same reasons?**

XChoice treats each decision maker (a human population or an LLM) as solving a constrained optimization problem, recovers the latent trade-off parameters that best explain its observed decisions via inverse optimization, and compares those parameters across humans and models. Alignment is then assessed at the level of the decision mechanism rather than the outcome.

The framework has three steps:

1. **Model** the decision as constrained optimization, with interpretable parameters that weight decision-relevant attributes.
2. **Estimate** the parameters from human data and from LLM-generated decisions using the same procedure (non-linear least squares for continuous-share decisions; maximum likelihood for discrete choices).
3. **Compare** the recovered parameters with model-level metrics (cosine similarity, mean absolute deviation) and attribute-level metrics that localize which attributes drive misalignment, then use the diagnostics to guide and evaluate targeted interventions.

## Case study: daily time allocation

We instantiate XChoice on how Americans allocate the 1,440 minutes of a day across work, leisure, sleep and personal care, and other activities.

- **Human benchmark:** 4,307 respondents from the 2023 American Time Use Survey (ATUS), accessed through the IPUMS ATUS Data Extract Builder.
- **LLMs evaluated:** GPT-4o, Claude-3.7-Sonnet, DeepSeek-V3, Llama-3.3-70B, and Qwen-2.5-72B, each prompted with the demographic profile of every ATUS respondent.
- **Model:** a Cobb–Douglas time-budget utility with softmax time shares, estimated by non-linear least squares.

**Main findings**

- Alignment is heterogeneous across models and activities; no model is uniformly aligned.
- Misalignment concentrates in subgroups defined by race (Black respondents) and marital status (spouse present), where most LLMs attenuate or reverse the human trade-off weights.
- The recovered parameters are more stable under covariate shifts than reduced-form (OLS) baselines, and the diagnostic ranking is robust to a broad set of specification and identification checks.
- A proof-of-concept retrieval-augmented intervention shows that mechanism-level re-estimation can reveal both when an intervention helps and when it hurts.
- A discrete-choice instantiation on Anthropic HH-RLHF shows that the framework extends beyond continuous-share decisions.

## Why this matters

LLMs are increasingly deployed as decision-support tools and autonomous agents that allocate limited resources: time, budget, tool calls, or context. Outcome agreement can look acceptable while the underlying trade-offs differ in systematic, subgroup-specific ways. XChoice provides an auditable, interpretable way to surface those differences before they propagate into downstream decisions.

## Repository structure (planned)

```
xchoice/
├── data/            # place your own IPUMS ATUS extract here (not included)
├── prompts/         # prompt templates used to elicit LLM decisions
├── generation/      # scripts for querying LLMs
├── estimation/      # structural estimation (softmax NLLS) and inverse optimization
├── metrics/         # alignment metrics (CosSim, M_l, A_f) and statistical tests
├── robustness/      # invariance analysis and robustness checks
├── rag/             # retrieval-augmented intervention and ablations
├── hh_rlhf/         # discrete-choice instantiation on HH-RLHF
└── notebooks/       # figures and tables in the paper
```

## Data

**This repository does not include or redistribute any ATUS data.** ATUS microdata are publicly available, but they must be obtained through IPUMS ATUS and are subject to the [IPUMS terms of use](https://www.atusdata.org/atus-action/faq). To reproduce our analysis, please create your own extract as follows.

### 1. Create an IPUMS ATUS extract

1. Register for a free account at [atusdata.org](https://www.atusdata.org/).
2. Open the **Data Extract Builder** and select the **2023** ATUS sample.
3. Add the following variables (some identifiers are selected by default):

   | Group | Variables |
   |---|---|
   | Identifiers and weights | `YEAR`, `CASEID`, `SERIAL`, `PERNUM`, `LINENO`, `WT06` |
   | Diary day | `DAY` |
   | Respondent characteristics | `AGE`, `SEX`, `RACE`, `EDUC`, `EDUCYRS`, `EARNWEEK`, `SPOUSEPRES` |
   | Time use (minutes per day) | `ACT_PCARE`, `ACT_SOCIAL`, `ACT_SPORTS`, `ACT_WORK` |

4. Choose **CSV** as the data format, submit the extract, and download it once IPUMS notifies you that it is ready.
5. Place the downloaded file in `data/raw/`. The preprocessing script (to be released) reads it from there.

The resulting extract contains 8,548 respondents.

### 2. Sample restrictions and variable construction

Preprocessing reduces the extract to the 4,307-respondent analytic sample used in the paper (details in Appendix C):

- **Sample restrictions:** drop respondents coded NIU on `SEX` or `SPOUSEPRES`, drop missing weekly earnings (`EARNWEEK` = 99999.99), and keep single-race respondents (`RACE` < 200).
- **Covariates:** age and weekly earnings (standardized), a four-level education variable collapsed from `EDUCYRS` (also standardized), sex, spouse present and unmarried partner present (reference: neither), and four single-race indicators (reference: White only).
- **Activity categories:** Work = `ACT_WORK`; Leisure = `ACT_SOCIAL` + `ACT_SPORTS`; Sleep and personal care = `ACT_PCARE`; Other = 1,440 minutes minus the three categories above.

### 3. Cite the data

Please cite the IPUMS ATUS extract in any work that uses it:

> Sarah M. Flood, Liana C. Sayer, and Daniel Backman. American Time Use Survey Data Extract Builder: Version 3.3 [dataset]. College Park, MD: University of Maryland and Minneapolis, MN: IPUMS, 2025. https://doi.org/10.18128/D060.V3.3

## Release plan

- [x] README, paper link, and data-access instructions
- [ ] Prompt templates
- [ ] Data preprocessing code
- [ ] Estimation code and alignment metrics
- [ ] Robustness checks and RAG experiments
- [ ] HH-RLHF instantiation

## Citation

If you find this work useful, please cite:

<!-- TODO: replace with the official ACL Anthology BibTeX once the proceedings are published -->
```bibtex
@inproceedings{qi2026xchoice,
  title     = {{XChoice}: Explainable Evaluation of {AI}--Human Alignment in {LLM}-based Constrained Choice Decision Making},
  author    = {Qi, Weihong and Huang, Fan and Muralidharan, Rasika and An, Jisun and Kwak, Haewoon},
  booktitle = {Proceedings of AACL},
  year      = {2026}
}
```

## Contact

For questions, please open an issue or contact Weihong Qi (wq3@iu.edu) or Haewoon Kwak (hwkwak@iu.edu).

<!-- TODO: add a LICENSE file before releasing code -->
