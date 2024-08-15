## CS728 - Cross-lingual In-context Learning in Multilingual LLMs

## Proposed Solution

For this assignment, we used the `mt0-large` and `mt0-xxl` models, which have 1.2B and 13B parameters, respectively, and have been specifically designed to perform tasks that can be expressed in natural language. The reason for this choice was that the suggested XGLM models have a specific format for prompting the model to perform evaluation tasks, which was restrictive for this assignment. We also experimented with the `flan-t5-large` model. The model performed well for English, but poorly for the other languages, so we decided not to use it in further evaluations.

## Model Setup

Both models were evaluated in a few-shot setup with example context in one language and corresponding tasks to be performed in another. This was done for all three pairs from the [English, Russian, and French] test cases.

### For XNLI Dataset

To evaluate the XNLI dataset, we set up our model as follows:

- **Zero-shot setup:** We provide the instructions with the statement first, followed by the query where the entities are enclosed in `{{ }}` braces. The model is expected to output one of three labels based on this prompt.
  
- **In-context setup:** For both random alignment and task-alignment setup, we provide examples in the source language followed by the query in the target language[1].

For this dataset, we define the labels as follows:
- **True** - Entailment
- **False** - Contradiction
- **Inconclusive** - No relation

### For SMiLER Dataset

To evaluate the SMiLER dataset, we set up our model as follows:

- **Zero-shot setup:** We provide the instructions with the statement first, followed by a list of all possible relations and then the query, where the entities are enclosed in `{{ }}` braces. The model is expected to output one of all possible labels based on this prompt.
  
- **In-context setup:** For both random alignment and task-alignment setup, we provide examples in the source language followed by the relations in the source language. We then include the task-alignment string in the source language with a mapping of every relation from the source language to the corresponding target language[1]. Finally, the query is attached in the target language.

We had also tried a logits-based setup as proposed in the paper[], but the results were not satisfactory, so we discarded it and proceeded with this generation-based approach.

A prompt for in-context learning looks like this, with 5 examples used:
- We give examples and context in the source language and ask the query in the target language.
  
  **Consider a domain where only the following relations can exist between any two entities:**
  - `{{is member of}}`
  - `{{movie has director}}`
  - …
  
  **Given the following examples:**
  - Statement: `{{Ernesto "Che" Guevara ( (CHAY gə-VAH-rə);Spanish: [ˈtʃe ɣeˈβaɾa] 14 June 1928…}}`
  - …
  - Statement: `{{North West Coastal Highway is a generally north-south…}}`
  - …

  **What relation best describes the relation between `{{North West Coastal Highway}}` and `{{highway}}` from the list of relations given above?**

A prompt for in-context learning with task alignment looks like this, with 5 examples used:
- We give examples and context in the source language and ask the query in the target language. We also provide the corresponding translations for each relation from source to target, as shown below.

  **Consider a domain where only the following relations can exist between any two entities:**
  - `{{is member of}}`
  - `{{movie has director}}`
  - …
  
  **Given the following examples:**
  - Statement: `{{Ernesto "Che" Guevara ( (CHAY gə-VAH-rə);Spanish: [ˈtʃe ɣeˈβaɾa] 14 June 1928…}}`
  - …

  **Given:**
  - In English `{{premier produit}}` means `{{first product}}`
  - In English `{{est membre de}}` means `{{is member of}}`
  - In English `{{film réalisé par}}` means `{{movie has director}}`
  - …
  
  **Statement:**
  - `{{North West Coastal Highway is a generally north-south…}}`

  **What relation best describes the relation between `{{North West Coastal Highway}}` and `{{highway}}` from the list of relations given above?**

## Results & Analysis

### For XNLI Dataset

#### For Zero-Shot Inference:

The following results display the number of accurate predictions out of 200 test cases each.

| Model      | English | French | Russian |
|------------|---------|--------|---------|
| mt0-base   | 83      | 82     | 28      |
| mt0-large  | 90      | 95     | 60      |
| mt0-XXL    | 141     | 131    | 47      |

The numbers in each cell represent the total number of accurate predictions out of 200 instances. By accurate, we mean situations where the primary word is present in the output, i.e., there is some non-trivial overlap between the ground truth label and the predicted label. Please note that we consider instruction violation, i.e., the situations where the model outputs something completely different from the set of labels or outputs a string in a different language, as an inaccurate prediction for our analysis.

#### For Cross-Lingual Few-Shot Inference:

| Model      | en-ru | ru-en | en-fr | fr-en | fr-ru | ru-fr |
|------------|-------|-------|-------|-------|-------|-------|
| mt0-base   | 78    | 8     | 77    | 75    | 75    | 12    |
| mt0-large  | 82    | 24    | 88    | 83    | 78    | 26    |
| mt0-XXL    | 110   | 11    | 121   | 94    | 80    | 12    |

#### For Task-Aligned Few-Shot Inference:

| Model      | en-ru | ru-en | en-fr | fr-en | fr-ru | ru-fr |
|------------|-------|-------|-------|-------|-------|-------|
| mt0-base   | 78    | 58    | 78    | 60    | 73    | 12    |
| mt0-large  | 77    | 60    | 89    | 74    | 73    | 14    |
| mt0-XXL    | 112   | 9     | 128   | 86    | 81    | 9     |

