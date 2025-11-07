# python 多线程
```
from concurrent.futures import ThreadPoolExecutor, as_completed

def task(n):
    pass

# 创建线程池，其中最多运行 max_workers 个线程
with ThreadPoolExecutor(max_workers=3) as executor:
    # 提交任务,任务数量 = df 或者 list 中的数据条数
    # task 函数表示对单个任务的处理方法
    # 一个线程同时只可以处理一个任务
    futures = [executor.submit(task, row) for row in samples_list ]

    # 处理结果，future 表示一条样本（一个任务）的处理结果
    for future in futures:
        # 如果任务还没完成，会阻塞当前线程，直到任务完成；完成后会返回处理结果
        result = future.result()
        print(result)
```
