# CPP

## 环境
1. 安装C++编译器

（1）下载 [mingw-w 64](https://github.com/niXman/mingw-builds-binaries/releases)，该项目是 gcc 的一个完整的运行时环境

（2）使用`gcc -- version`来验证是否安装成功

2. 安装 VC code 的2个 C++ 相关扩展。
安装 `C/C++` 扩展和`C/C++ Extension Pack`扩展

3. `git clone`  之后，使用 VC code 打开项目。
4. 编译代码
```
g++ main.cpp -o main # 生成可执行文件 main（Windows 下为 main.exe）
```
5. 其他
若有细节问题，参考[环境配置](https://blog.csdn.net/2502_90930283/article/details/145979656)

VS code 生成的`.vscode`可以删除，git 的时候不需要加上它

## 运行
```
# 运行 C 语言
gcc ./test.c -o test.exe
./test.exe

# 运行 C++
g++ ./test.c -o test.exe
./test.exe
```

点击 VCcode 的`run code`其实就是默认执行以下，它省略了 `.exe` 后缀，只会生成 `try` 文件
```
[Running] cd "/Users/nowcoder/zl/microtubule-dynamics-simulation/" && g++ try.cpp -o try && "/Users/nowcoder/zl/microtubule-dynamics-simulation/"try
hello word!
```
很有可能遇到代码使用的是C++11以上的版本，但是编译器默认以就版本执行，此时需要把运行命令改成
```
gcc ./test.c -o test.exe -std=c++11
```
或者编辑`/vscode/task.json`，在`args:的-g`下面加一行（中间这行是新加）
```
"-g",
"-std=c++11",
"${file}",
```

## C++与C不同的核心语法

### 数据类型
```
unsigned int 无符号整数

向量容器：
vector<类型> 变量名(大小);
vector<unsigned int> N(nsize);
# 赋初始值
N = {4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4};
```
