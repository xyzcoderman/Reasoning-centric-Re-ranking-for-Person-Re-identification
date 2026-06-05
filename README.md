# Reasoning-centric Re-ranking for Person Re-identification

> **Anonymous rebuttal supplement.**  
> This repository provides additional clarification, tables, and analysis for the anonymous submission **“Reasoning-centric Re-ranking for Person Re-identification.”**  
> The repository is intended only to support the rebuttal and to provide concise supplementary evidence under the double-blind review policy.

---

## 1. Purpose of This Repository

Due to rebuttal character limitations, this repository summarizes the main clarifications requested by the reviewers:

1. Why the proposed method is not only prompt engineering.
2. What **Constrained Feature Matching (CFM)** contributes as an inference-time ReID re-ranking framework.
3. What **Stable-Cue-Gated Chain-of-Thought Reasoning (SCG-CoT)** contributes and how it should be interpreted.
4. Why **Qwen3-VL-8B-Instruct** and **CUHK03** are used in the main ablation.
5. How the method compares with conventional re-ranking methods under both cost-efficient and accuracy-oriented shortlist sizes.
6. What limitations remain, especially for mAP/mINP and cross-backbone validation.

---

## 2. Paper Overview

Person re-identification (ReID) aims to retrieve images of the same individual across non-overlapping camera views. In practical scenarios, initial retrieval lists often contain visually similar distractors caused by viewpoint changes, pose variation, illumination shifts, occlusion, and near-duplicate appearances.

This work formulates ReID re-ranking as **evidence-constrained visual verification** over an initial top-`k` retrieval list. Instead of directly asking a multimodal large language model (MLLM) to produce a free-form ranking, the proposed framework constrains the model to reason over visible identity evidence and then re-ranks candidates using structured verification signals.

The method is designed as an **inference-time second-stage verifier** after fast embedding-based retrieval. It does not require task-specific MLLM fine-tuning, additional annotated training data, or modification of the visual backbone.

---

## 3. Main Contributions Clarified

### 3.1 Evidence-constrained ReID re-ranking

The proposed method introduces a verification-oriented re-ranking paradigm for image-based ReID. It asks the MLLM to compare a query image and top-`k` candidates using only visible identity cues. This differs from free-form MLLM ranking because the model is not allowed to invent unsupported evidence.

### 3.2 Constrained Feature Matching (CFM)

**CFM** is the core re-ranking procedure. It constrains the MLLM to:

- use only visible identity cues;
- suppress uncertain, occluded, or unverifiable evidence;
- accumulate cue-level contradictions through hierarchical penalty stacking;
- apply deterministic tie-breaking to maintain stable ranking behavior;
- produce evidence-grounded explanations for ranking decisions.

Therefore, the MLLM acts as a **constrained visual verifier**, not as an unconstrained ranking generator.

### 3.3 Stable-Cue-Gated Chain-of-Thought Reasoning (SCG-CoT)

**SCG-CoT** introduces a reliability-aware reasoning mechanism. ReID is sensitive to camera, viewpoint, pose, and illumination changes, so not all visual cues are equally reliable. SCG-CoT prioritizes more stable identity cues before considering less reliable appearance details.

Stable cues include:

- carrying state;
- footwear type;
- coarse garment category;
- other identity-related cues that are less sensitive to lighting and viewpoint changes.

If a candidate contradicts stable identity cues, its score is suppressed even when the candidate looks globally similar. This makes the reasoning more constraint-guided, interpretable, and robust to near-duplicate distractors.

---

## 4. Reviewer Concern Mapping

| Reviewer Concern | Clarification Provided |
|---|---|
| Comparison with prior MLLM ReID works | Prior works are motivation-relevant but not direct quantitative baselines because they focus on different settings such as identity verification or scene-aware text-based retrieval. |
| Experimental methodology | The revised paper will explain the evaluation protocol before presenting results. |
| SCG-CoT is unclear | SCG-CoT will be rewritten as a simple stable-cue gating mechanism with simplified notation. |
| Qwen3-VL and CUHK03 justification | Qwen3-VL-8B provides the strongest deployable performance among tested Qwen3-VL variants; CUHK03 is suitable for analyzing detection noise, cross-camera variation, and re-ranking errors. |
| Contribution seems incremental | CFM is positioned as an inference-time structured verification framework, not merely a prompt heuristic. |
| Mixed metrics | CFM improves top-rank identity selection, especially Rank-1, but does not uniformly dominate all metrics. The mINP limitation is explicitly acknowledged. |
| Single MLLM backbone | A model-size ablation is provided within the Qwen3-VL family; cross-backbone validation is left for future work. |
| Small shortlist | The method is a second-stage verifier after fast retrieval. Full-gallery MLLM inference is computationally impractical. Top-`k` sensitivity analysis is provided. |
| Heavy formalism | The revised manuscript will simplify notation and remove redundant equations. |

