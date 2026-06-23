# Planning: CareerSignalMeter

## Community

I chose **r/datascience** as my online community. I chose this community because it contains active discussions about data science careers, technical skills, industry changes, workplace realities, and the impact of AI on the field. Many people in the community are trying to understand what data science work actually looks like, what skills matter, and how the career path is changing.

This community is a good fit for a classification task because the discourse is varied but still centered around a shared professional context. Posts are not all the same type: some focus on technical skills, some ask for career direction, some describe industry frustrations, and some share personal experiences. These distinctions are meaningful because beginners often use this community to make decisions about what to learn, how to interpret the job market, and what kind of data science role they want to pursue.

## Labels

### `technical_skill_advice`

This label applies to posts or comments that mainly discuss concrete tools, methods, or technical practices used in data science work. These posts often mention skills such as SQL, Python, statistics, machine learning, dashboards, workflow testing, model evaluation, deployment, or other practical technical decisions.

Example posts:

* “Ideas for testing data science workflows on self-contained projects?”
* “What tools or workflows should I use to improve reproducibility in my data science projects?”

### `industry_reality_check`

This label applies to posts or comments that describe broader realities of data science work, the DS job market, or how data teams operate inside companies. These posts are usually less about one person’s next career move and more about patterns such as stakeholders ignoring insights, poor data quality, unrealistic expectations, deployment issues, AI disruption, or the gap between data science as advertised and data science as practiced.

Example posts:

* “After 5 years in data science, I’m starting to realize most insights we deliver are completely ignored. Is this normal?”
* “What is the biggest challenge you face in data science projects?”

### `career_path_advice`

This label applies to posts or comments focused on career direction, role transitions, specialization, promotions, job searching, or deciding what professional step to take next. Even when the author includes personal background, the post belongs here if the main purpose is to ask for or provide guidance about a data science career path.

Example posts:

* “Data directors, what’s your next step?”
* “Identity crisis — a generalist dilemma.”

### `personal_experience`

This label applies to posts or comments where the main purpose is to share the author’s own story, reflection, or lessons learned from their time in data science. These posts may contain career or industry insights, but they are framed primarily through one person’s experience rather than as general advice or a broad claim about the field.

Example posts:

* “A decade of being an average data scientist: my experience.”
* “My personal journey from analyst to data scientist and what I learned.”

## Hard Edge Cases

The hardest edge case will be posts that combine **personal experience** with either **career path discussion** or **industry reality check**. Many r/datascience posts begin with the author’s background and then use that experience to ask for advice or make a broader observation about the field.

I will handle these cases by labeling the post based on its main function. If the personal story is mainly used to ask what career step to take next, I will label it `career_path_advice`. If the personal story is mainly used to describe a broader pattern about data science work, companies, stakeholders, or the job market, I will label it `industry_reality_check`. If the post is mainly a retrospective story without a clear request for advice or broad industry claim, I will label it `personal_experience`.

Decision rule:

* “What should I do next?” → `career_path_advice`
* “This is what the field/company reality is like” → `industry_reality_check`
* “This is what happened to me and what I learned” → `personal_experience`
* “Here is what tool/method/skill to use” → `technical_skill_advice`

## Data Collection Plan

I will collect examples from public posts and comments in **r/datascience**. I will collect a minimum of **200 total examples**, aiming for about **50 examples per label**:

* `technical_skill_advice`: 50 examples
* `industry_reality_check`: 50 examples
* `career_path_advice`: 50 examples
* `personal_experience`: 50 examples

I will save the examples in a CSV file with at least two columns:

```csv
text,label
```

If one label is underrepresented after the first 200 examples, I will collect more examples targeted toward that label before finalizing the dataset. My first goal is to keep the label distribution roughly balanced, but I will allow some imbalance if the community naturally contains more of one type of discussion. If needed, I will collect up to around 100 examples for a difficult or underrepresented label, then downsample or document the imbalance in the README.

## Evaluation Metrics

I will evaluate the model using **overall accuracy**, **per-class precision**, **per-class recall**, **per-class F1 score**,and probably a **confusion matrix**.

Accuracy is useful because it gives a simple overall measure of how often the model predicts the correct label. However, accuracy alone is not enough for this task because the label distribution may be uneven. For example, if `industry_reality_check` appears more often than the other labels, a model could get decent accuracy by over-predicting that label while still failing to recognize smaller or more ambiguous categories.

Per-class precision, recall, and F1 are important because I need to know how well the model handles each specific type of career discussion. Precision will show whether the model is too broad when assigning a label, while recall will show whether the model misses examples that belong to that label. F1 is useful because it balances precision and recall for each category.


