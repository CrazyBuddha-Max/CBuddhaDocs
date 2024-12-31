---
comments: true
---
(原创)基于截断策略的机器阅读理解任务实现
***
## 机器阅读理解任务简介
### 什么是机器阅读理解任务
- 机器阅读理解（Machine Reading Comprehension, MRC）是一项自然语言处理（NLP）领域的重要任务，旨在使机器能够像人一样理解文本，并基于所阅读的内容回答问题。MRC的基本流程包括让机器阅读一段文本或文章，然后对该文本中的内容提出相关问题，机器需要根据文本内容回答这些问题。有几种不同类型的机器阅读理解任务，具体如下：

- 1.文本片段匹配 ：系统从文章中找到一个明确的片段作为问题的答案。这通常是“Span-based”的问题，比如在给定的段落中找到答案所在的具体位置。
- 2.多项选择问题 ：在给定的文本和多个选项的情况下，机器需要从这些选项中选择正确答案。这通常用于场景比较明确的问题。
- 3.自由生成答案 ：机器从理解文本的基础上生成答案，不仅限于原文中的片段。这类任务对模型的语言生成以及理解能力都有很高的要求。
- 4.真假判断 ：系统需要判断根据文本给出的陈述是否是正确的。
典型的MRC任务会涉及知识推理、语义关系理解等多个挑战，因此对于语言理解的深度要求较高。一些经典的MRC数据集测试集包括SQuAD、CoQA以及CMRC等。

### 机器阅读理解数据集格式
```json
# mcmrc2018数据集
{
  "id": "cmrc2018_train-00001",
  "text": "近年来，中国的科技发展迅速，特别是在人工智能领域。例如，深度求索公司开发的AI模型在图像识别、语音识别等方面表现出色。",
  "question": "深度求索是哪家公司？",
  "answers": ["深度求索公司"]
}
```

## 评估指标
- 精准匹配度(EM):计算预测结果与标准答案是否完全匹配。
- 模糊匹配度(F1):计算预测结果与标准答案之间字级别的匹配程度。
  
## 数据预处理
### 数据处理格式

|  [CLS]  |  Question  |  [SEP]  |  Context  |  SEP  |
|:--:|:--:|:--:|:--:|:--:|

### 如何确认定位答案位置
- 步骤一：得到tokens的start_positions和end_positions.(注意：这里的tokens不仅仅指一个单词，它也可以指一句话，特别是在中英混合的数据集中)
- 步骤二：得到offset_mapping。(类似于word ids，但在其基础上加入了对应到原始文本上的映射，便于后期定位到答案在原始文本中的位置)

### Context过长的问题
- 方案一：滑动窗口，会丢失部分上下文，比直接截断损失小。
- 方案二：直接截断，相比其他方案比较简单易用，但是会损失所有超过长度的数据，无法定位答案位置。

## 模型结构
- ModelForQuestionAnswering

