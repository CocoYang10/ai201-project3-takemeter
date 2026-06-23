# ai201-project3-takemeter: Classifying r/datascience Career Discourse

## Project Overview

This project builds **TakeMeter**, a text classifier for posts from **r/datascience**. The goal is to classify each post into one of four discourse-function labels:

* `technical_skill_advice`
* `industry_reality_check`
* `career_path_advice`
* `personal_experience`

I chose this project because many students, early-career data workers, and career switchers use online communities like r/datascience to make decisions about what to learn, how to understand the job market, and what career path to pursue. From my own experience and from conversations with people around me, career communities are useful but also noisy. Someone looking for technical skill advice may end up reading personal stories, broad industry complaints, or unrelated career anxiety posts. A classifier like this could help users filter the community more precisely and find the type of information they actually want.

This task is not just about identifying topic keywords. It is about identifying the **main function** of a post: whether the post is giving technical guidance, describing industry reality, discussing career direction, or sharing personal experience.

## Community

I chose **r/datascience** because it contains active discussions about data science careers, technical skills, industry changes, workplace realities, and the impact of AI on the field. The community is a good fit for classification because the discourse is varied but still centered around a shared professional context.

Posts in this community are not all the same type. Some ask what technical skills matter, some ask about career transitions, some describe frustration with stakeholder expectations or job-market changes, and some are reflective personal stories. These distinctions are meaningful because different users come to the community with different needs.

## Labels

### `technical_skill_advice`

This label applies to posts or comments mainly about concrete data science tools, methods, workflows, or technical skills. These posts may discuss SQL, Python, statistics, machine learning, dashboards, evaluation, deployment, model workflows, or technical projects.

Representative example:

> “Ideas for testing data science workflows on self-contained projects?”

### `industry_reality_check`

This label applies to posts or comments that describe broader realities of data science work, the DS job market, workplace dynamics, stakeholder behavior, AI impact, or the gap between the idealized version of data science and the actual job.

Representative example:

> “After 5 years in data science, I’m starting to realize most insights we deliver are completely ignored. Is this normal?”

### `career_path_advice`

This label applies to posts or comments focused on career direction, role transitions, specialization, promotions, job searching, or deciding what professional step to take next.

Representative example:

> “Identity crisis — a generalist dilemma.”

### `personal_experience`

This label applies to posts or comments where the main purpose is to share the author’s own story, reflection, struggle, or lessons learned from their experience in data science.

Representative example:

> “A decade of being an average Data Scientist! My personal experience.”

## Dataset

I collected public r/datascience posts using a Playwright-based script rather than the Reddit API. The final dataset contained **224 labeled examples** across four labels.

Final label distribution:

| Label                    |   Count |
| ------------------------ | ------: |
| `career_path_advice`     |      61 |
| `industry_reality_check` |      57 |
| `personal_experience`    |      55 |
| `technical_skill_advice` |      51 |
| **Total**                | **224** |

The dataset was split into train, validation, and test sets. The test set contained **34 examples**.

## Annotation Rules and Hard Edge Cases

The hardest edge case was that many posts combined **personal experience**, **career path discussion**, and **industry observation** in the same text. For example, a user might describe their own career history, express uncertainty about their next step, and also make a broader claim about the data science job market.

I handled these cases by labeling the post based on its **main function**:

* If the post mainly asked “What should I do next?” I labeled it `career_path_advice`.
* If the post mainly described a broader pattern about data science work, companies, stakeholders, or the job market, I labeled it `industry_reality_check`.
* If the post was mainly a retrospective story or reflection, I labeled it `personal_experience`.
* If the post mainly discussed tools, methods, workflows, or technical skills, I labeled it `technical_skill_advice`.

This rule helped keep the labels mutually exclusive, but the final model results showed that these boundaries were still difficult for a small model to learn.

## Models

I compared two models:

### Zero-shot LLM Baseline

The baseline model used **Groq / Llama-3.3-70B-Versatile**. This model was not fine-tuned on my dataset. It only received my label definitions in the prompt and then classified the test examples directly.

### Fine-tuned DistilBERT

