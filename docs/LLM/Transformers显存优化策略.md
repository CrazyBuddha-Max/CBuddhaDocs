---
comments: true
---
(原创)Transformers显存优化简易策略(本教程目标：4G显存也能跑BERT-Large)
***
## 显存占用分析(主要的)
### 模型权重
- 4bytes*模型参数量
### 优化器状态
- 8bytes*模型参数量(只适用于常用的AdamW优化器)
### 梯度
- 4bytes*模型参数量
### 前向激活值
- 取决于序列长度、隐层维度、Batch大小等多个因素

## 显存优化策略
### 以chinese-macbert-large模型为例构建优化案例
| 优化策略   |  优化对象  |  显存占用  |  训练时间  |
|:--:|:--:|:--:|:--:|
|  Baseline(BS 32,MaxLength 128)  |   -  |  15.1G  |  44.1s  |
| +Gradient Accumulation(BS 1,GA 32)   | 前向激活值   |  7.5G  |  122s  |
|  +Gradient Checkpoints(BS 1,GA 32)  |  前向激活值  |  7.3G  |  461s  |
|  +Adafactor Optiomizer(BS 1,GA 32)  | 优化器状态   |  5.5G  |455s   |
|  +Freeze Model(BS 1,GA 32)  |  前向激活值/梯度  |  3.5G  |  119s  |
|  +Data Length(BS 1,GA 32,MaxLength 64)  |  前向激活值  | 3.4G  |  110s  |

### 推荐的参数设置：小白劝退(需要具备一定的python和transformers库的深度理解水平)
```python
train_args = TrainingArguments(output_dir="./checkpoints",      # 输出文件夹
                               per_device_train_batch_size=1,  # 训练时的batch_size
                               gradient_accumulation_steps=32, #梯度累加
                               gradient_checkpointing=True,    #使用Gradient Checkpoints
                               per_device_eval_batch_size=1,  # 验证时的batch_size
                               optim="adafactor",               #使用adafactor优化器
                               num_train_epochs=1,
                               logging_steps=10,                # log 打印的频率
                               evaluation_strategy="epoch",     # 评估策略
                               save_strategy="epoch",           # 保存策略
                               save_total_limit=3,              # 输出文件夹中模型最大保存数
                               learning_rate=2e-5,              # 学习率
                               weight_decay=0.01,               # weight_decay
                               metric_for_best_model="f1",      # 设定评估指标
                               load_best_model_at_end=True)     # 训练完成后加载最优模型
train_args
```
```python
def process_function(examples):
    tokenized_examples = tokenizer(examples["review"], max_length=32, truncation=True, padding="max_length")
    tokenized_examples["labels"] = examples["label"]
    return tokenized_examples

tokenized_datasets = datasets.map(process_function, batched=True, remove_columns=datasets["train"].column_names)
```