# ADR-002: Scoring thresholds
## Context

Certifiable's exams require a consistent set of grading criteria to determine whether an exam question is deemed sufficiently correct. The introduction of AI-assisted grading, including the potential for some questions to be scored by AI while others are graded by humans, requires a consistent rubric for both AI and human-graded questions in order to ensure fairness.

Please also see the [ADR on Confidence-based scoring](/adr/004-confidence-based-grading-process.md) for details on the confidence assessment that is applied along with this scoring.

## Decision

The grading system assigns two key scores to each answer:

- **Correctness Score (1-10)**: Measures how well an answer aligns with reference answers.
- **Confidence Score (1-10)**: Reflects the AI’s certainty in its correctness assessment.

Scoring that is deemed to have sufficient confidence will proceed with the following sets of configurable thresholds:

| Correctness Score | Decision              |
|-------------------|-----------------------|
| 8-10              | Automatic Pass        |
| 6-7               | Requires Human Review |
| 1-5               | Automatic Fail        |

This approach ensures AI handles straightforward cases while human graders review ambiguous or complex answers.

The correctness score will be calculated differently for each test.

### Test 1
Since Certifiable is using a relational database with caching to store curated correct and incorrect answers, correctness levels can be tied to **semantic similarity to known high-quality responses**.

AI compares answers to pre-graded reference answers from the relational database. If an answer closely matches the semantic meaning of a previously accepted response, the AI assigns a high correctness score. If an answer deviates significantly, correctness is lowered.

#### Correctness Scaling Based on Answer Similarity

Correctness will be graded by the LLM on a scale from 1-10, with 1 being least similar to the supplied correct answers, and 10 being the most similar.

**TODO** Example prompt that would produce a bracketed correctness score for a new answer given a selection of known good answers to the same question

### Test 2 
Since Certifiable is using a multi-perspective grading approach ([see the ADR here](/adr/005-test-2-scoring.md)), the correctness score will be determined based on the **aggregate of all scores across the different agents**. This means that a single low correctness score might be offset by high correctness scores from other perspectives.

In addition, the use of weighted agent scoring (with the possible future inclusion of red-line minimum scoring for each agent to ensure that answers don't pass overall with an extremely low score from one agent) can be used to further accentuate certain points of view for individual questions. For example, a security-focussed agent could be weighted higher than a scalability/ops-focussed agent for a low-volume, critical service.

As an example, for a given answer, a supplied answer might receive:
* Security Agent: 9 (x 2, weighted agent)
* Infrastructure/Ops Agent: 7
* Disaster Recovery Agent: 5

Providing a total score of (9 x 2) + 7 + 5 = 30 out of a total possible score of 40.

Automatic pass / human review / automatic fail thresholds should be maintained on a per-question basis for exam 2 responses, given the likelihood of high variability/bespoke nature of the evaluation.

Completing our example, if this exam question's correctness thresholds were set with the following values:

| Correctness Score | Decision              |
|-------------------|-----------------------|
| 32+               | Automatic Pass        |
| 24-31             | Requires Human Review |
| 4-23              | Automatic Fail        |

Our answer would be routed for human review, as it falls in the middle bracket.

**TODO** Have we sufficiently explained the 1-10 grading for the agentic scoring here or elsewhere? Is it based on known good answers, or the stated criteria for each answer?

## Options Considered

1. **Fully Automated AI Grading**
   - AI assigns scores to all answers without human review.
   - Relies entirely on AI decision-making, without providing human oversight for borderline answers.
   
2. **Hybrid Model with Correctness-Based Thresholds (Selected Approach)**
   - AI assigns scores but defers uncertain cases to human graders.
   - Balances efficiency with quality control.
   
3. **Human-Only Grading**
   - AI is not used, all grading is manual.
   - Ensures full human oversight but is resource-intensive.
   - Automatically discarded because it's unsustainable as a business model. 

## Pros

- **Efficiency Gains**: Reduces human workload by filtering out clear pass/fail cases.
- **Scalability**: Supports increasing exam volumes without overwhelming human graders.
- **Cost Reduction**: Reduces grading expenses by automating a portion of the process.

## Cons

- **Potential AI Errors**: Some borderline cases may still be misclassified.
- **Admin overhead**: Requires setting of pass/fail thresholds for each question in exam 2
- **Ongoing Monitoring**: Requires regular tuning of correctness thresholds to maintain grading accuracy.
- **Infrastructure Costs**: AI processing costs, while lower than full human grading, still add to the operational cost.

## Conclusion

Confidence-based grading ensures a balance between automation and human oversight, maintaining grading quality while achieving substantial cost savings. By filtering out high-confidence pass and fail cases from the human grading queue, this approach reduces human grading workload while ensuring that nuanced or uncertain answers receive appropriate human review. This method supports Certifiable’s scalability while preserving certification credibility.
