# 盘点分布式平台构建与分布式训练技术
## hadoop 和 spark
### hadoop
Hadoop是用来搭建集群（即以单台计算机为节点搭建的多设备平台）用来满足硬件设施的要求，它构建了基于计算机集群的大数据平台。主要解决**海量数据的存储和离线批处理**问题。

现在这种技术有点用不到了，用户可以直接去购买阿里云等云平台已经搭建好了的大数据平台，往往不会再自己去搭建平台了。
- HDFS：用于存储，把一个大文件自动切分成很多个小块（Block），然后分散地存储在集群中几十台、上百台机器的硬盘上。
- YARN：负责管理整个集群的CPU、内存等资源，当用户提交一个计算任务（比如一个Spark程序）时，YARN 就负责协调，决定把这个任务分配给哪台机器、分配多少资源去执行。这就实现了统一的资源管理和调度。
### Spark
是一种在集群上执行的程序，基于内存的分布式计算引擎。它不提供底层存储（通常依赖 HDFS 或其他存储系统），而是通过优化计算模型（如 DAG 执行引擎、内存缓存）解决 快速批处理、实时流处理、机器学习 等场景，强调低延迟和高吞吐量。
- 在学习了 Spark 之后就不会再使用 Hadoop 中的 Map 和 Reduce 操作了，Spark 有提供更集成的方法
## pytorch 提供的方法
官方原生支持的分布式训练能力，**用于多卡训练**
### 数据并行- - -`nn.DataParallel()`
- 单进程多线程
- 将模型复制到多个 GPU 上，将一个批次的数据拆分后分别送入各个 GPU 进行前向和反向传播，然后在主 GPU 上汇总梯度并更新
- 通过 `nvidia-smi` 查看显卡状态，可以看到是同一个进程（PID=1490362）分别占用了 2 个显卡
```
+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|    0   N/A  N/A      3335      G   /usr/lib/xorg/Xorg                              4MiB |
|    0   N/A  N/A   1490362      C   python                                      17168MiB |
|    1   N/A  N/A      3335      G   /usr/lib/xorg/Xorg                              4MiB |
|    1   N/A  N/A   1490362      C   python                                      16052MiB |
+-----------------------------------------------------------------------------------------+
```
#### 代码写法（只需要一步！！此外注意模型保存的时候加上额外的键`module`）
在加载完模型并移动到显卡0，再判断是否有多个显卡，有则使用 `nn.DataParallel()` 包装模型
```
model = BertForSequenceClassification.from_pretrained(
chinese-bert-wwm", num_labels=2).to(device)

if torch.cuda.device_count() > 1:
    print(f"有 {torch.cuda.device_count()} 个GPU")
    model = nn.DataParallel(model)
```
其中，参数`device_ids` 可以指定用哪些显卡，`output_device`用于指定使用哪个显卡汇总
```
model = nn.DataParallel(model, device_ids=[0, 1, 2], output_device=0)
```
模型保存