I will also use a confusion matrix to identify which labels are most often mixed up. This is important because some categories naturally overlap, especially `personal_experience` and `career_path_advice`, or `personal_experience` and `industry_reality_check`. The confusion matrix will help me understand whether the errors come from weak label definitions, ambiguous posts, class imbalance, or genuinely difficult boundaries in the community discourse.

## Definition of Success

A genuinely useful classifier should do more than achieve a high overall accuracy. It should correctly recognize each major type of r/datascience career discussion and make interpretable mistakes when posts are ambiguous.

For this project, I would consider the classifier successful if it reaches at least **70% overall accuracy** on the test set. I would also want each label to have a reasonable F1 score, ideally above **0.60**, so that the model is not only learning the largest or easiest category.

For deployment in a real community tool, “good enough” would mean the classifier can be used as a lightweight sorting or filtering assistant, not as a final authority. For example, it could help users filter posts by technical advice, career path discussion, industry reality, or personal experience. I would not consider it reliable enough for moderation or high-stakes decisions unless it performed consistently across labels and was tested on a larger, more diverse dataset.

The most important success condition is that the model’s errors are understandable. If the model mainly struggles with genuinely ambiguous posts, such as personal stories that also ask for career advice, that would be acceptable. If it consistently confuses clearly different categories, such as technical skill advice and personal experience, then the label definitions or training data would need to be revised.



## AI Tool Plan

This project does not mainly require AI tools for code generation. Instead, I will use AI tools to support the data design, annotation, and evaluation process. My use of AI will focus on three stages: label stress-testing, annotation assistance, and failure analysis.

### 1. Label Stress-Testing

Before annotating the full dataset, I will use an AI tool such as ChatGPT to stress-test my label definitions. I will give the AI my four labels, their definitions, and my known hard edge case: posts that combine personal experience with career path discussion or industry reality observations.

I will ask the AI to generate 5–10 realistic r/datascience-style posts that sit at the boundary between two labels. For example, I will specifically test cases between `personal_experience` and `career_path_advice`, and between `personal_experience` and `industry_reality_check`.

If the AI generates examples that I cannot classify cleanly using my current rules, I will revise the label definitions before annotating my 200 examples. This will help make the labels more mutually exclusive and reduce inconsistency during annotation.

Example prompt:

```text
Here are my labels for classifying r/datascience posts:

1. technical_skill_advice: ...
2. industry_reality_check: ...
3. career_path_advice: ...
4. personal_experience: ...

My hardest edge case is that personal experience often blends with career advice or industry observation.

Generate 10 realistic r/datascience-style posts that sit at the boundary between two labels. For each one, identify the two possible labels and explain why it is ambiguous.
```

### 2. Annotation Assistance

I may use an LLM to pre-label a batch of examples before reviewing them myself. If I do this, the AI-generated label will not be treated as the final label. I will manually review every example and decide the final annotation based on my label definitions.

To keep this transparent, I will track AI assistance separately in my dataset. My CSV may include columns such as:

```csv
text,ai_prelabel,final_label,reviewed_by_human
```

For example:

```csv
"I have worked in DS for five years and most stakeholders ignore insights...",personal_experience,industry_reality_check,yes
```

This will allow me to disclose clearly that AI was used only as a pre-labeling assistant, while the final labels were reviewed and finalized by me.

If I find that the AI pre-labels are frequently wrong or inconsistent, I will stop using AI pre-labeling and continue with fully manual annotation.

### 3. Failure Analysis

After training and evaluating the model, I will collect the model’s wrong predictions. For each wrong prediction, I will compare the original text, the true label, and the model’s predicted label.

I will use an AI tool to help identify possible patterns in the model’s errors. For example, I will ask whether the model is confusing certain label pairs, over-relying on surface-level keywords, struggling with posts that include personal background, or failing because of class imbalance.

Example prompt:

```text
Here are my model's wrong predictions. Each example includes the text, the true label, and the predicted label.

Identify common error patterns. Are the mistakes caused by ambiguous label definitions, overlapping discourse types, class imbalance, or surface-level keywords? Group the errors into 3–5 patterns and suggest how I should explain them in my evaluation.
```

I will not automatically trust the AI’s analysis. I will verify any suggested patterns by going back to the actual wrong predictions and checking whether the pattern appears across multiple examples. I will only include failure patterns in my final evaluation if I can confirm them myself from the model outputs.

### AI Use

AI tools may be used to stress-test labels, pre-label examples before human review, and help summarize error patterns after evaluation. However, the final label definitions, final annotations, and final interpretation of the model’s performance will be my responsibility.
