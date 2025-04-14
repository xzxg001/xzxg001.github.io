---
title: Download from huggingface quickly
date: 2024-12-31
description: a tutorial to download from huggingface
image: image.png
tags: 
    - huggingface
    - tutorial
categories:
    - huggingface
    - tutorial
weight: 1
math: true
---
# 快速下载huggingface模型

如何快速下载huggingface模型——全方法总结 - 知乎[方法总结](https://zhuanlan.zhihu.com/p/663712983)

有的时候下载huggingface会出现错误，是因为网络连接相关的问题吗，因此有一些快速下载模型来解决这个问题的方法。

## 方法一：网页下载

在本站搜索，并在模型主页的`Files and Version`中下载文件。

## 方法二：huggingface-cli

[huggingface-cli ](https://hf-mirror.com/docs/huggingface_hub/guides/download#download-from-the-cli)是 Hugging Face 官方提供的命令行工具，自带完善的下载功能。

### **1. 安装依赖**

```bash
pip install -U huggingface_hub
```

### **2. 设置环境变量**

**Linux**

```bash
export HF_ENDPOINT=https://hf-mirror.com
```

**Windows Powershell**

```bash
$env:HF_ENDPOINT = "https://hf-mirror.com"
```

建议将上面这一行写入 `~/.bashrc`。

### **3.1 下载模型**

```bash
huggingface-cli download --resume-download gpt2 --local-dir gpt2
```

### **3.2 下载数据集**

```bash
huggingface-cli download --repo-type dataset --resume-download wikitext --local-dir wikitext
```

可以添加 `--local-dir-use-symlinks False` 参数禁用文件软链接，这样下载路径下所见即所得，详细解释请见上面提到的教程。

## 方法三：使用 hfd

***\*[hfd](https://gist.github.com/padeoe/697678ab8e528b85a2a7bddafea1fa4f)\**** 是本站开发的 huggingface 专用下载工具，基于成熟工具 `aria2`，可以做到稳定高速下载不断线。

**1. 下载hfd**

```bash
wget https://hf-mirror.com/hfd/hfd.sh
chmod a+x hfd.sh
```

**2. 设置环境变量**
**Linux**

```bash
export HF_ENDPOINT=https://hf-mirror.com
```

**Windows Powershell**

```bash
$env:HF_ENDPOINT = "https://hf-mirror.com"
```

### **3.1 下载模型**

```bash
./hfd.sh gpt2
```

### **3.2 下载数据集**

```bash
./hfd.sh wikitext --dataset
```

## 方法四：使用环境变量（非侵入式）

非侵入式，能解决大部分情况。huggingface 工具链会获取`HF_ENDPOINT`环境变量来确定下载文件所用的网址，所以可以使用通过设置变量来解决。

```bash
HF_ENDPOINT=https://hf-mirror.com python your_script.py
```

不过有些数据集有内置的下载脚本，那就需要手动改一下脚本内的地址来实现了。



## 常见问题

**Q**: 有些项目需要登录，如何下载？

**A**：部分 Gated Repo 需登录申请许可。为保障账号安全，本站不支持登录，需先前往 Hugging Face 官网登录、申请许可，在[官网这里获取 Access Token](https://huggingface.co/settings/tokens) 后回镜像站用命令行下载。
 部分工具下载 Gated Repo 的方法：

**huggingface-cli： 添加`--token`参数**

```bash
huggingface-cli download --token hf_*** --resume-download meta-llama/Llama-2-7b-hf --local-dir Llama-2-7b-hf
```

**hfd： 添加`--hf_username``--hf_token`参数**

```bash
hfd meta-llama/Llama-2-7b --hf_username YOUR_HF_USERNAME --hf_token hf_***
```

其余如`from_pretrained`、`wget`、`curl`如何设置认证 token，详见上面第一段提到的教程。