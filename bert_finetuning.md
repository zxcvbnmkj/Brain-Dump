# 微调一个BERT

## 一、分词器
- [CLS]、[SEP]对应的编号分别是101、102，BERT 分词器会自动在输入句子的开头加上 [CLS]，末尾加上 [SEP]。
- `input_ids` 是词在`vocab.txt`中对应的编号
- `attention_mask` 是注意力掩码，用来标注哪些位置的词是真实的（1），哪里是填充区域（0）
- `token_type_ids` 是句子类型ID，用于句子对任务中区别两个句子，当只有一种类型是，默认都为0。
- [CLS] 对 BERT 模型非常重要，BERT 选取了一个没有语义的 token 来存储整个句子的综合信息，BERT 分类就是在 [CLS] 向量的基础上做的线性层降维
- [SEP] 一般用于分割两种句子，比如 question 和 answer，让 BERT 更好理解文档结构。而不是只有判断两句话关系的任务中才可以使用，普通的基于双句的分类模型也可以用。
- chinese-bert-wwm 这个模型分词得到的 token 几乎都是单字的
```
文本: [CLS] 今天天气很好 [SEP] 适合出去玩 [SEP]
token_type_ids: 0   0  0  0  0   0   1  1  1  1   1
```
手动分步过程（理解原理）：
```
text = "我喜欢自然语言处理"

# 步骤1: 分词
tokens = tokenizer.tokenize(text)
# 结果: ['我', '喜欢', '自然', '语言', '处理']

# 步骤2: 添加特殊标记
tokens = ['[CLS]'] + tokens + ['[SEP]']
# 结果: ['[CLS]', '我', '喜欢', '自然', '语言', '处理', '[SEP]']

# 步骤3: 转换为ids
input_ids = tokenizer.convert_tokens_to_ids(tokens)
# 结果: [101, 2769, 4263, 3315, 3221, 686, 102]
```
代码中一般一步到位：
```
text = "我喜欢自然语言处理"
tokenizer=BertTokenizer.from_pretrained('chinese-bert-wwm')
encoded_inputs = tokenizer(
    text,
    padding=True,
    truncation=True, #当文本超出最大长度时，自动截断
    max_length=512,
    return_tensors='pt'  # 返回PyTorch张量
)
print(encoded_inputs)
-----------------------------输出结果----------------------------
{'input_ids': tensor([[ 101, 2769, 1599, 3614, 5632, 4197, 6427, 6241, 1905, 4415,  102]]), 'token_type_ids': tensor([[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]]), 'attention_mask': tensor([[1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1]])}

```
传入的参数可以是一个字符串的列表：
```
encoded_inputs = tokenizer(
    ["我喜欢自然语言处理","我是翠花"],
    padding=True,
    truncation=True,
    max_length=512,
    return_tensors='pt'  # 返回PyTorch张量
)
-----------------------------输出结果----------------------------
{'input_ids': tensor([[ 101, 2769, 1599, 3614, 5632, 4197, 6427, 6241, 1905, 4415,  102], [ 101, 2769, 3221, 5428, 5709,  102,    0,    0,    0,    0,    0]]), 'token_type_ids': tensor([[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],  [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]]), 'attention_mask': tensor([[1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1], [1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0]])}
```
## 二、微调步骤

### （一）处理数据

如果是 `json` 或 `txt` 格式的数据，建议将其转换为 `dataframe` 类型，其中包含 `text` 与 `label` 两列。

