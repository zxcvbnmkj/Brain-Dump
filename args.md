# 命令行传递超参数
## 一、使用方法
（1） 创建 args
```
parser = argparse.ArgumentParser()
parser.add_argument('--new_prompt', action='store_true', help='附加该参数测试新提示词的效果，否则测试旧提示词效果')
parser.add_argument('--json_path', type=str, default=f'{dir_name}/ai_extract_information_university_research.json',
                        help='数据加载路径')
args = parser.parse_args()
```
（2）使用 args 中的参数:`args.new_prompt`

（3）在命令行中调整控制参数的值

对于 Bool 变量，若在文件后面加了参数，就是为 ture；对于字符串等其他参数，则需要在参数名字后面指定值
```
python3 -m extract_information --json_path 'ai_extract_information_internship.json' -- new_prompt
```
## 二、注意事项
关于 bool 类型的变量与其他类型变量不同，不能写作：
```
    parser.add_argument('--new_prompt', type=bool, default=True, help='为True时测试新提示词的效果，False则测试旧提示词效果')
```
这样写的话，命令行传递`python3 -m extract_information -- new_prompt False`的话，`False` 会被识别为`str`类型，而不是`bool`类型，导致代码中的`if`判断失效。不管怎么传递，参数的值依旧是默认值，命令行无法改变。

应该要使用`action='store_ture`或`action='store_false`，前者意味写了该参数，其值就为`True`
