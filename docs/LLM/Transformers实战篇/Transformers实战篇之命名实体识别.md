---
comments: true
---
(原创)命名实体识别基于Transformers的解决方案
***
## 命名实体识别简介
### 什么是命名实体识别任务
- 指的是识别文本中有特定意义的实体，主要包括人名、地名、机构名、专有名词等。通常包括两部分：1.实体边界识别；2.确定实体类别(人名、地名、机构名或者其他)。

例句：*张敏在香港呆了8年*。

|  实体类别  |  实体  |
|:--:|:--:|
|  人名  |  张敏  |
|   地名 |  香港  |

### 数据标注体系案例
- IOB1、IOB2、IOE1、IOE2、IOBES、BLILOU
  
### 常用的数据标注体系之IOB2
- I表示实体内部，O表示实体外部、B表示实体开始
- B/I-XXX，XXX表示具体的类别
### 常用的数据标注体系之IOBES
- I表示实体内部，O表示实体外部、B表示实体开始，S表示一个词单独形成一个命名实体、e表示实体结束
- M可以替换I
### 案例(IOB2)
> 按照IOB2标注规则进行标注后，结果如下：
张三（B-PER）：人名的开始
三（I-PER）：人名的内部部分
于（O）：不在实体中
2019年（O）：不在实体中
在（O）：不在实体中
清华大学（B-LOC）：地名的开始
清华（I-LOC）：地名的内部部分
大学（I-LOC）：地名的内部部分
获得（O）：不在实体中
了（O）：不在实体中
计算机科学与技术（B-MISC）：其他未分类实体的开始
计算机（I-MISC）：其他未分类实体的内部部分
科学与技术（I-MISC）：其他未分类实体的内部部分
博士学位（O）：不在实体中  

### 评估指标
- Precision
- Recall
- F1
案例：
> 预测实体个数：predict_num=2
> 真实实体个数：gold_num=3
> 预测正确的实体个数：correct_num=1
> 
则评估指标分别为：

> <math xmlns="http://www.w3.org/1998/Math/MathML" display="block">  <mi>F</mi>  <mn>1</mn>  <mo>=</mo>  <mfrac>    <mrow>      <mn>2</mn>      <mo>&#x2217;</mo>      <mfrac>        <mn>1</mn>        <mn>2</mn>      </mfrac>      <mo>&#x2217;</mo>      <mfrac>        <mn>1</mn>        <mn>3</mn>      </mfrac>    </mrow>    <mrow>      <mfrac>        <mn>1</mn>        <mn>2</mn>      </mfrac>      <mo>+</mo>      <mfrac>        <mn>1</mn>        <mn>3</mn>      </mfrac>    </mrow>  </mfrac>  <mo>,</mo>  <mi>P</mi>  <mo>=</mo>  <mfrac>    <mn>1</mn>    <mn>2</mn>  </mfrac>  <mo>,</mo>  <mi>R</mi>  <mo>=</mo>  <mfrac>    <mn>1</mn>    <mn>3</mn>  </mfrac></math>

## 基于Transformers的解决方案
### 模型结构
- ModelForTokenClassification
### 评估函数
- seqeval

