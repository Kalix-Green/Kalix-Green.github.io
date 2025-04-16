---
title: "Fishspeech-1.5本地化部署"
categories: 人工智能
tags:
    - TTS
permalink: /:categories/:title.html
toc: true
author_profile: false

---

# Fishspeech-1.5本地化部署



## 下载发行包1.5和相关模型文件

在GitHub官方仓库[Releases · fishaudio/fish-speech](https://github.com/fishaudio/fish-speech/releases)下载v1.5版本安装包，并解压缩，切换到相应目录，目录查看如下：

```powershell
PS D:\My-TTS\my-fish-speech-1.5\fish-speech-1.5.0> pwd
```

执行结果如下：

```powershell
Path
----
D:\My-TTS\my-fish-speech-1.5\fish-speech-1.5.0
```

查看当前目录下文件：

```powershell
PS D:\My-TTS\my-fish-speech-1.5\fish-speech-1.5.0> dir
```

执行结果如下：

```powershell
目录: D:\My-TTS\my-fish-speech-1.5\fish-speech-1.5.0


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----        2024/12/21     23:27                .github
d-----        2024/12/21     23:27                docs
d-----        2024/12/21     23:27                fish_speech
d-----        2024/12/21     23:27                tools
------        2024/12/21     23:27             69 .dockerignore
------        2024/12/21     23:27            365 .gitignore
------        2024/12/21     23:27            532 .pre-commit-config.yaml
------        2024/12/21     23:27              0 .project-root
------        2024/12/21     23:27            438 .readthedocs.yaml
------        2024/12/21     23:27            236 API_FLAGS.txt
------        2024/12/21     23:27            352 docker-compose.dev.yml
------        2024/12/21     23:27           1317 dockerfile
------        2024/12/21     23:27           1074 dockerfile.dev
------        2024/12/21     23:27            171 entrypoint.sh
------        2024/12/21     23:27           5081 inference.ipynb
------        2024/12/21     23:27           4770 install_env.bat
------        2024/12/21     23:27          11340 LICENSE
------        2024/12/21     23:27           3752 mkdocs.yml
------        2024/12/21     23:27           1338 pyproject.toml
------        2024/12/21     23:27             63 pyrightconfig.json
------        2024/12/21     23:27           6586 README.md
------        2024/12/21     23:27            938 run_cmd.bat
------        2024/12/21     23:27           2040 start.bat
```

创建`checkpoints`目录，用于存放`fishspeech-1.5`的模型文件：

命令如下：

```powershell
PS D:\My-TTS\my-fish-speech-1.5\fish-speech-1.5.0> mkdir checkpoints
```

切换到`checkpoints`目录：

```powershell
PS D:\My-TTS\my-fish-speech-1.5\fish-speech-1.5.0> cd checkpoints
```

执行结果如下：

```powershell
PS D:\My-TTS\my-fish-speech-1.5\fish-speech-1.5.0\checkpoints>
```

安装git-lfs，执行如下命令：

```powershell
PS D:\My-TTS\my-fish-speech-1.5\fish-speech-1.5.0\checkpoints> git lfs install
```

使用git命令，拉取模型：

```powershell
PS D:\My-TTS\my-fish-speech-1.5\fish-speech-1.5.0\checkpoints> git clone https://hf-mirror.com/fishaudio/fish-speech-1.5
```

## 现在开始正式安装：

切回文件目录：

```powershell
PS D:\My-TTS\my-fish-speech-1.5\fish-speech-1.5.0> cd ..
```

基于当前目录，创建一个venv的python的虚拟环境：

```powershell
PS D:\My-TTS\my-fish-speech-1.5\fish-speech-1.5.0> python -m venv venv
```

加载相关虚拟环境：

```powershell
PS D:\My-TTS\my-fish-speech-1.5\fish-speech-1.5.0> .\venv\Scripts\activate
```

如果此处执行失败，被拒绝，“在此系统上禁止运行脚本”，则以管理员身份重新打开一个`powershell`，输入如下命令：

```powershell
PS C:\WINDOWS\system32> Set-ExecutionPolicy RemoteSigned
```

然后再原powershell中执行如下命令：

```powershell
PS D:\My-TTS\my-fish-speech-1.5\fish-speech-1.5.0> Get-ExecutionPolicy
```

执行结果如下：

```powershell
RemoteSigned
```

再次加载相关虚拟环境：

```powershell
PS D:\My-TTS\my-fish-speech-1.5\fish-speech-1.5.0> .\venv\Scripts\activate
```

正确的执行结果应该如下：

```powershell
(venv) PS D:\My-TTS\my-fish-speech-1.5\fish-speech-1.5.0>
```

安装cuda驱动，用于加速语音生成（否则速度太慢，毫无体验,pytorch官网目前最新是12.6版本的cuda，本地的cuda是12.8版本的）：

```powershell
(venv) PS D:\My-TTS\my-fish-speech-1.5\fish-speech-1.5.0> pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu126
```

其中，cuda文件下载的太慢，可以单独下载相应的`torch-2.6.0+cu126-cp312-cp312-win_amd64.whl`文件，可以使用迅雷等下载的速度快，然后将下载后的whl文件，移动到当前目录下，执行本地安装命令：

```powershell
(venv) PS D:\My-TTS\my-fish-speech-1.5\fish-speech-1.5.0> pip install .\torch-2.6.0+cu126-cp312-cp312-win_amd64.whl
```

然后再次执行上面的整体命令：

```powershell
(venv) PS D:\My-TTS\my-fish-speech-1.5\fish-speech-1.5.0> pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu126
```

结果如下：

```powershell
Looking in indexes: https://download.pytorch.org/whl/cu126
Requirement already satisfied: torch in d:\my-tts\my-fish-speech-1.5\fish-speech-1.5.0\venv\lib\site-packages (2.6.0+cu126)
Collecting torchvision
  Using cached https://download.pytorch.org/whl/cu126/torchvision-0.21.0%2Bcu126-cp312-cp312-win_amd64.whl.metadata (6.3 kB)
Requirement already satisfied: torchaudio in d:\my-tts\my-fish-speech-1.5\fish-speech-1.5.0\venv\lib\site-packages (2.6.0)
Requirement already satisfied: filelock in d:\my-tts\my-fish-speech-1.5\fish-speech-1.5.0\venv\lib\site-packages (from torch) (3.18.0)
Requirement already satisfied: typing-extensions>=4.10.0 in d:\my-tts\my-fish-speech-1.5\fish-speech-1.5.0\venv\lib\site-packages (from torch) (4.13.2)
Requirement already satisfied: networkx in d:\my-tts\my-fish-speech-1.5\fish-speech-1.5.0\venv\lib\site-packages (from torch) (3.4.2)
Requirement already satisfied: jinja2 in d:\my-tts\my-fish-speech-1.5\fish-speech-1.5.0\venv\lib\site-packages (from torch) (3.1.6)
Requirement already satisfied: fsspec in d:\my-tts\my-fish-speech-1.5\fish-speech-1.5.0\venv\lib\site-packages (from torch) (2024.2.0)
Requirement already satisfied: setuptools in d:\my-tts\my-fish-speech-1.5\fish-speech-1.5.0\venv\lib\site-packages (from torch) (78.1.0)
Requirement already satisfied: sympy==1.13.1 in d:\my-tts\my-fish-speech-1.5\fish-speech-1.5.0\venv\lib\site-packages (from torch) (1.13.1)
Requirement already satisfied: mpmath<1.4,>=1.1.0 in d:\my-tts\my-fish-speech-1.5\fish-speech-1.5.0\venv\lib\site-packages (from sympy==1.13.1->torch) (1.3.0)
Requirement already satisfied: numpy in d:\my-tts\my-fish-speech-1.5\fish-speech-1.5.0\venv\lib\site-packages (from torchvision) (1.26.4)
Requirement already satisfied: pillow!=8.3.*,>=5.3.0 in d:\my-tts\my-fish-speech-1.5\fish-speech-1.5.0\venv\lib\site-packages (from torchvision) (11.2.1)
Requirement already satisfied: MarkupSafe>=2.0 in d:\my-tts\my-fish-speech-1.5\fish-speech-1.5.0\venv\lib\site-packages (from jinja2->torch) (3.0.2)
Using cached https://download.pytorch.org/whl/cu126/torchvision-0.21.0%2Bcu126-cp312-cp312-win_amd64.whl (6.1 MB)
Installing collected packages: torchvision
Successfully installed torchvision-0.21.0+cu126
```

之后，执行相应的安装命令：

```powershell
(venv) PS D:\My-TTS\my-fish-speech-1.5\fish-speech-1.5.0> pip install -e .
```

执行成功后，启动UI界面，命令如下：

```powershell
(venv) PS D:\My-TTS\my-fish-speech-1.5\fish-speech-1.5.0> python.exe .\tools\run_webui.py
```

相应结果如下：

```powershell
D:\My-TTS\my-fish-speech-1.5\fish-speech-1.5.0\fish_speech\text\chn_text_norm\text.py:71: SyntaxWarning: invalid escape sequence '\d'
  + "(\d"
2025-04-16 22:15:52.663 | INFO     | __main__:<module>:53 - CUDA is not available, running on CPU.
2025-04-16 22:15:52.663 | INFO     | __main__:<module>:56 - Loading Llama model...
2025-04-16 22:15:57.313 | INFO     | tools.llama.generate:load_model:682 - Restored model from checkpoint
2025-04-16 22:15:57.314 | INFO     | tools.llama.generate:load_model:688 - Using DualARTransformer
2025-04-16 22:15:57.334 | INFO     | __main__:<module>:64 - Loading VQ-GAN model...
D:\My-TTS\my-fish-speech-1.5\fish-speech-1.5.0\venv\Lib\site-packages\vector_quantize_pytorch\vector_quantize_pytorch.py:445: FutureWarning: `torch.cuda.amp.autocast(args...)` is deprecated. Please use `torch.amp.autocast('cuda', args...)` instead.
  @autocast(enabled = False)
D:\My-TTS\my-fish-speech-1.5\fish-speech-1.5.0\venv\Lib\site-packages\vector_quantize_pytorch\vector_quantize_pytorch.py:630: FutureWarning: `torch.cuda.amp.autocast(args...)` is deprecated. Please use `torch.amp.autocast('cuda', args...)` instead.
  @autocast(enabled = False)
D:\My-TTS\my-fish-speech-1.5\fish-speech-1.5.0\venv\Lib\site-packages\vector_quantize_pytorch\finite_scalar_quantization.py:147: FutureWarning: `torch.cuda.amp.autocast(args...)` is deprecated. Please use `torch.amp.autocast('cuda', args...)` instead.
  @autocast(enabled = False)
D:\My-TTS\my-fish-speech-1.5\fish-speech-1.5.0\venv\Lib\site-packages\vector_quantize_pytorch\lookup_free_quantization.py:209: FutureWarning: `torch.cuda.amp.autocast(args...)` is deprecated. Please use `torch.amp.autocast('cuda', args...)` instead.
  @autocast(enabled = False)
2025-04-16 22:15:58.283 | INFO     | tools.vqgan.inference:load_model:43 - Loaded model: <All keys matched successfully>
2025-04-16 22:15:58.284 | INFO     | __main__:<module>:71 - Decoder model loaded, warming up...
2025-04-16 22:15:58.294 | INFO     | tools.llama.generate:generate_long:789 - Encoded text: Hello world.
2025-04-16 22:15:58.294 | INFO     | tools.llama.generate:generate_long:807 - Generating sentence 1/1 of sample 1/1
  2%|▉                                                 | 20/1023 [00:04<03:50,  4.36it/s]
2025-04-16 22:16:03.408 | INFO     | tools.llama.generate:generate_long:861 - Generated 22 tokens in 5.11 seconds, 4.30 tokens/sec
2025-04-16 22:16:03.408 | INFO     | tools.llama.generate:generate_long:864 - Bandwidth achieved: 2.74 GB/s
2025-04-16 22:16:03.409 | INFO     | tools.inference_engine.vq_manager:decode_vq_tokens:20 - VQ features: torch.Size([8, 21])
2025-04-16 22:16:27.657 | INFO     | __main__:<module>:98 - Warming up done, launching the web UI...
* Running on local URL:  http://127.0.0.1:7860

To create a public link, set `share=True` in `launch()`.
```

可以在浏览器中，输入网址[http://127.0.0.1:7860](http://127.0.0.1:7860)打开UI界面。

为了方便启动，在当前目录下，创建`start-venv.bat`脚本，脚本内容如下：

```powershell
@echo off
start cmd /k ".\venv\Scripts\activate && python.exe .\tools\run_webui.py"
```

并将此`start-venv.bat`脚本文件的快捷方式发送到桌面，方便下次调用，执行之后，即可打开UI界面。