---

## 5. Experimental Setting

The evaluation follows the standard ReID re-ranking setting:

- **Initial retriever:** CLIP-ReID
- **Re-ranking stage:** CFM with MLLM-based visual verification
- **Datasets:** Market-1501, DukeMTMC, CUHK03
- **Metrics:** mAP, Rank-1, mINP
- **Shortlist settings:** top-`k=5` and top-`k=10`

The method is evaluated as a second-stage re-ranker. A fast embedding-based model first retrieves the top candidates, and the proposed MLLM verifier only operates on the shortlist.

---

## 6. SOTA Comparison

The following table summarizes the updated comparison discussed in the rebuttal. Existing baselines are evaluated under top-`k=10`. CFM is reported under both top-`k=5` and top-`k=10`.

| Base | Method | Venue | Market mAP | Market R1 | Market mINP | Duke mAP | Duke R1 | Duke mINP | CUHK03 mAP | CUHK03 R1 | CUHK03 mINP |
|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| CLIP-ReID | Baseline | AAAI23 | 94.8 | 95.4 | 92.0 | 91.1 | 90.7 | 87.5 | 80.0 | 79.5 | 74.3 |
| CLIP-ReID | QE | ICCV07 | 95.5 | 95.0 | 94.3 | 92.5 | 91.6 | 91.2 | 83.0 | 79.2 | 81.8 |
| CLIP-ReID | LBR | ICCV19 | 95.0 | 94.6 | 93.8 | 92.5 | 91.9 | 91.3 | 82.0 | 79.5 | 80.0 |
| CLIP-ReID | KR | CVPR17 | 96.1 | 95.1 | **95.8** | 93.1 | 91.7 | 92.7 | 86.0 | 82.7 | 86.0 |
| CLIP-ReID | ECN | CVPR18 | 96.2 | 95.6 | 95.7 | **93.3** | 91.9 | **93.0** | 86.1 | 83.2 | 86.1 |
| CLIP-ReID | Cheb-GR | CVPR25 | 95.6 | 95.2 | 94.4 | 92.8 | 91.1 | 92.5 | 86.3 | 83.5 | **86.4** |
| CLIP-ReID | CFM top-`k=5` | N/A | 95.4 | 96.4 | 92.4 | 91.3 | 91.9 | 87.5 | 83.4 | 85.5 | 76.9 |
| CLIP-ReID | **CFM top-`k=10`** | N/A | **96.3** | **96.8** | 93.8 | 92.6 | **92.0** | 90.4 | **87.8** | **86.9** | 84.6 |
| CLIP-ReID | CFM top-`k=10` improvement over baseline | N/A | **+1.5** | **+1.4** | **+1.8** | **+1.5** | **+1.3** | **+2.9** | **+7.8** | **+7.4** | **+10.3** |

### Key observation

CFM top-`k=10` achieves the best Rank-1 on all three datasets. It also achieves the best mAP on Market-1501 and CUHK03 while remaining close on DukeMTMC. However, graph-based methods still remain stronger on some mINP results, which we explicitly acknowledge as a limitation.

---

## 7. MLLM Scale Ablation

The following ablation evaluates different Qwen3-VL model sizes under the same CFM process on CUHK03 with top-`k=5`.

| Model | mAP | Rank-1 | mINP |
|---|---:|---:|---:|
| Qwen3-VL-2B-Instruct | 69.6 | 61.1 | 60.0 |
| Qwen3-VL-4B-Instruct | 74.6 | 69.8 | 70.1 |
| **Qwen3-VL-8B-Instruct** | **83.4** | **85.5** | **76.9** |

### Key observation

