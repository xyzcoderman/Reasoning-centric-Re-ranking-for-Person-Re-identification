# Reasoning-centric Re-ranking for Person Re-identification

We thank all reviewers for their insightful and constructive comments. We appreciate all reviewers finding our work **novel, interesting, and timely**. We hope this rebuttal allows the reviewer to raise the score.

## Reviewer vA2m

### Q1.1. Comparison with prior MLLM ReID works [3, 28].

**Response.** We are sorry for not clearly explaining the rationale behind our baseline selection and the specific gaps of [3] and [28]. We agree that both works are relevant to the motivation of our study. However, they are not direct quantitative baselines because they address different problem settings. Specifically, [3] focuses on MLLM-based ReID-style identity verification in controlled one-to-one or one-to-many settings, without proposing a standard retrieval-list re-ranking algorithm or reporting comparable re-ranking metrics. [28] targets scene-aware text-based person retrieval, where the query is textual and the gallery contains full-scene images, which differs from our image-based ReID re-ranking setting. Therefore, our quantitative comparisons focus on methods that directly refine an initial ReID ranking list under the same evaluation protocol. We will revise the paper to clarify this distinction and discuss [3] and [28] as motivation-relevant.

### Q1.2. Experimental methodology before results.

**Response.** We will carefully revise the results section and first explain the full evaluation protocol.

### Q1.3. SCG-CoT is unclear and notation is convoluted.

**Response.** SCG-CoT contributes a reliability-aware reasoning mechanism for MLLM-based ReID re-ranking. Since ReID is sensitive to camera, viewpoint, pose, and illumination changes, stable identity cues are more reliable than superficial appearance cues. SCG-CoT introduces a stable-cue gate that prioritizes cues such as carrying state, footwear type, and coarse garment category. If a candidate contradicts these stable cues, its score is suppressed even if it looks globally similar. This makes the reasoning more constraint-guided, interpretable, and robust to near-duplicate distractors. We will simplify all notations and remove redundant equations.

### Q1.4. Justification for Qwen3-VL and CUHK03.

**Response.** We selected Qwen3-VL-8B-Instruct because it offers a practical balance between strong multimodal reasoning and feasible local inference. This is suitable for evidence-based person comparison, and our model-size ablation shows a clear trend: larger Qwen3-VL variants consistently improve CFM, with the 8B model performing best. This supports using Qwen3-VL-8B as the strongest deployable variant in our setting. We selected CUHK03 for ablation because it is a standard ReID benchmark with manually labeled and detector-generated boxes, making it suitable for analyzing detection noise, cross-camera variation, and re-ranking errors. We will revise the paper to state this rationale clearly, refer to the model-size ablation, and list additional MLLM backbones as future work.

### Q1.5. Positioning with respect to SOTA should be better positioned, and the gap analysis is insufficient.

**Response.** Conventional distance-and graph-based re-ranking methods mainly exploit embedding-space neighborhood structure, which is effective for global ranking refinement but can be brittle when the top candidates contain near-duplicate distractors or severe intra-class variation. Moreover, these strategies provide little interpretability: they do not explain why one candidate should rank above another. Recent MLLM-based ReID studies show the potential of multimodal reasoning, but mainly focus on identity verification or text-based retrieval rather than standard image-based ReID re-ranking. Our work addresses this missing setting by formulating ReID re-ranking as evidence-constrained visual verification over an initial top-k retrieval list while producing evidence-grounded explanations for ranking decisions.

## Reviewer KCRt

### Q2.1. Technical contribution seems incremental / inference-time heuristic.

**Response.** We clarify that CFM is not merely prompt/rule engineering on top of an off-the-shelf MLLM. It defines a structured evidence-verification procedure for ReID re-ranking: the model is constrained to use only visible identity cues, suppress uncertain or occluded evidence, accumulate cue-level contradictions through hierarchical penalty stacking, and apply deterministic tie-breaking for stable ranking. Thus, the MLLM acts as a constrained visual verifier rather than a free-form ranking generator. We agree that the current framework does not include task-specific training or learned adaptation. This is an intentional inference-time design choice to examine whether structured MLLM reasoning can improve ReID re-ranking without modifying the backbone or requiring additional annotated training data. Full fine-tuning or parameter-efficient adaptation may further improve task-specific performance but would introduce additional training costs and possible overfitting. We will clarify this trade-off and position CFM as an inference-time, verification-based re-ranking direction, with task-adaptive MLLM tuning left as future work.

### Q2.2. Mixed metrics: Rank-1 improves but mAP/mINP can be lower.

**Response.** We agree that CFM is not uniformly superior across all metrics. To address this, we will update the SOTA comparison with both CFM top-k=5 and CFM top-k=10 as shown in Table 2. Under the accuracy-oriented top-k=10 setting, CFM achieves the best Rank-1 on all three datasets and becomes more competitive in mAP, achieving the best mAP on Market-1501 and CUHK03 while remaining close on DukeMTMC. However, we acknowledge that CFM still lags behind graph-based methods on mINP.

### Q2.3. Single MLLM backbone.

**Response.** We agree that evaluating only one MLLM family limits evidence of robustness across different backbones.

**Table 1: Comparison of different MLLM variants on CUHK03. The suffix “B” denotes model size in billions of parameters. Performance is evaluated at top-k = 5.**

| Model | mAP | Rank-1 | mINP |
|---|---:|---:|---:|
| Qwen3-VL-2B-Instruct | 69.6 | 61.1 | 60.0 |
| Qwen3-VL-4B-Instruct | 74.6 | 69.8 | 70.1 |
| Qwen3-VL-8B-Instruct | **83.4** | **85.5** | **76.9** |