The fine-tuned model used **`distilbert-base-uncased`** with a classification head. I used the default notebook settings:

* Base model: `distilbert-base-uncased`
* Number of labels: 4
* Epochs: 3
* Learning rate: 2e-5
* Train batch size: 16
* Evaluation batch size: 32

## Evaluation Metrics

I used overall accuracy, per-class precision, recall, F1 score, macro F1, and a confusion matrix.

Accuracy gives a basic measure of how often the model predicts the correct label, but it is not enough for this task because the labels are not equally easy. A model could get a decent accuracy by over-predicting a common or broad category, while still failing to recognize harder categories.

Per-class precision, recall, and F1 are important because I need to know whether the model can recognize each type of r/datascience discourse. Macro F1 is especially useful because it treats each label equally instead of allowing the largest or easiest class to dominate the evaluation.

The confusion matrix is also important because this task has naturally overlapping labels. It shows which label boundaries the model failed to learn.

## Results

### Baseline Model Results

The zero-shot Llama baseline performed strongly:

| Metric   | Score |
| -------- | ----: |
| Accuracy | 0.735 |
| Macro F1 | 0.740 |

Per-class metrics:

| Label                    | Precision | Recall |   F1 | Support |
| ------------------------ | --------: | -----: | ---: | ------: |
| `technical_skill_advice` |      0.64 |   0.88 | 0.74 |       8 |
| `industry_reality_check` |      1.00 |   0.75 | 0.86 |       8 |
| `career_path_advice`     |      0.78 |   0.70 | 0.74 |      10 |
| `personal_experience`    |      0.62 |   0.62 | 0.62 |       8 |

The baseline result was much stronger than expected. It reached almost the level I originally hoped the fine-tuned classifier might reach. This suggests that the label definitions were understandable to a strong general-purpose LLM.

### Fine-tuned DistilBERT Results

The fine-tuned DistilBERT model performed much worse:

| Metric              |   Score |
| ------------------- | ------: |
| Accuracy            |   0.353 |
| Correct predictions | 12 / 34 |
| Wrong predictions   | 22 / 34 |

For a four-class task, random guessing would be around 0.25 accuracy. The fine-tuned model performed above random, but only slightly, and it was far below the Llama baseline.

### Baseline vs. Fine-tuned Comparison

| Model                            | Accuracy |
| -------------------------------- | -------: |
| Zero-shot baseline: Groq / Llama |    0.735 |
| Fine-tuned DistilBERT            |    0.353 |

The fine-tuned model did not outperform the zero-shot LLM baseline. This is likely because the classification task requires higher-level semantic understanding, while the dataset only contained 224 examples. DistilBERT is much smaller than Llama and had limited data to learn subtle discourse-level distinctions.

## Confusion Matrix

The fine-tuned model produced the following confusion matrix on the test set.

Rows are true labels. Columns are predicted labels.

| True \ Predicted         | `technical_skill_advice` | `industry_reality_check` | `career_path_advice` | `personal_experience` |
| ------------------------ | -----------------------: | -----------------------: | -------------------: | --------------------: |
| `technical_skill_advice` |                        0 |                        4 |                    4 |                     0 |
| `industry_reality_check` |                        0 |                        5 |                    3 |                     0 |
| `career_path_advice`     |                        1 |                        1 |                    6 |                     2 |
| `personal_experience`    |                        0 |                        1 |                    6 |                     1 |

The confusion matrix shows that the model over-predicted `career_path_advice`. Many examples from other categories, especially `personal_experience`, were classified as career path advice. This suggests that the model learned that many r/datascience posts are career-related, but it did not learn the finer distinction between career guidance, personal reflection, industry observation, and technical discussion.

## Failure Analysis

The fine-tuned DistilBERT model made 22 wrong predictions out of 34 test examples. Several clear error patterns appeared.

### Pattern 1: Over-prediction of `career_path_advice`

The model frequently predicted `career_path_advice`, even when the true label was `personal_experience`, `industry_reality_check`, or `technical_skill_advice`.

This suggests that the model learned a broad career-related signal from the dataset, but it did not learn the more specific function of each post. In r/datascience, many posts mention jobs, interviews, layoffs, skills, or career uncertainty, so the model may have treated “career context” as enough evidence for the career label.

