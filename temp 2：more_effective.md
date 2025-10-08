# python 进阶
1. @dataclass 类型的类
   它将自动生成 `__repr__` 方法。
   直接打印类名，得到的只是类地址。如： `<__main__.Person object at 0x7f8c12345678>`
   所以可在类中定义 `__repr__` 方法：

```
 def __repr__(self):
        return f"Person(name='{self.name}', age={self.age})"
```

随即可看到类中各个变量的值:`print(repr(person)) # 输出: Person(name='Alice', age=25)`

在dataclass中可通过设置把一些参数从repr中隐藏掉
```
@dataclass
class Context:
    dimension: str
    criteria: str = field(repr=False)  # 不显示在repr中
    skill: str
```