To assess sensitivity to model capacity, we provide a scale ablation in the supplementary material using Qwen3-VL-2B, 4B, and 8B under the same CFM process. As shown Table 1, performance improves consistently with model scale, and Qwen3-VL-8B achieves the best results. This suggests that CFM benefits from stronger MLLM capacity within the evaluated model family. We will revise the manuscript to clarify this scope and identify cross-backbone validation with additional MLLMs as future work.

### Q2.4. Interpretability and evidence-grounded reasoning.

**Response.** We include a generated explanation reliability analysis in the supplementary material, where 6,923 extracted attribute claims from 1,400 CUHK03 queries are evaluated using CLIP ViT-B/16 as a zero-shot prompt-ranking verifier. This analysis shows that the explanations are more reliable for coarse visual cues, especially color attributes. We will revise the manuscript to make this evidence more visible.

## Reviewer oPTM

### Q3.1. Limited overall ranking quality.

**Response.** We agree that the current discussion overemphasizes Rank-1 and does not sufficiently address the mAP/mINP trade-off. To address this concretely, we will update Table 2 by reporting both the default cost-efficient setting, CFM top-k=5, and an accuracy-oriented setting, CFM top-k=10. As shown in the Table 2, CFM top-k=10 achieves the best Rank-1 on all three datasets and becomes more competitive in mAP, achieving the best mAP on Market-1501 and CUHK03 while remaining close on DukeMTMC.

**Table 2: Comparison of our reasoning-centric re-ranking method with existing re-ranking approaches on three benchmarks. All existing baselines are evaluated under top-k=10. Red text indicates improvements over the baseline.**

| Base | Method | Venue | Market-1501 mAP | Market-1501 R1 | Market-1501 mINP | DukeMTMC mAP | DukeMTMC R1 | DukeMTMC mINP | CUHK03 mAP | CUHK03 R1 | CUHK03 mINP |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| CLIP-ReID [6] | - | AAAI23 | 94.8 | 95.4 | 92.0 | 91.1 | 90.7 | 87.5 | 80.0 | 79.5 | 74.3 |
| CLIP-ReID [6] | QE [1] | ICCV07 | 95.5 | 95.0 | 94.3 | 92.5 | 91.6 | 91.2 | 83.0 | 79.2 | 81.8 |
| CLIP-ReID [6] | LBR [10] | ICCV19 | 95.0 | 94.6 | 93.8 | 92.5 | 91.9 | 91.3 | 82.0 | 79.5 | 80.0 |
| CLIP-ReID [6] | KR [36] | CVPR17 | 96.1 | 95.1 | **95.8** | 93.1 | 91.7 | 92.7 | 86.0 | 82.7 | 86.0 |
| CLIP-ReID [6] | ECN [17] | CVPR18 | 96.2 | 95.6 | 95.7 | **93.3** | 91.9 | **93.0** | 86.1 | 83.2 | 86.1 |
| CLIP-ReID [6] | Cheb-GR [29] | CVPR25 | 95.6 | 95.2 | 94.4 | 92.8 | 91.1 | 92.5 | 86.3 | 83.5 | **86.4** |
| CLIP-ReID [6] | CFM (top-k=5) | N/A | 95.4 | 96.4 | 92.4 | 91.3 | 91.9 | 87.5 | 83.4 | 85.5 | 76.9 |
| CLIP-ReID [6] | **CFM (top-k=10)** | N/A | **96.3** | **96.8** | 93.8 | 92.6 | **92.0** | 90.4 | **87.8** | **86.9** | 84.6 |
| CLIP-ReID [6] | **CFM (top-k=10)** | N/A | **+1.5** | **+1.4** | **+1.8** | **+1.5** | **+1.3** | **+2.9** | **+7.8** | **+7.4** | **+10.3** |

We will also move the shortlist-size sensitivity analysis into the main paper. As shown in Figure 1, increasing k from 1 to 10 steadily improves Rank-1 accuracy on CUHK03 from 79.5% to 86.9%. This confirms that a relatively larger shortlist gives CFM more opportunities to recover correct identities missed by the initial retrieval. In the revision, we will explicitly discuss this cost-accuracy trade-off: k=5 provides an efficient balance, while k=10 gives stronger accuracy-oriented performance. Overall, we will revise the results discussion to clearly state that CFM improves top-rank identity selection.


<img width="757" height="502" alt="one" src="https://github.com/user-attachments/assets/e2c63241-ede6-4bbe-8e80-39bad8ef8820" />
**Figure 1: Effect of shortlist size k on Rank-1 accuracy.**

### Q3.2. Contribution is mostly prompt engineering.

**Response.** The current framework does not include task-specific training or learned adaptation. This is an intentional inference-time design choice to examine whether structured MLLM reasoning can improve ReID re-ranking without modifying the backbone or requiring additional annotated training data. Empirically, CFM top-k=10 achieves the best Rank-1 on all three datasets and improves mAP competitiveness, achieving the best mAP on Market-1501 and CUHK03 while remaining close on DukeMTMC.

### Q3.3. Unrealistically small shortlist.

**Response.** The small shortlist is an intentional design choice because our method is a second-stage verifier after fast embedding-based retrieval, while full-gallery MLLM inference is computationally impractical. However, we agree that this operating point should be better justified. In the revision, we will move the top-k sensitivity analysis into the main paper and report both accuracy and efficiency across different shortlist sizes.

### Q3.4. Formalism is heavier than the method warrants.

**Response.** We will simplify the formalism. This should improve readability and reduce the impression that equations are used to inflate the method.
