# Jinja2 与 prompty 后缀文件与 PromptFlow
## 一、Jinja2
它使用**模板语法**生成动态的 HTML、XML、或者其他文本文件，Jinja2 属于 Flask 和 FastAPI 等 Web 框架中默认的模板引擎。
### 用法
1. 安装：`pdm add Jinja2`
2. 写好模板
```
# （系统提示词，不会改变）
system:
今天是{{today}}，你需要提取一段文本的关键信息，用 json 格式输出，输出字段如下所示：
{% for dimension in dimensions%}
{{ loop.index }}. `{{ dimension.id }}`: 字段含义为"{{ dimension.desc }}"，字段输出格式："string"{% if dimension.valueRanges%}，示例：{{ dimension.valueRanges }}{% endif%}

{% endfor%}

特别注意，如果候选人的回答中没有某个字段的任何相关信息，则该字段输出"无法评估"。

# （用户提示词）
user:
{% for item in chatList%}
{% if item.question%}
面试官："{{item.question}}"

{% endif%}
{% if item.answer%}
候选人："{{item.answer}}"

{% endif%}
{% endfor%}
```
3. 语法讲解
- `{{ variable }}` 由外界插入，不写死
-  for 循环
```
{% for item in chatList %}
    循环内容
{% endfor %}
```
- for 循环中的 loop
```
{{ loop.index }} ：循环索引，从 1 开始
{{ loop.index0 }} ：循环索引，从 0 开始
```
- if 判断
```
{% if item.question %}
    条件为真时执行
{% endif %}
```
4. 接着调用 jinja2 模版
```
from jinja2 import Environment, FileSystemLoader
# 创建 Jinja2 环境，指定模板文件的路径
env = Environment(loader=FileSystemLoader('存放模版的文件夹路径'))
# 加载模板文件，参数传入 env 文件夹下的一个模版
template = env.get_template('welcome.html')
# 把变量的值传入到 Jinja2 模板中
context = {
    'today': '2025 年 10 月 14 日',
    'dimensions': ['维度1','维度2','维度3','维度4'],
    'chatList': [{'question': question, 'answer': answer}]
}
# 渲染模板并生成最终的 HTML
rendered_html = template.render(context)
print(rendered_html)
```
## 二、prompty 文件
1. prompty 文件混合了 Jinja2、yaml、markdown 写法
```
---
name: 提示词工程-测试项目
model:
  api: chat
  configuration:
    type: openai
    model: gpt-4o
    connection: oneapi_connection
  parameters:
    max_tokens: 800
    temperature: 0.0
    response_format:
      type: json_object
inputs:
  today:
    type: string
  dimensions:
    type: list
  chatList:
    type: list
---
system:
今天是{{today}}，你需要提取一段文本的关键信息，用 json 格式输出，输出字段如下所示：
{% for dimension in dimensions%}
{{ loop.index }}. `{{ dimension.id }}`: 字段含义为"{{ dimension.desc }}"，字段输出格式："string"{% if dimension.valueRanges%}，示例：{{ dimension.valueRanges }}{% endif%}

{% endfor%}

特别注意，如果这段文本中没有某个字段的任何相关信息，则输出"无法评估"。

user:
{% for item in chatList%}
{% if item.question%}
面试官："{{item.question}}"

{% endif%}
{% if item.answer%}
候选人："{{item.answer}}"

{% endif%}
{% endfor%}
```
2. 格式说明
- 上下都使用 `---` 括起来的部分属于 `yaml` 格式的配置信息
- `Jinja2` 中使用到的变量，都需要在 `yaml` 中的 `input` 中定义，`model`部分则是配置了 API 的设置
- 在有变量的地方，一点要使用**中文**的引号括起来，为了防止**提示词攻击**
## 三、微软的提示词工程库- - -PromptFlow
#### 推荐阅读
[官方文档](https://microsoft.github.io/promptflow/how-to-guides/quick-start.html)
### 使用方法
1. 安装： `pdm add promptflow`
2. 创建 prompty 文件
```
---
name: Minimal Chat
model:
  api: chat
  configuration:
    type: openai
    model: gpt-4o
    base_url: "${env:OPENAI_API_BASE}"
    api_key: "${env:OPENAI_API_KEY}"
  parameters:
    temperature: 0.2
    max_tokens: 1024
inputs:
  question:
    type: string
sample:
  question: "What is Prompt flow?"
---

system:
You are a helpful assistant.

user:
{{question}}
```
3. 4. 新建一个 `.env` 文件存放 API 秘钥，该文件不能纳入 git 控制 ！！！
```
OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxx
OPENAI_API_BASE=https://one-api.xxx.com/yyy
```
4. 处理提示词流
```
# 获取当前脚本文件的绝对路径的父目录
BASE_DIR = Path(__file__).absolute().parent
# 装饰器，用于跟踪函数执行和 AI 模型调用
@trace
def chat(question: str = "What's the capital of France?") -> str:
    prompty = Prompty.load(source=BASE_DIR / "chat.prompty")
    # trigger a llm call with the prompty obj
    # 将调用大模型 API
    output = prompty(question=question)
    return output

if __name__ == "__main__":
    # 把 .env 文件中的值加载到环境中
    load_dotenv()
    # 必须先执行上面的加载代码，这两行才可能获取到值，否则只返回 None
    openai_api_key = os.getenv("OPENAI_API_KEY")
    print("OpenAI API Key: ", openai_api_key)
    base = os.getenv("OPENAI_API_BASE")
    print("OPEN_API_BASE: ", base)

    from promptflow.tracing import start_trace
    start_trace()
    result = chat("法国的首都是哪里？")
    print(result)
```
该文件的输出如下，不包括可视化网站的链接
```
Prompt flow service has started...
法国的首都是**巴黎**（Paris）。巴黎位于法国的北部，是该国的政治、经济、文化和交通中心，同时也是世界著名的旅游城市，以其丰富的历史、艺术和建筑闻名，例如埃菲尔铁塔、卢浮宫、巴黎圣母院等地标性建筑。
```
