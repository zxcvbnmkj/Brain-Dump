# 杂记
1. 训练样本数十万条的时候，一定要每 n 个批次打印一次信息，而不能只在每一个 epoch 后打印，因为训练一轮可能需要仅一小时，这个过程中，控制台没有一点反馈。
2. pycharm 中打开指定文件，然后菜单栏操作 `view --> Active Editor -->Soft-Wrap` 即可实现文件的自动换行，尤为适合 `json` 文件
3. 模块化执行方法，使用参数 `-m` ，文件路径不使用 `/`，而是用 `.` 代替，最后文件名不加后缀 `.py`
```
python -m gpt.ai_extract.ai_extract_information --json_path ai_extract_information_large_ai_models.json --max_workers 1 --new_prompt
```
