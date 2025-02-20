# usfive - O'Reilly Architectural Katas: Spring 2025

## Team Members 
[Nick Hegarty](mailto:nickhegarty@ustwo.com)  
[Belén Molina](mailto:belenmolina@ustwo.com)  
[Joe McGuinness](mailto:joe.mcguinness@ustwo.com)  
[Graham Richards](mailto:grahamrichards@ustwo.com)  
[Vinicios Neves](mailto:vinicios.neves@ustwo.com)  

## Framing the problem

Certifiable operates the grading and awarding of standard certifications via a mixture of automated and human-involved processes. With their certification numbers increasing, they need to keep up with demand, while considering the following real factors affecting the exam grading:

* **Grading quality needs to be high**. Their current approach relies upon human grading for all non-multiple choice questions. While this introduces a layer of subjectivity, architecture as a discipline is not a set of binary choices, and this subjectivity could provide room for nuanced, unusual solutions that, on balance, are appropriate for the task at hand. With that in mind, any change that reduces the quality of the grading process is likely to erode confidence in the certification and, by extension, erode confidence in Certifiable's offering.
* **The exam grading process needs to remain profitable**. The financial considerations are:  
  * Exam takers pay a fee of $800  
  * Assuming the exam taker passes the first round (and therefore both rounds are graded), it takes a grader 11 hours on average to complete grading  
  * Exam graders are paid $50/hr  
  * This means the cost for grading is $550, leaving a margin of $250 for each exam (some of which will be consumed by hosting/compute/storage costs)  
* And the **time for an exam to be graded should remain within the guaranteed window** of 1 week for each exam round (1 and 2\)  
* The **anticipated scaling of exam submissions is 5-10x**, based on additional overseas applicants, in addition to the projected growth of 21% over the next 4 years

As such, when contemplating the deployment of AI to assist in Certifiable’s exam grading, *we should be focussing on areas that can materially decrease the human time necessary to perform the grading, while keeping a close eye on any impacts to quality*.

## Optimising Test 1

Test 1 contains both a multiple choice component, which is automatically graded today, and a short answer component that is hand-graded by reviewers, taking approximately 3 hrs/test.

The fundamental question to be answered in grading each of the individual short-answer questions is: *is the answer correct*? And, breaking it down further, *are all identifiable hallmarks of a correct answer present*?

And as a secondary layer that is specific to the introduction of AI: *how confident am I of my assessment of correctness*? For example, a correct answer to the question “where is the nearest gas station?” might be “go down this road and take a left”, but a confidently correct/incorrect answer would be “go 1 mile down this road, then take your first left onto Main Street and you will see it in 200 yards on your right.” The latter would be an answer that could be assessed with more confidence than the former.

### In action

* Each answer is submitted to the **RAG Question Evaluator** with a prompt requesting a correctness score out of 10 to be explicitly assigned, as well as a confidence score out of 10  
* Thresholds for human intervention must then be set to flag answers that require further consideration. For example, you might say:  
  * Any scores with a confidence level of \< 7 require human intervention  
  * Otherwise, scores 8-10 pass, 6-7 require human intervention, and 1-5 fail  
* Of course these thresholds will need adjustment, but can be used as starting points  
* “Automatic pass” must be an explicit status value for each answer, with “Reviewer pass” as an alternative. Similar gradients of status for failing grades should also be used.

![Test 1 Optimisation](/images/test1.png)

### Returns
Conservatively, we could estimate that 50% of answers can be automatically passed/failed using this approach. Depending on the nature \+ specificity of the short-answer questions, this could well be a higher percentage.

Doing the maths on potential gains from optimising test 1, then:

* In today’s terms, at 200 exams per week, assuming 100% submit test 1, roughly 600 hrs/week (200 exams \* 3 hrs each to assess) is spent on assessing the short-answer portion of this test. That equates to roughly 600 hours @ $50/hr \= $30,000/week spent. Assuming 50% of this in savings puts us at **$15,000/week**  
* Scales at 5-10x, this ranges from 3000-6000 hrs \= $150k-300k/week. Assuming 50% of this in savings puts us at **$75k-150k/week**.