### Pattern 2: `personal_experience` vs. `career_path_advice`

This was one of the hardest boundaries. For example:

> “A decade of being an average Data Scientist! My personal experience.”

True label: `personal_experience`
Predicted label: `career_path_advice`

This post is framed as a personal reflection, but it still discusses a data science career. The model likely focused on the career-related language rather than the autobiographical function of the post.

Another example:

> “Disillusioned with the field of data science…”

True label: `personal_experience`
Predicted label: `career_path_advice`

This post describes the author’s own disappointment and uncertainty. However, because it is about dissatisfaction with a career field, the model treated it as career-path discussion.

This confirms the hard edge case from my planning document: personal experience often blends with career advice and industry observation.

### Pattern 3: `technical_skill_advice` was too broad

The model also struggled with `technical_skill_advice`. For example:

> “GBNet: fit XGBoost inside PyTorch”

True label: `technical_skill_advice`
Predicted label: `industry_reality_check`

This example shows a weakness in my label design. The `technical_skill_advice` label included several types of technical content: direct advice, technical project sharing, research links, and workflow discussions. These are meaningful to users looking for technical content, but they do not always share the same language pattern.

A short technical title like “GBNet: fit XGBoost inside PyTorch” may be clear to a human reader as technical content, but it does not contain much context for a small model to classify reliably.

### Pattern 4: `industry_reality_check` vs. `career_path_advice`

Some posts about industry trends were predicted as career advice. For example:

> “So what do y’all think of the Block layoffs?”

True label: `industry_reality_check`
Predicted label: `career_path_advice`

This post discusses layoffs and the direction of the industry, but it also relates directly to job searching and career planning. That makes the boundary difficult.

Another example:

> “‘Full stack’ data science…”

True label: `industry_reality_check`
Predicted label: `career_path_advice`

This post describes a broader industry trend: more roles requiring end-to-end production skills. However, this trend also affects individual career planning, so the model treated it as career path advice.

### Pattern 5: Low confidence across wrong predictions

Many wrong predictions had confidence scores around 0.25–0.27. For a four-class task, this is close to uniform probability. This suggests that the model was not confidently choosing the wrong labels; rather, it had not learned stable decision boundaries.

The issue was probably not a single mislabeled category. It was a combination of small training data, overlapping labels, short or low-context examples, and a task that requires higher-level semantic understanding.

## Sample Classifications

The table below shows representative model behavior. Some rows are wrong predictions from the fine-tuned model output. The correct-prediction row should be replaced with an exact recovered model output if `sample_classifications.csv` can be exported later.

| Text summary                                                                         | True label               | Predicted label          | Confidence | Analysis                                                                                                                                                  |
| ------------------------------------------------------------------------------------ | ------------------------ | ------------------------ | ---------: | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| A post about stakeholder insights being ignored after several years in data science. | `industry_reality_check` | `industry_reality_check` |        0.28 | Representative correct classification. This fits the label because the post describes a broader workplace pattern, not only one personal career decision. |
| “A decade of being an average Data Scientist! My personal experience.”               | `personal_experience`    | `career_path_advice`     |       0.26 | Incorrect. The model treated a first-person career reflection as career advice, likely because the post discusses a data science career.                  |
| “GBNet: fit XGBoost inside PyTorch.”                                                 | `technical_skill_advice` | `industry_reality_check` |       0.26 | Incorrect. The title is technical, but short and context-light. The model failed to recognize it as technical content.                                    |
| “So what do y’all think of the Block layoffs?”                                       | `industry_reality_check` | `career_path_advice`     |       0.26 | Incorrect. Layoffs are an industry-level issue, but they also connect to job searching, so the model confused industry reality with career path advice.   |
| “‘Full stack’ data science.”                                                         | `industry_reality_check` | `career_path_advice`     |       0.27 | Incorrect. The post discusses a broad industry shift in DS role expectations, but the model interpreted it as individual career planning.                 |

## What the Model Captured vs. What I Intended