当保存经过 DataParallel 包装的模型时，其状态字典的键名会带有 module. 前缀
```
# 如果使用了分布式训练
if hasattr(model, 'module'):
    model.module.save_pretrained(f'{dir_name}/bert_classifier')
else:
    model.save_pretrained(f'{dir_name}/bert_classifier')
```
#### 缺陷
- 所有数据的拆分、损失的计算、梯度的汇总都在主GPU（output_device）上进行，导致主GPU的内存使用和计算负载远高于其他GPU，造成负载不均衡
- 该方法有点是使用非常简单，只需要一行代码，适合用于测试模型是否可以跑通。对于任何严肃的项目，都应该使用 `torch.nn.parallel.DistributedDataParallel`。见 pytorch 的[官方函数文档](https://docs.pytorch.org/docs/stable/generated/torch.nn.DataParallel.html#torch.nn.DataParallel) 来代替它，后者提供了更好的性能和真正的分布式训练能力。

![](https://pic-gino-prod.oss-cn-qingdao.aliyuncs.com/zhangli2025/20251024033145860-paste.png)

![](https://pic-gino-prod.oss-cn-qingdao.aliyuncs.com/zhangli2025/20251024094249405-paste.png)

### 分布式数据并行- - - `DistributedDataParallel, DDP`
- 主流的、推荐的方案
- 多进程模式，每个 GPU 都有一个独立的进程
- 使用 `torch.distributed` 库进行进程间通信，所有进程的梯度通过 `All-Reduce` 操作进行同步，效率远高于 DP
#### 代码写法
- 使用 DDP 微调 BERT 的完整代码见[链接](https://github.com/zxcvbnmkj/BERT_Finetuning)，分解的各个步骤如下

告诉当前进程应该使用哪张GPU，因为会启动多个进程
```
# 1，获取进程号，用于分配GPU，写在 import 语句的下方
local_rank = int(os.environ.get("LOCAL_RANK", 0))
# 进程数，等于使用到了 GPU 数目
world_size = int(os.environ.get("WORLD_SIZE", 1))
# 当前是否主进程
is_main_process = (local_rank == 0)
```
防止每个进程的 DataLoader 会产生完全相同的数据顺序，也防止模型随机初始化部分不同进程的初始值完全一样
- 进程0 (local_rank=0)：种子 = 1234 + 0*10 = 1234
- 进程1 (local_rank=1)：种子 = 1234 + 1*10 = 1244
- 进程2 (local_rank=2)：种子 = 1234 + 2*10 = 1254
```
# 2，不同进程设置不同的随机种子
# 设置 torch 的随机种子
torch.manual_seed(1234 + local_rank * 10)
# 设置 numpy 的随机种子
np.random.seed(1234 + local_rank * 10)
```
使得批次可以整除显卡个数，然后把全局批次大小（如64）改为单卡批次大小（若2卡，则是32）
```
if world_size > 1:
  assert args.batch_size % torch.cuda.device_count() == 0
  args.batch_size = args.batch_size // torch.cuda.device_count()
  # 将进程号和GPU号对应起来
  torch.cuda.set_device(local_rank)
  # nccl 是 NVIDIA的集合通信库，专为GPU间通信优化
  dist.init_process_group(backend="nccl")
  # 方便用于后面的 .to(device)
  device = torch.device('cuda:{}'.format(local_rank))
```
数据加载器
```
# 这个类继承自 Dataset ,它只是以元组的形式返回输入的各个参数而已。当数据集逻辑并不复杂的时候，可以直接使用它，从而避免自定义 Dataset
train_sample = TensorDataset(train_data['input_ids'], train_data['attention_mask'], train_data['labels'])
# shuffle=True 不能写在 DataLoader 中，应该放在 DistributedSampler 中。
# DistributedSampler 非常重要，它负责把全局批次（如64）随机拆分成 N 份，并保证 **每一份（每一个GPU）上面的样本不重复不缺少**
# 分布式中不可以写 train_dataloader = DataLoader(train_sample, batch_size=args.batch_size, shuffle=True)，因为这会使每个每张显卡都训练全局批次，等于重复训练了，一批数据被训练 N 次
train_sampler = DistributedSampler(train_sample,shuffle=True)
train_dataloader = DataLoader(train_sample, sampler=train_sampler, batch_size=args.batch_size)
```
兼容 单显卡 或 CPU 的加载方式
```
train_sample = TensorDataset(train_data['input_ids'], train_data['attention_mask'], train_data['labels'])
val_sample = TensorDataset(val_data['input_ids'], val_data['attention_mask'], val_data['labels'])
if world_size >1:
    # shuffle=True 不能写在 DataLoader 中，应该放在 DistributedSampler 中。
    # DistributedSampler 非常重要，它负责把全局批次（如64）随机拆分成 N 份，并保证 **每一份（每一个GPU）上面的样本不重复不缺少**
    # 分布式中不可以写 train_dataloader = DataLoader(train_sample, batch_size=args.batch_size, shuffle=True)，因为这会使每个每张显卡都训练全局批次，等于重复训练了，一批数据被训练 N 次
    train_sampler = DistributedSampler(train_sample,shuffle=True)
    train_dataloader = DataLoader(train_sample, sampler=train_sampler, batch_size=args.batch_size)
    val_sampler = DistributedSampler(val_sample, shuffle=True)
    val_dataloader = DataLoader(val_sample, sampler=train_sampler, batch_size=args.batch_size)
else:
    train_dataloader = DataLoader(train_sample, batch_size=args.batch_size, shuffle=True)
    val_dataloader = DataLoader(val_sample, batch_size=args.batch_size, shuffle=True)
```
使用 DDP 包装模型
```
model = torch.nn.parallel.DistributedDataParallel(model, device_ids=[local_rank], output_device=local_rank, find_unused_parameters=True)
```
在 `for epoch_i in range(epoch)` 之下使用，为新的 epoch 打乱样本
如果不调用 `set_epoch`，每个epoch的数据顺序将完全相同
```
if world_size > 1:
  train_sampler.set_epoch(epoch)
```
任何打印或者输入日志前，都要加一个判断当前是否主进程
```
if is_main_process:
    logging.info(f"当前是第{epoch_i}轮")
    logging.info("================训练中==================")
```
同步指标（在计算出 acc\p\r\f 等之后）
```
# 同步所有进程的指标 ，若不加则只能得到 主进程上面的批次数量数据
if world_size > 1:
    # 将指标转换为tensor
    train_metrics = torch.tensor([avg_train_loss, avg_train_acc, avg_train_p, avg_train_r, avg_train_f1],
                                 device=device)
    # 同步
    dist.all_reduce(train_metrics)
    # 均值
    train_metrics /= world_size
    avg_train_loss, avg_train_acc, avg_train_p, avg_train_r, avg_train_f1 = train_metrics.cpu().numpy()
```
保存模型
```
if avg_val_f1 > best_f1:
    if is_main_process:
        logging.info(f"存储当前最佳模型，它验证集 f1 为{avg_val_f1}")
        best_f1 = avg_val_f1
        patient = 0
        # 强制确保模型参数的内存布局是连续，用于防止错误 "你在保持一个非连续的张量"
        for name, param in model.named_parameters():
            if param is not None:
                param.data = param.data.contiguous()
        # 如果使用了分布式训练
        if hasattr(model, 'module'):
            model.module.save_pretrained(f'{dir_name}/bert_classifier')
        else:
            model.save_pretrained(f'{dir_name}/bert_classifier')
else:
    patient += 1
if patient == max_patient:
    break
```
仅主进程保存分词器
```
if is_main_process:
    tokenizer.save_pretrained(f'{dir_name}/bert_classifier')
```
释放资源
```
if world_size > 1:
    dist.destroy_process_group()
```
运行方法: `nproc_per_node` 为卡的数量
```
# 显卡编号从0开始，CUDA_VISIBLE_DEVICES=1,3 说明至少 4 张卡
CUDA_VISIBLE_DEVICES=1,3 torchrun --nproc_per_node=2 train.py
简写：只需要指定用几张显卡就行
torchrun --nproc-per-node=2 train.py
```
### 致命缺点
**有N个显卡，就有N个模型副本**

传统的 DataParallel 和 DistributedDataParallel 在每个GPU上都**复制完整的模型状态**，造成了巨大的内存冗余


## DeepSpeed
由微软开发的一个开源的深度学习优化库，**主要用于大模型**，和 DDP 一样属于多进程，相比起 DDP 它还进行了内存和通信优化
### 推荐阅读
- [官方介绍，来自微软](https://www.microsoft.com/en-us/research/blog/deepspeed-extreme-scale-model-training-for-everyone/?locale=fr-ca)
- [deepspeed 微调 BERT 实战](https://cloud.tencent.com/developer/article/2483965)
- [代码使用文档](https://www.deepspeed.ai/getting-started/)
### 内存管理技术: 零冗余优化器 (Zero Redundancy Optimizer，ZeRO)
- **ZeRO启用到哪个阶段，完全由训练者自己选择**
- ZeRO 的目的并非提升训练时间，而是提升空间利用率，是在**以时间换空间，即想办法节省内存**
- 但是使用了 ZeRO 方法，可以加大批次数，也可以训练的快一点
-  各种 ZeRO 组合

ZeRO-Stage 2 + Offload（最常见、最实用的组合）

ZeRO-Stage 1 + Offload：不常见的组合，因为 Offload 需要使用到优化器和梯度，一般不会只用Stage 1，只用优化器分区

ZeRO-Stage 3 + Offload：用于训练巨型模型
#### 回顾：单显卡的模型训练过程
1. 前向传播得到 LOSS
2. 计算损失 Loss 对最终输出的梯度
即在回答：**模型的最终输出值每变动一点点，损失会相应地变动多少**，即*对 Loss 关于 Y_pred 求导*

$梯度 = d(损失值) / d(预测值)$

3. 利用链式法则，进行梯度反向传播，使得每一个权重都得到自己的梯度
计算 Loss 对 W2、b2 的梯度，然后计算 Loss 对 ReLU 输出的梯度，再继续向前计算 Loss 对 W1、b1 的梯度。
4. 使用优化器更新模型权重
5. 清空梯度，进入新的循环
#### ZeRO Stage 1：优化器状态分区
1. 每个 GPU 上面都有一个模型副本
2. 将优化器状态均匀地分割成 N 份，每个 GPU 只存储和更新其中一份
3. 在反向传播后，每个 GPU 都拥有完整的梯度
4. **Reduce-Scatter（梯度聚合与分区）**: 每个 GPU 去聚合它所负责的优化器在其他所有 GPU 上的梯度均值，然后只保留其负责的优化器状态的梯度，丢弃其余梯度
5. 每个 GPU 使用自己那部分的梯度和优化器状态，独立更新自己负责的那部分参数
6. **All-Gather（参数同步）**：所有 GPU 都能从其他 GPU 那里获取更新后的参数，从而在本地重建完整的模型参数（GPU 之间互相发送自己的梯度信息，对齐颗粒度）
#### ZeRO Stage 2：梯度分区（步骤同上）
#### ZeRO Stage 3：模型参数分区
- 每个 GPU 只存储模型参数的 1/N
- 前向传播：当需要计算某一层时，通过 All-Gather 操作从所有其他 GPU 收集（Gather）构成这一层所需的完整参数。计算完成后，立即丢弃这些收集来的参数，只保留自己负责的那部分。
- 反向传播：同上
- 通信开销最大：在前向和反向传播的每一层都需要进行 All-Gather 操作
- **！！！【慎用，除非模型实在太大，一张卡存不下】** 它会引入巨大的通信开销，显著降低训练速度
####  最新版本 - - ZeRO-2
**不是 ZeRO 的阶段，而是该技术的版本，阶段还是上文 3 个**
- 支持对 2000 亿个参数进行模型训练，与最先进的技术相比速度提高了 10 倍
- 为最快的 BERT 训练提供支持
- 单张显卡上的训练也支持


### ZeRO-Offload
将**优化器状态和梯度**从 GPU 显存卸载到主机内存中，并使用 CPU 计算。从而让 GPU 能够训练远超其自身内存容量的大模型。
- 步骤：
1.  GPU 上进行前向传播和反向传播。反向传播结束后，GPU 上得到了 FP16 的梯度
2. 将梯度从 GPU 显存拷贝到 CPU 内存中
3. 优化器步骤（如 Adam 更新）在 CPU 上 执行
4. CPU 将更新后的**参数增量(不需要保存模型参数)** 或**新的参数值(模型一开始加载到内存，然后移动到 GPU ，内存可以不释放这一部分参数，直接在参数上面计算，得到新参数再传递到 GPU)** 从 CPU 内存拷贝回 GPU 显存
- ZeRO-Stage 2 + Offload的步骤：
1. GPU 聚合其他
2. 丢弃无关
3. GPU 把自己负责的那部分卸载到 CPU
4. GPU 从 CPU 中加载回自己负责的那部分参数
5. 与其余 GPU 通信，以更新所有参数
![](https://pic-gino-prod.oss-cn-qingdao.aliyuncs.com/zhangli2025/20251023172202182-paste.png)
>为什么不让 GPU 直接从 CPU 中加载所有新参数?
>- GPU 之间的通信带宽（尤其是同一台服务器内通过 NVLink）通常远高于 GPU 与 CPU 之间的带宽（通过PCIe）。让 CPU 来收集和分发所有参数会慢得多。
>- CPU 不是铁板一块，它也是分区的，每个区负责一块 GPU 的参数更新处理
### 3D 并行性
DeepSpeed 支持三种并行方法的灵活组合——ZeRO 支持的数据并行性、流水线并行性和张量切片模型并行性
### DeepSpeed Sparse Attention- - - 稀疏注意力
### 1-bit Adam
- 虽然名字是 Adam ，但该技术不是针对优化器，而是梯度的。目标是：在不影响模型最终收敛精度的情况下，将梯度同步的通信量减少到极致
- 将梯度同步的精度从 32 位大幅降低到仅用 1 位来表示，从而将通信量减少到原来的 1/32
- 在预热阶段（训练初期，不压缩Adam）,训练后期才压缩
- 在训练初期，大家用精确的地图（完整梯度）找方向；后期方向明确了，就只用简单的旗语（1位信号）来保持队伍整齐前进即可
### 代码写法
编写配置文件 `deepspeed_config.json`
```
{
  "train_batch_size": 128,   # 全局批次大小
  "gradient_accumulation_steps": 4,  # // 梯度累积步数
  "optimizer": {
    "type": "Adam",
    "params": {
      "lr": 0.00015,
      "eps": 1e-8    # 数值稳定性参数
    }
  },
  "scheduler": {
    "type": "WarmupDecayLR",   // DeepSpeed 自定义的一种学习率调度器，它结合了 热身（Warmup） 和 衰减（Decay） 两个阶段
    "params": {
      "warmup_min_lr": 0,    // 热身起始学习率
      "warmup_max_lr": 1.5e-4,     // 热身结束学习率
      "warmup_num_steps": 1000,     // 热身步数
      "warmup_type": "cosine" // 热身阶段学习率以 consine形式变化
    }
  },
  "fp16": {
    "enabled": true,   // 启用FP16训练，即混合精度学习
    "loss_scale": 0,   // 0=动态loss scaling, 其他值=静态loss scaling
    "loss_scale_window": 1000,    // 动态loss scaling的窗口大小
  },
  "zero_optimization": {
    "stage": 2,
    "contiguous_gradients": true,
    "cpu_offload": true    // 是否启用 CPU 卸载
  },
  "gradient_clipping": 1.0,     // 梯度裁剪阈值
}
```
使用 DeepSpeed 封装模型
```
if torch.cuda.device_count() > 1:
    print(f"有 {torch.cuda.device_count()} 个GPU")
    model_engine, _, _, _ = deepspeed.initialize(model=model, config_params="deepspeed_config.json")
```

## 要点
### 并行方法
- 数据并行：**复制模型，拆分数据**
  - 将一个大的训练批次数据平均分成若干份，每个GPU处理一份。
  - 所有GPU独立地进行前向传播和反向传播，计算出梯度。
  - torch 提供的分布式训练（DP、DDP）都是用的这种方式
  - DeepSpeed的ZeRO本质上也是一种内存优化的数据并行
  - **缺点：每个GPU都必须能容纳整个模型**
- 模型并行： 将一个模型本身拆分到多个设备上
  - 细分为 **流水线并行** 和 **张量并行** 两种
  - 只有模型非常非常大，一个显卡无法存储的时候，才需要使用模型并行，将模型分布式存储到不同的显卡里
- 流水线并行（Pipeline Parallelism）：将模型按层切分，形成流水线
  - 将一个完整的模型按网络层顺序切分成多个阶段，每个阶段放置在不同的GPU上
  - 一个训练批次的数据也被拆分成多个微批次
  - 第一个GPU处理完第一个微批次的前面几层后，将中间结果（激活值）传给下一个GPU，同时自己开始处理第二个微批次。如此类推，使多个GPU可以同时工作
  - 缺点： 会引入**流水线气泡**。即在一批数据开始和结束时，部分GPU处于空闲等待状态，造成了计算资源的浪费。批次越大，气泡比例相对越小
  - 代表技术： GPipe、DeepSpeed的Pipeline Parallelism
- 张量并行：将模型中的单个大权重大张量进行切分，分布到不同的设备上
  - 特点：跨节点（设备）之间的通讯频繁
  - 例如，一个全连接层的计算是 Y = XA + b。我们可以将权重矩阵 A 按列切分，每个GPU持有一部分列。每个GPU分别用输入 X 与自己那部分 A 做计算，得到输出 Y 的一部分，最后再将所有GPU的结果通过通信拼接起来。
  - 代表技术： Megatron-LM
- 混合并行：数据并行 + 模型并行
### 模型的内存开销
1. 仅推理
- 模型本身的参数，如 chatGLM3 是 4B，即 60 亿参数量
2. 也进行训练
- 梯度：反向传播时计算得到的梯度
- 优化器状态：如Adam优化器中的**动量（momentum）和方差（variance）**。这部分开销非常巨大，对于混合精度训练，Adam优化器状态所需内存是模型参数的12-16倍
- 前向传播过程中计算的中间结果
- 碎片化内存：模型张量不一定连续，可能存储在不同的内存部分，会降低内存利用率
### 分布式训练中的一些概念
- 全局进程编号（rank）：分配给整个系统中的每个进程的唯一标识符，全局有效，用于区分不同进程之间的通信
- 局部进程编号（local_rank）：仅在本节点内有效，用于区分同一节点内的不同进程之间的通信。