## 实现代码
### 方案一：
```python

from datasets import load_dataset, DatasetDict
from transformers import AutoTokenizer, AutoModelForQuestionAnswering, TrainingArguments, Trainer, DefaultDataCollator

datasets = DatasetDict.load_from_disk("mrc_data")
datasets

datasets["train"][0]

tokenizer = AutoTokenizer.from_pretrained("hfl/chinese-macbert-base")
tokenizer

sample_dataset = datasets["train"].select(range(10))

tokenized_examples = tokenizer(text=sample_dataset["question"],
                               text_pair=sample_dataset["context"],
                               return_offsets_mapping=True,
                               return_overflowing_tokens=True,
                               stride=128,
                               max_length=384, truncation="only_second", padding="max_length")
tokenized_examples.keys()

tokenized_examples["overflow_to_sample_mapping"], len(tokenized_examples["overflow_to_sample_mapping"])

for sen in tokenizer.batch_decode(tokenized_examples["input_ids"][:3]):
    print(sen)

print(tokenized_examples["offset_mapping"][:3])

print(tokenized_examples["offset_mapping"][0], len(tokenized_examples["offset_mapping"][0]))


sample_mapping = tokenized_examples.pop("overflow_to_sample_mapping")

for idx, _ in enumerate(sample_mapping):
    answer = sample_dataset["answers"][sample_mapping[idx]]
    start_char = answer["answer_start"][0]
    end_char = start_char + len(answer["text"][0])

    context_start = tokenized_examples.sequence_ids(idx).index(1)
    context_end = tokenized_examples.sequence_ids(idx).index(None, context_start) - 1

    offset = tokenized_examples.get("offset_mapping")[idx]
    example_ids = []

    if offset[context_end][1] < start_char or offset[context_start][0] > end_char:
        start_token_pos = 0
        end_token_pos = 0
    else:
        token_id = context_start
        while token_id <= context_end and offset[token_id][0] < start_char:
            token_id += 1
        start_token_pos = token_id
        token_id = context_end
        while token_id >= context_start and offset[token_id][1] > end_char:
            token_id -=1
        end_token_pos = token_id
        example_ids.append([sample_mapping[idx]])
        
    print(answer, start_char, end_char, context_start, context_end, start_token_pos, end_token_pos)
    print("token answer decode:", tokenizer.decode(tokenized_examples["input_ids"][idx][start_token_pos: end_token_pos + 1]))

def process_func(examples):
    tokenized_examples = tokenizer(text=examples["question"],
                               text_pair=examples["context"],
                               return_offsets_mapping=True,
                               return_overflowing_tokens=True,
                               stride=128,
                               max_length=384, truncation="only_second", padding="max_length")
    sample_mapping = tokenized_examples.pop("overflow_to_sample_mapping")
    start_positions = []
    end_positions = []
    example_ids = []
    for idx, _ in enumerate(sample_mapping):
        answer = examples["answers"][sample_mapping[idx]]
        start_char = answer["answer_start"][0]
        end_char = start_char + len(answer["text"][0])
    
        context_start = tokenized_examples.sequence_ids(idx).index(1)
        context_end = tokenized_examples.sequence_ids(idx).index(None, context_start) - 1
        offset = tokenized_examples.get("offset_mapping")[idx]
      
        if offset[context_end][1] < start_char or offset[context_start][0] > end_char:
            start_token_pos = 0
            end_token_pos = 0
        else:
            token_id = context_start
            while token_id <= context_end and offset[token_id][0] < start_char:
                token_id += 1
            start_token_pos = token_id
            token_id = context_end
            while token_id >= context_start and offset[token_id][1] > end_char:
                token_id -=1
            end_token_pos = token_id
        start_positions.append(start_token_pos)
        end_positions.append(end_token_pos)
        example_ids.append(examples["id"][sample_mapping[idx]])
        tokenized_examples["offset_mapping"][idx] = [
            (o if tokenized_examples.sequence_ids(idx)[k] == 1 else None)
            for k, o in enumerate(tokenized_examples["offset_mapping"][idx])
        ]

    
    tokenized_examples["example_ids"] = example_ids
    tokenized_examples["start_positions"] = start_positions
    tokenized_examples["end_positions"] = end_positions
    return tokenized_examples


tokenied_datasets = datasets.map(process_func, batched=True, remove_columns=datasets["train"].column_names)
tokenied_datasets


print(tokenied_datasets["train"]["offset_mapping"][1])


tokenied_datasets["train"]["example_ids"][:10]


import collections

example_to_feature = collections.defaultdict(list)
for idx, example_id in enumerate(tokenied_datasets["train"]["example_ids"][:10]):
    example_to_feature[example_id].append(idx)
example_to_feature



import numpy as np
import collections

def get_result(start_logits, end_logits, exmaples, features):

    predictions = {}
    references = {}

    
    example_to_feature = collections.defaultdict(list)
    for idx, example_id in enumerate(features["example_ids"]):
        example_to_feature[example_id].append(idx)

  
    n_best = 20

    max_answer_length = 30

    for example in exmaples:
        example_id = example["id"]
        context = example["context"]
        answers = []
        for feature_idx in example_to_feature[example_id]:
            start_logit = start_logits[feature_idx]
            end_logit = end_logits[feature_idx]
            offset = features[feature_idx]["offset_mapping"]
            start_indexes = np.argsort(start_logit)[::-1][:n_best].tolist()
            end_indexes = np.argsort(end_logit)[::-1][:n_best].tolist()
            for start_index in start_indexes:
                for end_index in end_indexes:
                    if offset[start_index] is None or offset[end_index] is None:
                        continue
                    if end_index < start_index or end_index - start_index + 1 > max_answer_length:
                        continue
                    answers.append({
                        "text": context[offset[start_index][0]: offset[end_index][1]],
                        "score": start_logit[start_index] + end_logit[end_index]
                    })
        if len(answers) > 0:
            best_answer = max(answers, key=lambda x: x["score"])
            predictions[example_id] = best_answer["text"]
        else:
            predictions[example_id] = ""
        references[example_id] = example["answers"]["text"]

    return predictions, references




from cmrc_eval import evaluate_cmrc

def metirc(pred):
    start_logits, end_logits = pred[0]
    if start_logits.shape[0] == len(tokenied_datasets["validation"]):
        p, r = get_result(start_logits, end_logits, datasets["validation"], tokenied_datasets["validation"])
    else:
        p, r = get_result(start_logits, end_logits, datasets["test"], tokenied_datasets["test"])
    return evaluate_cmrc(p, r)




model = AutoModelForQuestionAnswering.from_pretrained("hfl/chinese-macbert-base")

args = TrainingArguments(
    output_dir="models_for_qa",
    per_device_train_batch_size=32,
    per_device_eval_batch_size=32,
    eval_strategy="steps",
    eval_steps=200,
    save_strategy="epoch",
    logging_steps=50,
    num_train_epochs=1
)

trainer = Trainer(
    model=model,
    args=args,
    tokenizer=tokenizer,
    train_dataset=tokenied_datasets["train"],
    eval_dataset=tokenied_datasets["validation"],
    data_collator=DefaultDataCollator(),
    compute_metrics=metirc
)

trainer.train()

model.save_pretrained("save-model-solution2")

```
### 方案二：
```python

from datasets import load_dataset, DatasetDict
from transformers import AutoTokenizer, AutoModelForQuestionAnswering, TrainingArguments, Trainer, DefaultDataCollator

datasets = DatasetDict.load_from_disk("mrc_data")
datasets

datasets["train"][0]

tokenizer = AutoTokenizer.from_pretrained("hfl/chinese-macbert-base")
tokenizer

sample_dataset = datasets["train"].select(range(10))

tokenized_examples = tokenizer(text=sample_dataset["question"],
                               text_pair=sample_dataset["context"],
                               return_offsets_mapping=True,
                               max_length=512, truncation="only_second", padding="max_length")
tokenized_examples.keys()


print(tokenized_examples["offset_mapping"][0], len(tokenized_examples["offset_mapping"][0]))

offset_mapping = tokenized_examples.pop("offset_mapping")

for idx, offset in enumerate(offset_mapping):
    answer = sample_dataset[idx]["answers"]
    start_char = answer["answer_start"][1]
    end_char = start_char + len(answer["text"][0])
    context_start = tokenized_examples.sequence_ids(idx).index(1)
    context_end = tokenized_examples.sequence_ids(idx).index(None, context_start) - 1

    if offset[context_end][1] < start_char or offset[context_start][0] > end_char:
        start_token_pos = 0
        end_token_pos = 0
    else:
        token_id = context_start
        while token_id <= context_end and offset[token_id][0] < start_char:
            token_id += 1
        start_token_pos = token_id
        token_id = context_end
        while token_id >= context_start and offset[token_id][1] > end_char:
            token_id -=1
        end_token_pos = token_id
        
    print(answer, start_char, end_char, context_start, context_end, start_token_pos, end_token_pos)
    print("token answer decode:", tokenizer.decode(tokenized_examples["input_ids"][idx][start_token_pos: end_token_pos + 1]))

def process_func(examples):
    tokenized_examples = tokenizer(text=examples["question"],
                               text_pair=examples["context"],
                               return_offsets_mapping=True,
                               max_length=384, truncation="only_second", padding="max_length")
    offset_mapping = tokenized_examples.pop("offset_mapping")
    start_positions = []
    end_positions = []
    for idx, offset in enumerate(offset_mapping):
        answer = examples["answers"][idx]
        start_char = answer["answer_start"][0]
        end_char = start_char + len(answer["text"][0])
        context_start = tokenized_examples.sequence_ids(idx).index(1)
        context_end = tokenized_examples.sequence_ids(idx).index(None, context_start) - 1
        if offset[context_end][1] < start_char or offset[context_start][0] > end_char:
            start_token_pos = 0
            end_token_pos = 0
        else:
            token_id = context_start
            while token_id <= context_end and offset[token_id][0] < start_char:
                token_id += 1
            start_token_pos = token_id
            token_id = context_end
            while token_id >= context_start and offset[token_id][1] > end_char:
                token_id -=1
            end_token_pos = token_id
        start_positions.append(start_token_pos)
        end_positions.append(end_token_pos)
    
    tokenized_examples["start_positions"] = start_positions
    tokenized_examples["end_positions"] = end_positions
    return tokenized_examples

tokenied_datasets = datasets.map(process_func, batched=True, remove_columns=datasets["train"].column_names)
tokenied_datasets

model = AutoModelForQuestionAnswering.from_pretrained("hfl/chinese-macbert-base")


args = TrainingArguments(
    output_dir="models_for_qa",
    per_device_train_batch_size=32,
    per_device_eval_batch_size=32,
    eval_strategy="epoch",
    save_strategy="epoch",
    logging_steps=50,
    num_train_epochs=3
)

trainer = Trainer(
    model=model,
    args=args,
    tokenizer=tokenizer,
    train_dataset=tokenied_datasets["train"],
    eval_dataset=tokenied_datasets["validation"],
    data_collator=DefaultDataCollator()
)

trainer.train()

model.save_pretrained("saved-model")


```