### 具体实现代码
```python
# %% [markdown]
# # 基于Transformers的命名实体识别

# %% [markdown]
# ## Step1 导入相关包

# %%
import evaluate
from datasets import load_dataset
from transformers import AutoTokenizer, AutoModelForTokenClassification, TrainingArguments, Trainer, DataCollatorForTokenClassification

# %% [markdown]
# ## Step2 加载数据集

# %%
# 如果可以联网，直接使用load_dataset进行加载
#ner_datasets = load_dataset("peoples_daily_ner", cache_dir="./data")
# 如果无法联网，则使用下面的方式加载数据集
from datasets import DatasetDict
ner_datasets = DatasetDict.load_from_disk("ner_data")
ner_datasets

# %%
ner_datasets["train"][0]

# %%
ner_datasets["train"].features

# %%
label_list = ner_datasets["train"].features["ner_tags"].feature.names
label_list

# %% [markdown]
# ## Step3 数据集预处理

# %%
# 加载模型
tokenizer = AutoTokenizer.from_pretrained("hfl/chinese-macbert-base")

# %%
tokenizer(ner_datasets["train"][0]["tokens"], is_split_into_words=True)   # 对于已经做好tokenize的数据，要指定is_split_into_words参数为True，101和102只会出现一次

# %%
res = tokenizer("interesting word")
res

# %%
res.word_ids()

# %%
# 借助word_ids 实现标签映射
def process_function(examples):
    tokenized_exmaples = tokenizer(examples["tokens"], max_length=128, truncation=True, is_split_into_words=True)
    labels = []
    for i, label in enumerate(examples["ner_tags"]):
        word_ids = tokenized_exmaples.word_ids(batch_index=i)
        label_ids = []
        for word_id in word_ids:
            if word_id is None:
                label_ids.append(-100)
            else:
                label_ids.append(label[word_id])
        labels.append(label_ids)
    tokenized_exmaples["labels"] = labels
    return tokenized_exmaples

# %%
tokenized_datasets = ner_datasets.map(process_function, batched=True)
tokenized_datasets

# %%
print(tokenized_datasets["train"][0])

# %% [markdown]
# ## Step4 创建模型

# %%
# 对于所有的非二分类任务，切记要指定num_labels，否则就会device错误
model = AutoModelForTokenClassification.from_pretrained("hfl/chinese-macbert-base", num_labels=len(label_list))

# %%
model.config.num_labels

# %% [markdown]
# ## Step5 创建评估函数

# %%
# 这里方便大家加载，替换成了本地的加载方式，无需额外下载
# seqeval = evaluate.load("seqeval")
seqeval = evaluate.load("seqeval_metric.py")
seqeval


# %%
import numpy as np

def eval_metric(pred):
    predictions, labels = pred
    predictions = np.argmax(predictions, axis=-1)

    # 将id转换为原始的字符串类型的标签
    true_predictions = [
        [label_list[p] for p, l in zip(prediction, label) if l != -100]
        for prediction, label in zip(predictions, labels) 
    ]

    true_labels = [
        [label_list[l] for p, l in zip(prediction, label) if l != -100]
        for prediction, label in zip(predictions, labels) 
    ]

    result = seqeval.compute(predictions=true_predictions, references=true_labels, mode="strict", scheme="IOB2")

    return {
        "f1": result["overall_f1"]
    }
    

# %% [markdown]
# ## Step6 配置训练参数

# %%
args = TrainingArguments(
    output_dir="models_for_ner",
    per_device_train_batch_size=64,
    per_device_eval_batch_size=128,
    evaluation_strategy="epoch",
    save_strategy="epoch",
    metric_for_best_model="f1",
    load_best_model_at_end=True,
    logging_steps=50,
    num_train_epochs=3
)

# %% [markdown]
# ## Step7 创建训练器

# %%
trainer = Trainer(
    model=model,
    args=args,
    tokenizer=tokenizer,
    train_dataset=tokenized_datasets["train"],
    eval_dataset=tokenized_datasets["validation"],
    compute_metrics=eval_metric,
    data_collator=DataCollatorForTokenClassification(tokenizer=tokenizer)
)

# %% [markdown]
# ## Step8 模型训练

# %%
trainer.train()

# %%
trainer.evaluate(eval_dataset=tokenized_datasets["test"])

# %% [markdown]
# ## Step9 模型预测

# %%
from transformers import pipeline

# %%
model.config

# %%
# 使用pipeline进行推理，要指定id2label
model.config.id2label = {idx: label for idx, label in enumerate(label_list)}
model.config

# %%
# 如果模型是基于GPU训练的，那么推理时要指定device
# 对于NER任务，可以指定aggregation_strategy为simple，得到具体的实体的结果，而不是token的结果
ner_pipe = pipeline("token-classification", model=model, tokenizer=tokenizer, device=0, aggregation_strategy="simple")

# %%
# 诡异bug，“小”开头的人名无法识别“小”这个字，不知道哪里出问题了
# res = ner_pipe("小张在上海上班")
res = ner_pipe("晓李在上海上班")
res

# %%
# 根据start和end取实际的结果
ner_result = {}
x = "晓李在北京上班"
for r in res:
    if r["entity_group"] not in ner_result:
        ner_result[r["entity_group"]] = []
    ner_result[r["entity_group"]].append(x[r["start"]: r["end"]])

ner_result

# %%




```