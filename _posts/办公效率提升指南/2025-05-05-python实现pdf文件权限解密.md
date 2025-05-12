---
title: "python实现pdf文件的权限限制解除"
categories: 博客
tags:
    - 技术
permalink: /posts/:title.html
toc: true
---

# 基于python实现，解除pdf文件的限制，导出为图片

以下代码，仅限于个人研究学习使用，解密的PDF文件，仅限于个人拥有全部所有权的文件。请勿用于非法用途，若有非法使用，本人概不负责。



个人产生这个需求的情景：我想要将一个自己拥有的pdf文件，导出为图片，但是pdf文件，只给了我打印为纸质文件的权限。如果截图的话，非常不清晰，不方便。因此催生了这个需求。经过一番查找答案，使用AI，询问AI，最终得到了一个基本可用的如下代码，可以满足基本使用需求。仅限于个人研究学习使用，请勿用作非法用途。



## python安装：

去python官网，下载并安装python可执行程序。

## 依赖安装：

```python
pip install pyqt5 pypdf2 pikepdf
```

## 基本可用代码如下：

```python

```

## 执行测试：

将上述代码，保存为`PDF-Password-Remover.py`文件。

将如下命令写入`start.bat`脚本文件，双击即可自动调用这个文件：

```powershell
chcp 65001

@echo off
cd /d "%~dp0"
python PDF-Password-Remover.py
```

选择相应的pdf文件，点击开始解密，即可生成全部权限的pdf文件，可以使用相关工具导出为图片格式。