## Optimising Test 2

Grading of Test 2 is similar in process to Test 1, but has at least 3 complicating factors:

1. The answers are longer and more nuanced. There are potentially more disparate acceptable answers.  
2. And there is presumably an expectation of diagrams/charts to represent architectural thinking, which are non-trivial to interpret via AI.  
3. And there are multiple scenarios in play.

These distinctions can be addressed via the following mitigating techniques:

1. Additional care must be taken to upload multiple different acceptable answers for each scenario  
2. Ensure all diagram submissions are in the form of text-based models, such as PlantUML. This will allow for the simplest ingestion of diagrams in a form that is simple for our RAG system to reason about  
3. Similar to the grouping of related documentation for each question in Test 1, information can be grouped by scenario, with correct answers \+ scoring for each. As such, each submission can be viewed as grading the answer to 1/x questions, where x is the total number of scenarios.

And finally, the RAG query’s output should be modified to not only score as in Test 1, but also to provide a nuanced critique, highlighting similarities and differences against known good answers.

Scoring should again be on a 1-10 scale, with high scores automatically passed, low scores automatically failed, and middle scores flagged for human grading.

![Test 2 Optimisation](/images/test2.png)

### Returns 
Conservatively, we could estimate that 30% of answers can be automatically passed/failed using this approach (a smaller percentage than Test 1, due to the more complex nature of the task).

Doing the maths on potential gains from optimising test 2, then:

* In today’s terms, at 200 exams per week, assuming 80% of users submit test 2 (based on potential attrition and failures on the first exam), roughly 600 hrs/week (160 exams \* 8 hrs each to assess) is spent on assessing the test. That equates to roughly 1280 hours @ $50/hr \= $64,000/week spent. Assuming 30% of this in savings puts us at **$19,200/week saved**  
* Scales at 5-10x, this ranges from 6400-12800 hrs \= $320k-640k/week. Assuming 30% of this in savings puts us at **$96k-192k/week saved**.

## Cross-cutting concerns

### Building the knowledge base

* Up-front work will be necessary to build a picture of what a “good” answer looks like. This might be done by either writing ideal correct answers for each question, or selecting multiple previously submitted high quality answers for each question as a baseline  
* Additionally, each question might also contain explicit grading guidelines, or rubric, to ensure complete answers are submitted

![Knowledge base population and usage](/images/kb.png)

### Technology to be used

* Would suggest something like AWS Bedrock as an all-inclusive RAG platform would be sufficient to load and inquire against the knowledge base  
* Not explicitly broken out in the diagrams are enablers like API Gateway, Lambda functions, S3, and the Foundational Model used for natural language processing


### AI Optionality

Note that the brief indicates a tentative desire to explore how AI might assist with Certifiable’s exam grading \- not a commitment to making it so. As such, trialling AI-assisted exam grading may prove to be ultimately fruitless. This could be because:

* Thinking of what a good answer looks like and/or identifying past good answers is too time consuming  
* The AI engine is consistently unable to score answers with high confidence  
* The hosting/execution cost of RAG is very high

As such, we must provide optionality to the AI-assistance, allowing for exams (or even a single exam) to be assessed via the legacy method. This can be accomplished by adding an assessor interface that both AI and human assessors can adhere to. Additionally, a defined percentage of incoming exams can be routed for AI assessment (assumed to be 100% in the first instance). This capability is housed in the **AI/Human Allocation** component in the diagrams

### Safeguarding

* Questions that are consistently answered incorrectly should be flagged to superusers for possible intervention  
* Spot-checking of “automatic pass” and “automatic fail” scores should be regularly performed as time allows. For example, harvesting back 6 hours of saved time/week would be a worthwhile investment to ensure the system hasn’t gone off the rails

## Assumptions

* Short-answer questions are graded on a pass/fail basis, with no partial credit being assigned for partially correct answers

## To do
For next round:
* Quantify approximate running costs to factor into the cost savings calculations
* From there, identify the critical threshold for effectiveness of AI grading where it moves from a cost to a savings
* ADRs on rolling their own RAG vs. using and off-the-shelf solution like Bedrock
* Make this page look lovely ✨