Performance improves consistently with model scale. This supports the selection of Qwen3-VL-8B-Instruct as the strongest deployable variant within the evaluated model family.

---

## 8. Top-`k` Sensitivity

The rebuttal clarifies that the shortlist size controls a cost-accuracy trade-off.

- A smaller shortlist such as top-`k=5` is more efficient.
- A larger shortlist such as top-`k=10` gives the MLLM verifier more opportunities to recover correct identities missed by the initial retriever.
- On CUHK03, increasing `k` from 1 to 10 improves Rank-1 from 79.5% to 86.9%.

The revised paper will move the top-`k` sensitivity analysis into the main paper and discuss both efficiency and accuracy.

---

## 9. Explanation Reliability Analysis

To further support interpretability, the supplementary analysis evaluates generated explanation reliability.

- **Dataset:** CUHK03
- **Number of queries:** 1,400
- **Extracted attribute claims:** 6,923
- **Verifier:** CLIP ViT-B/16 zero-shot prompt-ranking verifier

The analysis shows that explanations are more reliable for coarse visual cues, especially color attributes. The revised manuscript will make this evidence more visible.

---

## 10. Why Prior MLLM ReID Works Are Not Direct Quantitative Baselines

The rebuttal clarifies the relationship with prior MLLM-based ReID works.

- Prior MLLM ReID-style identity verification works mainly focus on controlled one-to-one or one-to-many identity verification.
- They do not propose a standard retrieval-list re-ranking algorithm.
- They do not report directly comparable re-ranking metrics under the same protocol.
- Scene-aware text-based person retrieval uses textual queries and full-scene gallery images, which differs from image-based ReID re-ranking.

Therefore, these works are motivation-relevant but not direct quantitative baselines. Quantitative comparison focuses on methods that directly refine an initial ReID ranking list under the same evaluation protocol.

---

## 11. Method Scope and Limitations

This repository also clarifies the current scope of the method.

### Current scope

- Inference-time re-ranking.
- No task-specific MLLM fine-tuning.
- No additional annotated training data.
- Second-stage verification over an initial top-`k` shortlist.
- Evidence-grounded ranking decisions.

### Current limitations

- The method does not uniformly dominate all metrics.
- mINP can still lag behind strong graph-based re-ranking methods.
- Only Qwen3-VL family variants are evaluated in the current ablation.
- Full-gallery MLLM inference is computationally impractical.
- Cross-backbone validation with additional MLLMs remains future work.
- Parameter-efficient adaptation or fine-tuning may further improve performance but introduces extra training cost and possible overfitting.

---

## 12. Suggested Repository Structure

This anonymous repository can be organized as follows:

```text
.
├── README.md
├── tables/
│   ├── sota_comparison.md
│   └── mllm_scale_ablation.md
├── figures/
│   └── topk_effect.pdf
├── supplementary/
│   ├── explanation_reliability.md
│   └── reviewer_response_summary.md
└── assets/
    └── anonymous_notice.md
```

For a lightweight rebuttal-only repository, the most important files are:

- `README.md`: this file;
- `tables/sota_comparison.md`: updated SOTA comparison;
- `tables/mllm_scale_ablation.md`: Qwen3-VL scale ablation;
- `figures/topk_effect.pdf`: top-`k` sensitivity figure;
- `supplementary/explanation_reliability.md`: explanation reliability analysis.

---

## 13. Anonymous Review Policy

This repository is prepared for anonymous review. It should not contain:

- author names;
- affiliations;
- personal emails;
- acknowledgements;
- institutional paths;
- personal account names;
- non-anonymous commit metadata;
- dataset paths that reveal identity or institution.

Suggested anonymous repository name:

```text
anonymous-reid-reranking-supplement
```

Suggested repository description:

```text
Anonymous rebuttal supplement for reasoning-centric person ReID re-ranking.
```

All communication should remain through the official review system.

---

## 14. Summary

This repository supports the rebuttal by clarifying that the proposed framework is an inference-time, evidence-constrained MLLM verification approach for image-based ReID re-ranking. CFM provides structured visual verification, while SCG-CoT adds reliability-aware stable-cue gating. The updated results show that CFM top-`k=10` improves Rank-1 across Market-1501, DukeMTMC, and CUHK03, while the remaining limitations on mINP and cross-backbone validation are explicitly acknowledged.
