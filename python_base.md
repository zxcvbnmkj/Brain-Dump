# python基础

1. range不能写作：`for i in range(2,len(data),step=2):`
```
range(stop)                    # 0 到 stop-1，步长1
range(start, stop)            # start 到 stop-1，步长1
range(start, stop, step)      # start 到 stop-1，步长step
```
2. 创建矩阵（多维数组）：
```
mat2=np.zeros((M,N),dtype=int)
```
3. json.loads(str)负责加载一个字符串，并将其解析为json类型。

json.load()不能读取字符串！！！

它仅能从文件中读取：
```
with open('data.json', 'r', encoding='utf-8') as f:
    data = json.load(f)
    print(data)
```
