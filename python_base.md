# python基础
## 数据转换
1. np 与 df
```
array_2d = np.array([
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
])
#注意，D和F需要大写
df = pd.DataFrame(array_2d)
```

```
df = pd.DataFrame({
    'A': [1, 2, 3],
    'B': [4, 5, 6],
    'C': [7, 8, 9]
})
# 方法1: 使用 .values 属性
numpy_array1 = df.values
# 方法2
numpy_array2 = df.to_numpy()
# 方法3: 使用 numpy.array() 函数
numpy_array3 = np.array(df)
```
## Dataframe

## numpy
创建矩阵（多维数组）：
```
mat2=np.zeros((M,N),dtype=int)
```

## 读取
json.loads(str)负责加载一个字符串，并将其解析为json类型。

json.load()不能读取字符串！！！

它仅能从文件中读取：
```
with open('data.json', 'r', encoding='utf-8') as f:
    data = json.load(f)
    print(data)
```

## 其他
range不能写作：`for i in range(2,len(data),step=2):`
```
range(stop)                    # 0 到 stop-1，步长1
range(start, stop)            # start 到 stop-1，步长1
range(start, stop, step)      # start 到 stop-1，步长step
```
`for i in range(len(my_list))` 循环次数一定等于列表长度
