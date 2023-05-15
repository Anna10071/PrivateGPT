# privateGPT
使用LLMs的能力，在无需互联网连接的情况下向您的文档提问。100%私密，您的数据在任何时候都不会离开您的执行环境。您可以在无互联网连接的情况下摄取文档并提出问题！

使用[LangChain](https://github.com/hwchase17/langchain)、[GPT4All](https://github.com/nomic-ai/gpt4all)和[LlamaCpp](https://github.com/ggerganov/llama.cpp)构建。

# 环境设置
为了设置您的环境以运行这里的代码，首先安装所有的要求：

```shell
pip install -r requirements.txt
```

然后，下载这两个模型并将它们放在您选择的目录中。
- LLM：默认为[ggml-gpt4all-j-v1.3-groovy.bin](https://gpt4all.io/models/ggml-gpt4all-j-v1.3-groovy.bin)。如果您更喜欢不同的GPT4All-J兼容模型，只需下载它并在您的`.env`文件中引用它。
- 嵌入：默认为[ggml-model-q4_0.bin](https://huggingface.co/Pi3141/alpaca-native-7B-ggml/resolve/397e872bf4c83f4c642317a5bf65ce84a105786e/ggml-model-q4_0.bin)。如果您更喜欢不同的兼容嵌入模型，只需下载它并在您的`.env`文件中引用它。

将`example.env`重命名为`.env`并适当地编辑变量。
```
MODEL_TYPE: 支持LlamaCpp或GPT4All
PERSIST_DIRECTORY: 是您想要存放您的向量库的文件夹
LLAMA_EMBEDDINGS_MODEL: 您的LlamaCpp支持的嵌入模型的（绝对）路径
MODEL_PATH: 您的GPT4All或LlamaCpp支持的LLM的路径
MODEL_N_CTX: 嵌入和LLM模型的最大标记限制
```

注意：由于`langchain`加载`LLAMA`嵌入的方式，您需要指定嵌入模型二进制文件的绝对路径。这意味着如果您使用主目录快捷方式（例如`~/`或`$HOME/`），它将无法工作。

## 测试数据集
这个仓库使用[国情咨文的转录](https://github.com/imartinez/privateGPT/blob/main/source_documents/state_of_the_union.txt)作为一个例子。

## 摄取您自己的数据集的指南

将您所有的.txt、.pdf或.csv文件放入source_documents目录

运行以下命令摄取所有数据。

```shell
python ingest.py
```

它将创建一个包含本地向量存储的`db`文件夹。根据您的文档的大小，这将需要一些时间。您可以摄取尽可能多的文档，所有文档都将累积在本地嵌入数据库中。如果您想从一个空数据库开始，删除`db`文件夹。

注意：在摄取过程中，没有数据离开您的本地环境。您可以在没有互联网连接的情况下摄取数据。

## 向您的文档提问，本地化!
为了提出一个问题，运行像这样的命令：

```shell
python privateGPT.py
```

然后等待脚本要求您输入。

```shell
> 输入一个查询：
```

按回车。您需要等待20-30秒（取决于您的机器），而LLM模型消化提示并准备答案。完成后，它将打印出答案和它从您的文档中用作上下文的4个来源；然后您可以再提一个问题，无需重新运行脚本，只需再等待提示即可。

注意：您可以关闭您的互联网连接，脚本推断仍会工作。没有数据会离开您的本地环境。

输入`exit`来结束脚本。

# 它是如何工作的？
选择正确的本地模型和`LangChain`的强大能力，您可以在本地运行整个流水线，不会有任何数据离开您的环境，并且性能合理。

- `ingest.py`使用`LangChain`工具解析文档并使用`LlamaCppEmbeddings`在本地创建嵌入。然后使用`Chroma`向量存储在本地向量数据库中存储结果。
- `privateGPT.py`使用基于`GPT4All-J`或`LlamaCpp`的本地LLM来理解问题并创建答案。答案的上下文是通过相似性搜索从本地向量存储中提取的，以找到文档中正确的上下文片段。
- `GPT4All-J`包装器在LangChain 0.0.162中引入。

# 系统要求

## Python版本
要使用此软件，您必须安装Python 3.10或更高版本。早期版本的Python将无法编译。

## C++编译器
如果在`pip install`过程中在构建轮子时遇到错误，您可能需要在您的电脑上安装C++编译器。

### 对于Windows 10/11
在Windows 10/11上安装C++编译器，请按照以下步骤操作：

1. 安装Visual Studio 2022。
2. 确保选中以下组件：
   * 通用Windows平台开发
   * 适用于Windows的C++ CMake工具
3. 从[MinGW网站](https://sourceforge.net/projects/mingw/)下载MinGW安装程序。
4. 运行安装程序并选择"gcc"组件。

