# 该文档为macbook pro M2上使用chatglm2-6b模型的教程

## 硬件与软件
- 型号:MacBook Pro 14英寸 2023
- 芯片:Apple M2 Pro
- 内存:32G
- macOS:13.3(22E252)

### 在 Mac 平台上，情况更复杂一些：
- Mac 上只支持本地运行，也就是项目代码和模型分开下载，然后修改 web_demo.py 中模型地址运行
- 搭载了 Apple Silicon 或者 AMD GPU 的 Mac，需使用 MPS 后端在 GPU 上运行，修改 web_demo.py 中运行方式
- 加载需要 13G 内存，使用过程会不断上涨至 20G 以上，建议使用 32G 以上内存设备
- 内存不足设备，可使用量化后的 INT4 模型，但量化后只能使用 CPU 推理，为了充分使用 CPU 并行，还需要单独安装 OpenMP []。

### Mac 上的 MPS 支持 Apple silicon or AMD GPUs，还需要软件环境：
- macOS 12.3 or later
- Python 3.7 or later
- Xcode command-line tools: xcode-select --install

### 总结：
因为这些限制的存在，对小内存设备及 GPU 较差的设备极不友好，所以这里只推荐 32G 内存以上的 m1 pro/max/ultra 与 m2 pro/max/ultra 设备来进行测试，24G 内存的 m2 也可以尝试。

## 环境安装
1. 安装 Anaconda，用来安装 Pytorch Nightly 版
```bash
curl -O https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-arm64.sh
sh Miniconda3-latest-MacOSX-arm64.sh
```

2. 创建conda虚拟python环境
```bash
conda create -n chatglm2-6b python==3.10
```

3. 激活虚拟环境
```bash
conda activate chatglm2-6b
```

4. 安装 Pytorch Preview (Nightly) 版，用来提供 MPS 支持
```bash
conda install pytorch torchvision torchaudio -c pytorch-nightly
```

or

```bash
pip3 install --pre torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/nightly/cpu
```

5. 下载前端仓库
```bash
git clone https://github.com/THUDM/ChatGLM2-6B
cd ChatGLM2-6B

# 安装项目依赖
pip install -r requirements.txt
```

6. 安装 Git LFS，用来从 Huggingface 下载模型文件
```bash
brew install git-lfs

git lfs install
# 下载速度较慢,容易超时,先用下列命令下载小文件,再手动下载标注 LFS 的文件
GIT_LFS_SKIP_SMUDGE=1 git clone https://huggingface.co/THUDM/chatglm2-6b
```
[国内清华网盘](https://cloud.tsinghua.edu.cn/d/674208019e314311ab5c/)

## 修改 demo.py 代码
这里 Web 版有两种运行方式，加上 命令行 版，所以共有 3 个 demo 文件需要更改，修改内容是一样的。每个都需修改两个地方：
本地地址修改为模型的磁盘路径（第 2 步中从 Huggingface 下载的模型路径）
模型加载使用 to('mps') 方式

```python
# eg: web_demo.py
tokenizer = AutoTokenizer.from_pretrained("修改为第 2 步中存放 Huggingface 模型的路径", trust_remote_code=True)
model = AutoModel.from_pretrained("修改为第 2 步中存放 Huggingface 模型的路径", trust_remote_code=True).to('mps')
model = model.eval()
```

如果要部署 api 版，也要按同样方式修改 api.py 或 openai_api.py 中的代码，api 版中使用 cuda 的代码也需要自行修改

## 运行 Web Demo
1. 普通版 web_demo.py 首先安装 Gradio：pip install gradio，然后运行仓库中的 web_demo.py：
```bash
python web_demo.py
```

2. 基于 Streamlit 的网页版 Demo web_demo2.py 使用时首先需要额外安装依赖：pip install streamlit streamlit-chat，然后通过以下命令运行：
```bash
streamlit run web_demo2.py
```
