# Test 2 scoring approach
Date: 2025-03-10

## Context
Architecture submissions are graded by Certifiable Inc, different approaches can be taken to use AI to grade submissions.  Care should be taken to avoid bias.

## Decision
The option to grade from different viewpoints has been chosen.

## Options Considered

All options are taken to include:
Recording the reasoning behind attributing scores
Being primed with statements of industry best practices
A random sample of gradings being reviewed by experts 

1. Similarity to ‘known good’ submissions

Submissions are compared to a body of expert selected high scoring previous submissions for this case study and scored for similarity.

### Indicative prompt

```
Grade the following architecture for a <use case summary> for similarity to these architectures. ...
```

### Pros

Transparent grading process.

### Cons

Risk of bias toward irrelevant factors.  For example - if most architecture diagrams are created using draw.io, the scoring could prejudice diagrams created using a different diagramming tool.


2. Grading from different viewpoints

Models grade submissions from a series of points of view, which are then combined to form an overall score.

### Indicative prompt

```
Grade the following architecture from the point of view of <role>: ...
```

`<Role>` being an iteration of
A security expert
A software engineer tasked with implementing the system
Privacy compliance
Etc.

### Pros

Bias is reduced
Gradings are easier to explain

### Cons

## Consequences

The grading from multiple points of view makes the grading process easier to review and is less likely to produce a monoculture.

### Risks

The approach is still not foolproof against marking-down innovative solutions, grading context should include clauses to identify controversial choices that are well explained and flag these for human review.
