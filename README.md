# usfive - O'Reilly Architectural Katas: Spring 2025

## The Team
![usfive team members](images/banner.png)

Our team is comprised of 5 engineers from [ustwo](https://www.ustwo.com). Hello 👋 it's nice to meet you! We are:
* Belén Molina | [LinkedIn](https://www.linkedin.com/in/bel%C3%A9n-molina-del-campo-16476644) | [email](mailto:belenmolina@ustwo.com)  
* Graham Richards | [LinkedIn](https://www.linkedin.com/in/graham-richards-a48b276) | [email](mailto:grahamrichards@ustwo.com)  
* Joe McGuinness | [LinkedIn](https://www.linkedin.com/in/joemcguinness) | [email](mailto:joe.mcguinness@ustwo.com)  
* Nick Hegarty | [LinkedIn](https://www.linkedin.com/in/nick-hegarty-5a10439/) | [email](mailto:nickhegarty@ustwo.com)  
* Vinicios Neves | [LinkedIn](https://www.linkedin.com/in/vinny-neves) | [email](mailto:vinicios.neves@ustwo.com)  

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

* Each answer enters the **Manual Grading** component with a status value indicating it is "new" - it hasn't yet been processed/graded
* Answers in a "new" state trigger the **Grading Admin** component's attention, which will then use the **AI vs. Human Allocation** component to determine whether the answer should be graded via AI or a expert grader, based on thresholds described later (See "AI Optionality"). Any answer flagged for expert grading will receive a status of "expert grading required".  
* Any other answer will be set to a status of "AI grading required" and is submitted to the **AI Grader** with a prompt requesting a correctness score out of 10 to be explicitly assigned, as well as a confidence score out of 10  
* Thresholds for expert intervention must then be set to flag answers that require further consideration. For details on  scoring, please see the ADRs on correctness + confidence scoring linked below. Upon receipt of these grades from the **AI Grader**, the **Grading Admin** component will apply rules based on these thresholds and mark the answer's status to "AI pass", "AI fail", or "expert grading required".
* Answers marked as "expert grading required" should notify an expert grader (via email or otherwise), who will see their list of questions for grading when they log in via the existing web portal. Any questions that have been taken off their plate by the AI grader will not be in their queue.

Possible state transitions for an answer:
* new->expert grading required
* new->AI grading required
* AI grading required->AI pass
* AI grading required->AI fail
* AI grading required->expert grading required
* expert grading required->expert pass
* expert grading required->expert fail

![Test 1 Optimisation](/images/test1.jpg)

### Returns
Conservatively, we could estimate that 50% of answers can be automatically passed/failed using this approach. Depending on the nature \+ specificity of the short-answer questions, this could well be a higher percentage.

Doing the maths on potential gains from optimising test 1, then:

* In today’s terms, at 200 exams per week, assuming 100% submit test 1, roughly 600 hrs/week (200 exams \* 3 hrs each to assess) is spent on assessing the short-answer portion of this test. That equates to roughly 600 hours @ $50/hr \= $30,000/week spent. Assuming 50% of this in savings puts us at **$15,000/week**  
* Scales at 5-10x, this ranges from 3000-6000 hrs \= $150k-300k/week. Assuming 50% of this in savings puts us at **$75k-150k/week**.

### Relevant ADRs
* [Database and caching for Test 1 grading](adr/001-test-1-database.md)
* [Scoring thresholds](adr/002-scoring-thresholds.md)
* [Anti-cheating approach](adr/003-anti-cheater.md)
* [Confidence-based scoring and thresholds](adr/004-confidence-based-grading-process.md)
* [RAG vs. raw LLM approach](/adr/006-rag-vs-raw-llm.md)

## Optimising Test 2

Grading of Test 2 is similar in process to Test 1, but has at least 3 complicating factors:

1. The answers are longer and more nuanced. There are potentially more disparate acceptable answers.  
2. There is presumably an expectation of diagrams/charts to represent architectural thinking, which are non-trivial to interpret via AI.  
3. And there are multiple scenarios in play.

These distinctions can be addressed via the following mitigating techniques:

1. Additional care must be taken to upload multiple different acceptable answers for each scenario  
2. Ensure all diagram submissions are in the form of text-based models, such as PlantUML or Mermaid JS. This will allow for the simplest ingestion of diagrams in a form that is easy for our AI graders to reason about  
3. Similar to the grouping of related documentation for each question in Test 1, information can be grouped by scenario, with correct answers \+ scoring for each. 

And finally, the LLM query’s output should be modified to not only score as in Test 1, but also to provide a nuanced critique, highlighting similarities and differences against known good answers.

Scoring should again be on a 1-10 scale, with high scores automatically passed, low scores automatically failed, and middle scores flagged for human grading.

Our approach for scoring Test 2 answers needs to accommodate both the agreed upon list of criteria currently in use to arrive at a manual score, but we believe it can also uplift the scoring process by explicitly using different scoring agents to assess the architecture via different lenses. This approach is captured in [this ADR](adr/005-test-2-scoring.md).

![Test 2 Optimisation](/images/test2.jpg)

### Returns 
Conservatively, we could estimate that 30% of answers can be automatically passed/failed using this approach (a smaller percentage than Test 1, due to the more complex nature of the task).

Doing the maths on potential gains from optimising test 2, then:

* In today’s terms, at 200 exams per week, assuming 80% of users submit test 2 (based on potential attrition and failures on the first exam), roughly 600 hrs/week (160 exams \* 8 hrs each to assess) is spent on assessing the test. That equates to roughly 1280 hours @ $50/hr \= $64,000/week spent. Assuming 30% of this in savings puts us at **$19,200/week saved**  
* Scales at 5-10x, this ranges from 6400-12800 hrs \= $320k-640k/week. Assuming 30% of this in savings puts us at **$96k-192k/week saved**.

### Relevant ADRs
* [Scoring thresholds](adr/002-scoring-thresholds.md)
* [Anti-cheating approach](adr/003-anti-cheater.md)
* [Confidence-based scoring and thresholds](adr/004-confidence-based-grading-process.md)
* [Agent scoring approach](adr/005-test-2-scoring.md)
* [RAG vs. raw LLM approach](/adr/006-rag-vs-raw-llm.md)

## Cross-cutting concerns

### Technology to be used

* While our initial submission included the use of Retrieval Augmented Generation, we have pivoted away from this approach (see ADR linked below), so are no longer suggesting the use of AWS Bedrock or similar.
* Not explicitly broken out in the diagrams are enablers like API Gateway, Lambda functions, S3, and the Foundational Model used for natural language processing


### AI Optionality

Note that the brief indicates a tentative desire to explore how AI might assist with Certifiable’s exam grading \- not a commitment to making it so. As such, trialling AI-assisted exam grading may prove to be ultimately fruitless. This could be because:

* Thinking of what a good answer looks like and/or identifying past good answers is too time consuming  
* The AI engine is consistently unable to score answers with high confidence  
* The hosting/execution cost of AI components is very high

As such, we must provide optionality to the AI-assistance, allowing for exams (or even a single exam) to be assessed via the legacy method. This can be accomplished by adding an assessor interface that both AI and human assessors can adhere to. Additionally, a defined percentage of incoming exams can be routed for AI assessment (assumed to be 100% in the first instance). This capability is housed in the **AI/Human Allocation** component in the diagrams

### Safeguarding

* Questions that are consistently answered incorrectly should be flagged to superusers for possible intervention  
* Spot-checking of "AI pass” and “AI fail” scores should be regularly performed as time allows. For example, harvesting back 6 hours of saved time/week would be a worthwhile investment to ensure the system hasn’t gone off the rails
* We have also recommended a set of anti-cheating guardrails to ensure that the AI grading components are not gamed by malicious users

### Relevant ADRs
* [Anti-cheating approach](adr/003-anti-cheater.md)
* [RAG vs. raw LLM approach](/adr/006-rag-vs-raw-llm.md)

## Assumptions

* Short-answer questions are graded on a pass/fail basis, with no partial credit being assigned for partially correct answers
* Non-functional requirements related to raw throughput and traditional problems of scale that are frequent hallmarks of real-time systems have been explicitly de-prioritised in our solution, including:
  * Availability
  * Scalability
  * Performance/Response Time
* Whereas these other common NFRs, which are pertinent to the core aims of the system, are prioritised in this solution:
  * Security
  * Reliability
  * Maintainability
* Running cost has been considered in the sections above, but is not a highly weighted NFR given that even a cursory glance at the numbers confirm the overwhelming advantage that AI-assisted processes will have to the exam scoring process, even if only a very low percentage of tests were automatically graded by the AI-supported components
