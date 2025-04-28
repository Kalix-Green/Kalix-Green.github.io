---
title: "FFmpeg实现视频抽帧提取图片"
categories: 博客
tags:
    - 技术
permalink: /posts/:title.html
toc: true
---

## 第一步，安装FFmpeg二进制文件

在如下github仓库 ***https://github.com/GyanD/codexffmpeg/releaseshttps://github.com/GyanD/codexffmpeg/releases***

下载FFmpeg的二进制文件：***ffmpeg-7.1.1-full_build.zip***

解压缩到一个目录：并将  ****\ffmpeg-2025-04-23\bin*** 添加到系统环境变量，使命令行终端下，可以执行`ffmpeg`命令。



## 第二步，将如下命令，写入一个bat文件

```powershell
chcp 65001

@echo off
setlocal enabledelayedexpansion

for %%a in (*.mp4 *.avi *.mov *.mkv) do (
    set "fname=%%~na"
    if not exist "!fname!\" (
        mkdir "!fname!"
    )
    ffmpeg -i "%%a" -vf "select=not(mod(n\,15))" -vsync vfr "!fname!\output_%%04d.png"
)

echo 处理完成
pause
```

这个脚本的功能是，遍历当前目录下的所有视频格式文件，实现每15帧抽取1帧，输出图片到对应的目录。

如果需要修改抽取帧数的频率，可修改脚本的如下部分：

```powershell
ffmpeg -i "%%a" -vf "select=not(mod(n\,15))" -vsync vfr "!fname!\output_%%04d.png"
```