### （二）分词器处理数据
```
sentences = df['text'].tolist()
labels = df['label'].tolist()

tokenizer = BertTokenizer.from_pretrained('chinese-bert-wwm')
encoded_inputs = tokenizer(
    sentences,
    padding=True,
    truncation=True,
    max_length=512,
    return_tensors='pt'  # 返回PyTorch张量
)

train_inputs, val_inputs, train_masks, val_masks, train_labels, val_labels = train_val_data = train_test_split(
    encoded_inputs['input_ids'],
    encoded_inputs['attention_mask'],
    labels,
    test_size=0.01,
    random_state=42
)

train_data = {
    'input_ids': train_inputs.clone().detach(),
    'attention_mask': train_masks.clone().detach(),
    'labels': torch.tensor(train_labels)
}
val_data = {
    'input_ids': val_inputs.clone().detach(),
    'attention_mask': val_masks.clone().detach(),
    'labels': torch.tensor(val_labels)
}

train_sample = TensorDataset(train_data['input_ids'], train_data['attention_mask'], train_data['labels'])
train_sampler = RandomSampler(train_sample)
train_dataloader = DataLoader(train_sample, sampler=train_sampler, batch_size=args.batch_size)

val_sample = TensorDataset(val_data['input_ids'], val_data['attention_mask'], val_data['labels'])
val_sampler = RandomSampler(val_sample)
val_dataloader = DataLoader(val_sample, sampler=val_sampler, batch_size=args.batch_size)
```
### （三）加载模型
```
# 若用于分类任务
model = BertForSequenceClassification.from_pretrained("chinese-bert-wwm", num_labels=2).to(device)

# 兼容分布式训练
if torch.cuda.device_count() > 1:
    print(f"有 {torch.cuda.device_count()} 个GPU")
    model = nn.DataParallel(model)
```
### （四）设置优化器

```
param_optimizer = list(model.named_parameters())
no_decay = ['bias', 'LayerNorm.weight']
optimizer_grouped_parametes = [
    {'params': [p for n, p in param_optimizer if not any(nd in n for nd in no_decay)],
     'weight_decay_rate': 0.1},
    {"params": [p for n, p in param_optimizer if any(nd in n for nd in no_decay)],
     'weight_decay_rate': 0.0}
]
optimizer = AdamW(optimizer_grouped_parametes, lr=2e-5)
```
### （五）微调过程
```
for epoch_i in trange(epochs, desc="Epoch"):
    # ========== 训练阶段 ==========
    total_train_loss, total_train_acc, total_train_p, total_train_r, total_train_f1 = 0, 0, 0, 0, 0
    for batch in train_dataloader:
        b_input_ids, b_input_mask, b_labels = tuple(t.to(device) for t in batch)
        # 1. 梯度为0
        model.zero_grad()
        # 2. 前向传播
        outputs = model(b_input_ids,
                        attention_mask=b_input_mask,
                        labels=b_labels)
        # 3. 计算损失
        loss = outputs.loss.mean()
        # 4. 反向传播
        loss.backward()
        # 5. 梯度裁剪（为防止梯度爆炸，也可省略）
        torch.nn.utils.clip_grad_norm_(model.parameters(), 1.0)
        # 6. 优化器进行参数更新
        optimizer.step()
        # 7. 学习率调度器更新（可省略）
        #scheduler.step()

        #计算评估指标
        logits = outputs.logits.detach().cpu().numpy()
        label_ids = b_labels.to('cpu').numpy()
        acc, p, r, f1 = calculate_metrics(logits, label_ids)
```
### （六）模型存储与加载
比起 `torch.save()` 下列方法更适合存储来自 huggingface 的模型
```
if hasattr(model, 'module'):
    # 如果模型被包装在 DataParallel 中
    model.module.save_pretrained(f'{dir_name}/bert_classifier')
else:
    model.save_pretrained(f'{dir_name}/bert_classifier')
```
```
# 模型加载
model = BertForSequenceClassification.from_pretrained(f'{dir_name}/bert_classifier')
```
## 三、BERT 的特点
1. `BERT` 可处理的最大序列长度是 `512`，在分词阶段，分词器的参数 max_lenth 的值可以大于 512，在这一步不出错，但是在 BERT 前向传播环节，如果 token 长度大于 512 会报错。
2. 可以直接使用 transformers 库提供的 BertForSequenceClassification 类来微调，此时 指定了`label`会自动返回损失，不需要外部创建损失函数的实例。除此之外，也可以自己定义 BERT 微调模型的结果（即在 BERT 模型之后接一个 MLP等层），然后还需要自己定义损失函数。
## 四、进阶
BERT 的分词器可以接受两个文本参数，作为双句任务的子句1、2，还可以通过参数指定截断子句 1还是子句2