The models performed better using zero-shot inference than in-context random and task-aligned few-shot setups. The cross-lingual examples appear to degrade the performance in all cases: base, large, and XXL. Some setups see drastic degradation as well. The models show a lexical bias towards English, i.e., the outputs, even when degraded, are better for English queries compared to other cross-lingual setups. The models generally improve in performance as the size of the models increases. This observation is consistent with all the tasks that we experimented with. However, the performance decreased drastically when the models used Russian examples and queries in other languages.

In situations where several inconclusive outputs appear, many of those are English outputs, which might be semantically correct in terms of the label but can’t be attributed to correct output as we expect the output to be in the corresponding target language. This phenomenon is observed throughout our experiments, indicating the model’s alignment towards English in case of uncertainty in action. It was also observed that Russian test cases are prone to higher hallucination instances compared to the other two languages. Moreover, the performance seems to degrade when in-task alignment is used in such instances across models.

### For SMiLER Dataset

#### For Zero-Shot Inference:

The following results display the number of accurate predictions out of 200 test cases each. Please note that the Russian instances didn’t have 200 but 132 instead.

| Model      | English   | French   | Russian |
|------------|-----------|----------|---------|
| mt0-base   | 2/200     | 1/200    | 0/132   |
| mt0-large  | 48/200    | 46/200   | 14/132  |
| mt0-XXL    | 139/200   | 154/200  | 110/132 |

It was observed that the base model wasn’t performing well at all. It gave random strings as output, which didn’t even adhere to the language of the query or the examples. Furthermore, it gave outputs that were sometimes the entity itself or some random substring from the input statement. In summary, since the base model couldn’t perform the zero-shot task well, we didn’t use it for further tasks. The performance of the XXL model was surprisingly good for the Russian language compared to English in terms of accuracy. We believe this is happening due to the availability of fewer unique relations for this language. The observation is aligned with the fact that French also performs better than English as there are comparatively fewer relations in the former language.

#### For Cross-Lingual Few-Shot Inference:

| Model      | en-ru | ru-en | en-fr | fr-en | fr-ru | ru-fr |
|------------|-------|-------|-------|-------|-------|-------|
| mt0-large  | 0     | 25    | 3     | 4     | 0     | 40    |
| mt0-XXL    | 112/132 | 98/200 | 144/200 | 118/200 | 117/132 | 114/200 |

The models improve in performance as the size of the models increases. This observation is consistent with all the tasks that we experimented with. The large model performs way worse compared to the XXL model. On manually observing the outputs, we found out that the large model was hallucinating heavily and didn’t adhere to the problem's constraints in many cases. It was also observed that the large model doesn’t work well for in-context learning-based setups. We tried modifying the prompts a bit to see if the results changed but we didn’t find any significant changes in the results. However, the XXL model didn’t degrade drastically compared to the large model. There was a loss of accuracy compared to the zero-shot counterparts. In the instances where Russian cases were to be predicted based on other language examples, the model showed a slight improvement, suggesting that Russian, a low-resource language, might benefit from the examples in English and French.

#### For Task-Aligned Few-Shot Inference:

| Model      | en-ru | ru-en | en-fr | fr-en | fr-ru | ru-fr |
|------------|-------|-------|-------|-------|-------|-------|
| mt0-large  | 2     | 0     | 3     | 1     | 13    | 0     |
| mt0-XXL    | 111/132 | 32/200 | 138/200 | 97/200 | 104/132 | 23/200 |

Here it was observed that the degradation was higher than the random example-based in-context learning setup. Even the Russian instances took a dip in terms of accuracy. Furthermore, it was observed that the model outputs were out of domain (all possible relations) where the model sometimes predicted the output words without any spaces as a single word or the correct relation but in a different language (mainly English or the Source language) which we didn’t assign as correct output.

## Insights and Conclusion

Upon conducting all these experiments, we could make a few observations aligned with the results observed in the tables earlier in the report. The foremost observation was that the zero-shot setup always gave the best results compared to the multilingual setups. We believe a large contribution to this phenomenon is due to the fine-tuning objective of the considered model. The models were trained with instances where the prompts were monolingual, covering various languages, i.e., the models didn’t get to learn enough of cross-lingual contexts during their training or fine-tuning, and thus, the in-context examples caused more harm than good.

We also hypothesized that the models’ performance degrades even further during task-aligned in-context examples-based prompting as the model might be confused with the description of what the relations/labels corresponding to the target language mean in the source language. Thus, it causes the model to output the labels in a language different from the intended one, which was frequently observed in all cases.

We expected that the models would show Constructive Superposition in the cross-lingual settings, helping each other increase their performance; instead, there seems to be a destructive superposition happening, which causes the models to perform worse when provided with more examples. We hypothesize that the model does not understand the cross-lingual prompt setting for the given setup well.

To further investigate this hypothesis, we looked into the training and fine-tuning setup of the model mt0 to understand why the models behaved as they did during our experiments and found some details related to their objective in this context. The models were trained on the MC4 dataset, which has a significantly high share of English but a considerable amount of Russian and French data. Interestingly, the fine-tuning dataset, i.e., xP3, didn’t have any Russian instances whatsoever, which might explain why the smaller models perform terribly for Russian instances. This would also explain why the Russian test instances improved their performance when prompts were given in French or English, as the model didn’t fine-tune any such setup, even in Russian. Hence, the prompts carried information about how the model could try to work on Russian instances. For this reason, the model seemed to hallucinate the most in Russian as well. Also, the model was fine-tuned on English prompts only, so it seems natural that the outputs were inclined towards English when the prompts made it difficult for the model to stick to the query language.

We also observed an expected positive correlation between the size
