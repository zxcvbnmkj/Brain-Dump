# Sphinx
### 推荐阅读
- [官方文档](https://www.osgeo.cn/sphinx/index.html#get-started)
- [CSDN 的一篇不错帖子](https://blog.csdn.net/lly1122334/article/details/103970663)
## 一、准备工作
创建虚拟环境
```
conda create -n test python=3.10
conda activate test
```
使用 pdm 创建项目
```
pdm new sphinx_demo
```
如果 `pyproject.toml` 中指定的 python 解释器版本和当前激活的虚拟环境的 python 版本不一致，此时 pdm 会在项目根目录下生成虚拟环境（对应文件夹 `.venv`）
```
INFO: The saved Python interpreter doesn't match the project's requirement.  Trying to find another one.
WARNING: Project requires a python version of ==3.9. *, The virtualenv is being created for you as it cannot be matched to the right version.
INFO: python. use_venv is on, creating a virtualenv for this project...
Successfully installed cpython@3.9.24
Version: 3.9.24
Executable: /Users/nowcoder/Library/Application Support/pdm/python/cpython@3.9.24/bin/python3
Virtualenv is created successfully at /Users/nowcoder/zl/sphinx_demo/.venv
0:00:08 🔒 Lock successful.
Changes are written to pyproject.toml.
Synchronizing working set with resolved packages: 24 to add, 0 to update, 0 to remove
```
安装 sphinx
```
pdm add sphinx
```
## 二、Start
1. 新建 `doc` 和 `src` 两个文件夹，前者放文档，后者放代码
2. 代码写完后，`cd` 进 `doc` 文件夹，执行 `sphinx-quickstart`
```
(sphinx_demo-3.9) (base)  🐍 base  ~/zl/sphinx_demo/doc   master ±✚  sphinx-quickstart
Welcome to the Sphinx 7.4.7 quickstart utility.

Please enter values for the following settings (just press Enter to
accept a default value, if one is given in brackets).

Selected root path: .

You have two options for placing the build directory for Sphinx output.
Either, you use a directory "_build" within the root path, or you separate
"source" and "build" directories within the root path.
> Separate source and build directories (y/n) [n]: y

The project name will occur in several places in the built documentation.
> Project name: sphinx_demo 【需要输入项目名字】
> Author name(s): zxcvbnmkj   【作者】
> Project release []: V1.0      【版本】

If the documents are to be written in a language other than English,
you can select a language here by its language code. Sphinx will then
translate text that it generates into that language.

For a list of supported codes, see
https://www.sphinx-doc.org/en/master/usage/configuration.html#confval-language.
> Project language [en]: zh_CN  【语言输入中文，直接回车默认英文】

Creating file /Users/nowcoder/zl/sphinx_demo/doc/source/conf.py.
Creating file /Users/nowcoder/zl/sphinx_demo/doc/source/index.rst.
Creating file /Users/nowcoder/zl/sphinx_demo/doc/Makefile.
Creating file /Users/nowcoder/zl/sphinx_demo/doc/make.bat.

Finished: An initial directory structure has been created.

You should now populate your master file /Users/nowcoder/zl/sphinx_demo/doc/source/index.rst and create other documentation
source files. Use the Makefile to build the docs, like so:
   make builder
where "builder" is one of the supported builders, e.g. html, latex or linkcheck.
```
完毕后生成的内容为
```
doc/
    ├── build/
    ├── source/
    │   ├── _static/
    │   ├── _templates/
    │   ├── conf.py
    │   └── index.rst
    ├── make.bat
    └── Makefile
```
3. 修改 sphinx 的配置，在 `doc/source/conf.py` 中
- 导入代码文件夹的位置
- 修改扩展extensions，添加功能【包括注释中的文档】、【支持NumPy和Google风格】、【包括测试片段】、【链接到其他项目的文档】、【TODO项】、【文档覆盖率统计】、【通过javascript呈现数学】
```
import os
import sys
sys.path.insert(0, os.path.abspath('../../src'))

extensions = [
    'sphinx.ext.autodoc',
    'sphinx.ext.napoleon',
    'sphinx.ext.doctest',
    'sphinx.ext.intersphinx',
    'sphinx.ext.todo',
    'sphinx.ext.coverage',
    'sphinx.ext.mathjax',
]
```
4. 执行命令生成说明文档，确保当前路径在 `doc` 上
```
sphinx-apidoc -o source ../src/
```
完毕后，在 `doc/source` 下多出了
- 与 `src` 中 python 文件名字完全相同的 `.rst` 文件
- 生成 `modules.rst` 文件

5. 执行 `make html` ，完毕后目录变为

ps. 在这个示例中 `src` 文件夹下面只有 `1.py` 一个文件
```
doc/
    ├── build/
    │   ├── doctrees/
    │   │   ├── 1.doctree
    │   │   ├── environment.pickle
    │   │   ├── index.doctree
    │   │   └── modules.doctree
    │   └── html/
    │       ├── _sources/
    │       │   ├── 1.rst.txt
    │       │   ├── index.rst.txt
    │       │   └── modules.rst.txt
    │       ├── _static/
    │       │   ├── alabaster.css
    │       │   ├── basic.css
    │       │   ├── custom.css
    │       │   ├── doctools.js
    │       │   ├── documentation_options.js
    │       │   ├── file.png
    │       │   ├── language_data.js
    │       │   ├── minus.png
    │       │   ├── plus.png
    │       │   ├── pygments.css
    │       │   ├── searchtools.js
    │       │   ├── sphinx_highlight.js
    │       │   └── translations.js
    │       ├── .buildinfo
    │       ├── 1.html
    │       ├── genindex.html
    │       ├── index.html
    │       ├── modules.html
    │       ├── objects.inv
    │       ├── py-modindex.html
    │       ├── search.html
    │       └── searchindex.js
    ├── source/
    │   ├── _static/
    │   ├── _templates/
    │   ├── 1.rst
    │   ├── conf.py
    │   ├── index.rst
    │   └── modules.rst
    ├── make.bat
    └── Makefile
```
6. 打开 `build/html/1.html` 这就是对应文件的模块说明

![](https://pic-gino-prod.oss-cn-qingdao.aliyuncs.com/zhangli2025/20251014113803968-paste.png)
## 三、变更代码后重新生成说明文档
- 删除 `doc/build` 下的所有文件夹
- 删除 `doc/source` 下除 `index.rst` 的所有 `.rst` 文件
- 在 `doc` 下执行命令 `sphinx-apidoc -o source ../src/`
- 在 `doc` 下执行命令 `make html`
## 四、切换文档风格
1. 安装一种主题库 `pdm add sphinx_rtd_theme`
2. 使用这种主题库，在 `doc/source/conf.py` 中的 `html_theme`