I intended the model to capture the main function of each r/datascience post: whether the post was technical advice, industry reality, career path discussion, or personal experience.

The model partially captured that many posts in r/datascience are career-related. However, it did not learn the finer distinctions I intended. In particular, it often collapsed different types of career-related discourse into `career_path_advice`.

This revealed an important gap between my label definitions and the actual structure of the data. Human career discourse is not cleanly separable. A single post can contain a personal story, a career question, and an industry observation at the same time. Forcing each post into exactly one label made the task harder.

The model also showed that labels meaningful to users are not always easy for a model to learn. From a user perspective, it is useful to separate technical advice from career advice or personal stories. But from a text-pattern perspective, many posts share similar language about jobs, skills, uncertainty, and experience.

## Spec Reflection

The planning specification helped me define the project before implementation. Writing the label definitions, edge-case rules, data collection plan, and evaluation metrics before training gave the project a clear structure. In particular, the edge-case rule for separating `personal_experience`, `career_path_advice`, and `industry_reality_check` helped me make more consistent annotation decisions.

However, the implementation also revealed a limitation in the original spec. I originally framed the task as a single-label classification problem, where every post belongs to exactly one category. After collecting and labeling the data, I realized that many r/datascience posts are multi-functional. One post can share a personal experience, ask for career advice, and make an industry observation at the same time.

The implementation also diverged from the ideal version of the spec because `technical_skill_advice` turned out to be broader than expected. It included direct technical advice, technical project sharing, research links, and workflow discussions. These examples were meaningful for users looking for technical content, but they did not always share the same language patterns.

If I continued this project, I would revise the spec before collecting a larger dataset. I would either make the labels narrower or redesign the task as a multi-label classification problem, where one post could receive multiple tags such as both `personal_experience` and `career_path_advice`.

## Future Improvements

The biggest improvement would be to collect a larger and more diverse dataset. The final dataset had 224 examples, which was enough to run the pipeline but probably not enough for a small model like DistilBERT to learn subtle discourse-level distinctions.

I would also revise the label design. Some labels were meaningful from a user-search perspective but difficult from a modeling perspective. For example, users may want to separate career advice from industry reality, but in real posts those two functions often appear together.

In a future version, I would consider multi-label classification instead of forcing each post into exactly one category. This would better reflect how people actually write in career communities. A post could be tagged as both `personal_experience` and `career_path_advice`, or both `industry_reality_check` and `career_path_advice`.

Finally, I would run a smaller pilot labeling round before scaling up. After labeling 30–50 examples, I would check which labels overlap most often and revise the definitions before collecting the full dataset.

## AI Usage

I used AI tools in two main ways during this project.

First, I used ChatGPT to help interpret unexpectedly poor fine-tuning results. After my fine-tuned DistilBERT model performed much worse than the Llama baseline, I discussed possible causes including label mapping issues, small data size, model limitations, overlapping labels, and unstable decision boundaries. I used this discussion to decide what to check and how to explain the result.

Second, I used ChatGPT to help identify patterns in the wrong predictions and confusion matrix. I provided the true labels, predicted labels, confidence scores, and example texts, then asked for recurring error patterns. I reviewed the suggested patterns myself and kept the ones supported by multiple examples, such as over-prediction of `career_path_advice` and confusion between `personal_experience` and `career_path_advice`.

I did not use AI to replace my final judgment. The final labels, final interpretation of the results, and final written analysis were reviewed and decided by me.

## Conclusion

The main result of this project is that the zero-shot Llama baseline performed much better than the fine-tuned DistilBERT model. The Llama baseline achieved 0.735 accuracy, while the fine-tuned DistilBERT model achieved 0.353 accuracy.

This result suggests that the task requires higher-level semantic understanding and that the dataset was too small for DistilBERT to learn reliable boundaries. The project also showed that label design is one of the hardest parts of applied machine learning. The model’s failures were not random; they revealed real ambiguity in the task, especially around personal experience, career advice, and industry reality.

Even though the fine-tuned model did not perform well, the failure was useful. It showed that a better version of this project would need more data, tighter label definitions, and possibly a multi-label structure that better matches how people actually write in online career communities.
