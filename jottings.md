# 杂记
1. 训练样本数十万条的时候，一定要每 n 个批次打印一次信息，而不能只在每一个 epoch 后打印，因为训练一轮可能需要仅一小时，这个过程中，控制台没有一点反馈。
2. pycharm 中打开指定文件，然后菜单栏操作 `view --> Active Editor -->Soft-Wrap` 即可实现文件的自动换行，尤为适合 `json` 文件
3. 模块化执行方法，使用参数 `-m` ，文件路径不使用 `/`，而是用 `.` 代替，最后文件名不加后缀 `.py`
```
python -m gpt.ai_extract.ai_extract_information --json_path ai_extract_information_large_ai_models.json --max_workers 1 --new_prompt
```
4. 如果遇到不符合预期的情况，python 里面完全可以抛出异常，而不是仅仅只打印问题
```
data_files = glob.glob(osp.join(f"{dir_name}/data", "*"))
if not data_files:
    raise FileNotFoundError(f"data 文件夹下没有文件")
```
其中，`FileNotFoundError` 是预定义的异常类型，但它不需要 import ，遇到异常时会得到：
```
Traceback (most recent call last):
  File "/Users/nowcoder/BERT_Finetuning/main.py", line 181, in <module>
    raise FileNotFoundError(f"data 文件夹下没有文件")
FileNotFoundError: data 文件夹下没有文件
```
## 提示词工程
- 如何写一本电子书，解决这种复杂问题- - -**搭建工作流**
  - **工作流**和**思维链**的区别：工作流是多次调用大模型，思维链是一次
  - 提示词中的 system 和 user 部分的区别。
    - system prompt 是可选的，有的模型不需要这一部分
    - 有些大模型有缓存机制的话，会把 system 放到缓存中，因此不变的文本最好放到 system 中
    - system 中的内容可以看做是 STAR 的情景



## 实习总结
- 认识到了提示词工程的重要性，在要点提取任务中感受最深，来之前完全忽略了
- 在工作和生活中挖掘需求与痛点，尝试通过技术解决它
-

#### 工具
- 知源笔记：已经放弃了使用 word 写文档和记录
  - markdown 的痛点：（1）图片不在文档内部而是存放在单独文件夹中，发送不方便（2）手机打不开
  - git 解决了前者问题，知源解决了后者
- git
-



雷点：
1. 文档，尤其是分享，用 AI 去写。必须要保持真情实感，热忱，自己的体会
