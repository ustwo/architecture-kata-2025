# ADR-006: Confidence-Based Grading Process
## Context

Certifiable is incorporating AI to assist in grading while maintaining accuracy and fairness. A key challenge is determining when AI can autonomously grade an exam response versus when human intervention is required. This ADR defines how confidence levels guide the grading process and provides a cost analysis of AI-assisted grading.

## Decision

The grading system assigns two key scores to each answer:

- **Correctness Score (1-10)**: Measures how well an answer aligns with reference answers.
- **Confidence Score (1-10)**: Reflects the AI’s certainty in its correctness assessment.

Thresholds are used to determine when AI grading is sufficient:

| Correctness Score | Confidence Score | Decision          |
|-------------------|-----------------|-------------------|
| 8-10             | ≥ 7              | Automatic Pass   |
| 6-7              | ≤ 6              | Requires Human Review |
| 1-5              | Any              | Automatic Fail   |

This approach ensures AI handles straightforward cases while human graders review ambiguous or complex answers.

The confidence score will be calculated differently for each test.

### Test 1
Since Certifiable is using a relational database with caching to store curated correct and incorrect answers, AI confidence levels can be tied to similarity to known high-quality responses.

AI compares answers to pre-graded reference answers from the relational database. If an answer closely matches a previously accepted response, the AI assigns a high confidence score. If an answer deviates significantly, confidence is lowered.

#### Confidence Scaling Based on Answer Similarity

| Similarity to Reference Answers | Confidence Score |
|---------------------------------|-----------------|
| 90%+ (Exact Match)              | **10**          |
| 75-89% (High Similarity)        | **8-9**         |
| 50-74% (Moderate Similarity)    | **6-7**         |
| < 50% (Low Similarity)          | **1-5**         |

### Test 2 
Since Certifiable is using a multi-perspective grading approach, the confidence score will be based on agreement between the different perspectives. Confidence is increased if multiple viewpoints converge around the same score. Confidence is lowered if different viewpoints give heterogeneous scores. 

#### Confidence Adjustment Based on Multi-Perspective Agreement

| Agreement Among AI Perspectives | Confidence Score |
|---------------------------------|-----------------|
| 5/5 perspectives agree          | **10**          |
| 4/5 perspectives agree          | **8-9**         |
| 3/5 perspectives agree          | **6-7**         |
| 2/5 perspectives agree          | **4-5**         |
| 1/5 perspectives agree          | **1-3**         |

## Options Considered

1. **Fully Automated AI Grading**
   - AI assigns scores to all answers without human review.
   - Relies entirely on confidence-based decision-making.
   
2. **Hybrid Model with Confidence-Based Thresholds (Selected Approach)**
   - AI assigns scores but defers uncertain cases to human graders.
   - Balances efficiency with quality control.
   
3. **Human-Only Grading**
   - AI is not used, all grading is manual.
   - Ensures full human oversight but is resource-intensive.
   - Automatically discarded because it's unsustainable as a business model. 

## Pros

- **Efficiency Gains**: Reduces human workload by filtering out clear pass/fail cases.
- **Quality Control**: Confidence thresholds prevent unreliable AI decisions.
- **Scalability**: Supports increasing exam volumes without overwhelming human graders.
- **Cost Reduction**: 
  - Reduces grading expenses by automating a portion of the process.
  **TODO**: include cost calculations here 

## Cons

- **Potential AI Errors**: Some borderline cases may still be misclassified.
- **Ongoing Monitoring**: Requires regular tuning of confidence thresholds to maintain grading accuracy.
- **Infrastructure Costs**: AI processing costs, while lower than full human grading, still add a to the operational cost.

## Conclusion

Confidence-based grading ensures a balance between automation and human oversight, maintaining grading quality while achieving substantial cost savings. By filtering out high-confidence pass and fail cases, this approach reduces human grading workload while ensuring that nuanced or uncertain answers receive appropriate human review. This method supports Certifiable’s scalability while preserving certification credibility.
