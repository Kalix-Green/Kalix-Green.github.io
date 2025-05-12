---
title: "python实现图片合成pdf文件"
categories: 博客
tags:
    - 技术
permalink: /posts/:title.html
toc: true

---

# 基于python，实现图片合成为PDF文件

以下代码，仅限于个人研究学习使用，处理操作的图像文件，仅限于个人拥有全部所有权的文件。请勿用于非法用途，若有非法使用，本人概不负责。



个人产生这个需求的情景：我想要将一个自己拥有的许多的图像图片，查阅不方便，我想要将它们合并为一个PDF文件，方便管理，方便查阅，因此催生了这个需求。经过一番查找答案，使用AI，询问AI，最终得到了一个基本可用的如下代码，可以满足基本使用需求。仅限于个人研究学习使用，请勿用作非法用途。



## python安装：

去python官网，下载并安装python可执行程序。

## 依赖安装：

```python
pip install PyQt5 Pillow psutil
```

## 基本可用代码如下：

```python
import sys
import os
import re
from concurrent.futures import ThreadPoolExecutor
from PIL import Image, ImageOps
from PyQt5.QtWidgets import (QApplication, QMainWindow, QWidget, QVBoxLayout,
                            QHBoxLayout, QPushButton, QFileDialog, QListWidget,
                            QLabel, QProgressDialog, QGroupBox, QCheckBox,
                            QSlider, QComboBox, QMessageBox, QSplitter,
                            QListWidgetItem)
from PyQt5.QtCore import Qt, QDir, QSize
from PyQt5.QtGui import QPixmap, QImage

class ImageToPDFConverter(QMainWindow):
    def __init__(self):
        super().__init__()
        self.preview_size = QSize(300, 400)  # 保持QSize定义
        self.image_files = []
        self.current_dir = ""
        self.settings = {
            'resolution': 300,
            'compression': 75,
            'page_size': '原始尺寸',
            'auto_rotate': True
        }
        self.init_ui()

    def init_ui(self):
        self.setWindowTitle('图片转PDF工具')
        self.setGeometry(300, 300, 1000, 600)

        main_widget = QWidget()
        self.setCentralWidget(main_widget)
        main_layout = QHBoxLayout(main_widget)

        splitter = QSplitter(Qt.Horizontal)

        # 左侧面板
        left_panel = QWidget()
        left_layout = QVBoxLayout(left_panel)

        # 文件管理区域
        dir_group = QGroupBox("文件管理")
        dir_layout = QVBoxLayout(dir_group)
        self.btn_choose = QPushButton("选择目录")
        self.btn_choose.clicked.connect(self.choose_directory)
        self.lbl_dir = QLabel("未选择目录")

        btn_group = QHBoxLayout()
        self.btn_add_files = QPushButton("添加文件")
        self.btn_remove_selected = QPushButton("移除选中")
        self.btn_clear_list = QPushButton("清空列表")
        btn_group.addWidget(self.btn_add_files)
        btn_group.addWidget(self.btn_remove_selected)
        btn_group.addWidget(self.btn_clear_list)

        dir_layout.addWidget(self.btn_choose)
        dir_layout.addWidget(self.lbl_dir)
        dir_layout.addLayout(btn_group)

        # 文件列表
        self.list_widget = QListWidget()
        self.list_widget.setDragDropMode(QListWidget.InternalMove)
        self.list_widget.itemSelectionChanged.connect(self.show_preview)

        # 设置区域
        settings_group = QGroupBox("转换设置")
        settings_layout = QVBoxLayout(settings_group)
        self.page_size_combo = QComboBox()
        self.page_size_combo.addItems(["原始尺寸", "A4 (210x297mm)", "Letter (216x279mm)"])
        self.compression_check = QCheckBox("启用压缩")
        self.compression_slider = QSlider(Qt.Horizontal)
        self.compression_slider.setRange(1, 100)
        self.compression_slider.setValue(75)
        self.rotate_check = QCheckBox("自动旋转")
        self.rotate_check.setChecked(True)

        settings_layout.addWidget(QLabel("页面尺寸:"))
        settings_layout.addWidget(self.page_size_combo)
        settings_layout.addWidget(self.compression_check)
        settings_layout.addWidget(self.compression_slider)
        settings_layout.addWidget(self.rotate_check)

        # 组装左侧布局
        left_layout.addWidget(dir_group)
        left_layout.addWidget(QLabel("文件列表:"))
        left_layout.addWidget(self.list_widget)
        left_layout.addWidget(settings_group)

        # 右侧预览区域
        right_panel = QWidget()
        right_layout = QVBoxLayout(right_panel)
        self.preview_label = QLabel("预览区域")
        self.preview_label.setAlignment(Qt.AlignCenter)
        self.preview_label.setFixedSize(self.preview_size)  # 正确使用QSize
        right_layout.addWidget(self.preview_label)

        # 转换按钮
        self.btn_convert = QPushButton("开始转换")
        self.btn_convert.clicked.connect(self.convert_to_pdf)
        left_layout.addWidget(self.btn_convert)

        # 连接按钮事件
        self.btn_add_files.clicked.connect(self.add_single_file)
        self.btn_remove_selected.clicked.connect(self.remove_selected_files)
        self.btn_clear_list.clicked.connect(self.clear_file_list)

        splitter.addWidget(left_panel)
        splitter.addWidget(right_panel)
        main_layout.addWidget(splitter)

    def show_preview(self):
        """修复尺寸转换错误的预览方法"""
        if not self.list_widget.currentItem():
            self.preview_label.clear()
            return

        try:
            # 获取完整路径
            filename = self.list_widget.currentItem().text()
            full_path = os.path.join(self.current_dir, filename) if self.current_dir else filename

            with Image.open(full_path) as img:
                # 自动旋转
                if self.settings['auto_rotate']:
                    img = ImageOps.exif_transpose(img)

                # 修复：将QSize转换为元组
                thumbnail_size = (
                    self.preview_size.width(), 
                    self.preview_size.height()
                )
                img.thumbnail(thumbnail_size)  # 使用元组参数

                # 处理透明通道
                if img.mode == 'RGBA':
                    background = Image.new('RGB', img.size, (255, 255, 255))
                    background.paste(img, mask=img.split()[3])
                    img = background
                elif img.mode not in ['RGB', 'L']:
                    img = img.convert('RGB')

                # 转换为QImage
                if img.mode == 'RGB':
                    format = QImage.Format_RGB888
                    bytes_per_line = img.width * 3
                elif img.mode == 'L':
                    format = QImage.Format_Grayscale8
                    bytes_per_line = img.width
                else:
                    format = QImage.Format_RGBA8888
                    bytes_per_line = img.width * 4

                qimg = QImage(img.tobytes(), img.width, img.height, 
                            bytes_per_line, format)

                # 保持宽高比缩放
                pixmap = QPixmap.fromImage(qimg).scaled(
                    self.preview_size.width(), 
                    self.preview_size.height(),
                    Qt.KeepAspectRatio,
                    Qt.SmoothTransformation
                )
                self.preview_label.setPixmap(pixmap)

        except Exception as e:
            QMessageBox.warning(self, "预览错误", f"{str(e)}")
            self.preview_label.setText("预览不可用")

    # 其他保持不变的方法...
    def choose_directory(self):
        directory = QFileDialog.getExistingDirectory(self, "选择目录", QDir.homePath())
        if directory:
            self.current_dir = directory
            self.lbl_dir.setText(directory)
            self.scan_image_files(directory)

    def scan_image_files(self, directory):
        valid_ext = ['.jpg', '.jpeg', '.png', '.bmp', '.tiff', '.webp']

        def natural_sort(s):
            return [int(c) if c.isdigit() else c.lower() for c in re.split(r'(\d+)', s)]

        files = sorted(
            [f for f in os.listdir(directory) if os.path.splitext(f)[1].lower() in valid_ext],
            key=natural_sort
        )
        self.image_files = files
        self.update_file_list()

    def update_file_list(self):
        self.list_widget.clear()
        for f in self.image_files:
            item = QListWidgetItem(f)
            item.setFlags(item.flags() | Qt.ItemIsUserCheckable)
            item.setCheckState(Qt.Checked)
            self.list_widget.addItem(item)

    def add_single_file(self):
        files, _ = QFileDialog.getOpenFileNames(self, "选择文件", 
            self.current_dir or QDir.homePath(),
            "图片文件 (*.jpg *.jpeg *.png *.bmp *.tiff *.webp)")
        if files:
            if self.current_dir:
                new_files = [os.path.relpath(f, self.current_dir) for f in files]
            else:
                new_files = files
                self.current_dir = os.path.dirname(files[0])
                self.lbl_dir.setText(self.current_dir)

            self.image_files.extend(new_files)
            self.update_file_list()

    def remove_selected_files(self):
        selected = sorted([self.list_widget.row(item) for item in self.list_widget.selectedItems()], reverse=True)
        for row in selected:
            del self.image_files[row]
        self.update_file_list()

    def clear_file_list(self):
        self.image_files.clear()
        self.list_widget.clear()

    def convert_to_pdf(self):
        if not self.image_files:
            QMessageBox.warning(self, "错误", "请先添加图片文件")
            return

        save_path, _ = QFileDialog.getSaveFileName(self, "保存PDF", 
                                                 QDir.homePath(), 
                                                 "PDF文件 (*.pdf)")
        if not save_path:
            return

        files = [os.path.join(self.current_dir, f) for f in self.get_checked_files()]

        progress = QProgressDialog("转换中...", "取消", 0, len(files), self)
        progress.setWindowModality(Qt.WindowModal)

        try:
            with ThreadPoolExecutor() as executor:
                futures = []
                images = []

                for file_path in files:
                    futures.append(executor.submit(self.process_image, file_path))

                for i, future in enumerate(futures):
                    if progress.wasCanceled():
                        break
                    progress.setValue(i)
                    img = future.result()
                    if img:
                        images.append(img)

                if images:
                    images[0].save(save_path, "PDF", 
                                 resolution=self.settings['resolution'],
                                 save_all=True, 
                                 append_images=images[1:],
                                 quality=self.settings['compression'])
                    QMessageBox.information(self, "完成", "转换成功!")
        except Exception as e:
            QMessageBox.critical(self, "错误", f"转换失败: {str(e)}")
        finally:
            progress.close()

    def process_image(self, file_path):
        try:
            img = Image.open(file_path)
            if self.settings['auto_rotate']:
                img = ImageOps.exif_transpose(img)

            img = self.resize_image(img)
            if img.mode != 'RGB':
                img = img.convert('RGB')
            return img
        except Exception as e:
            QMessageBox.warning(self, "错误", f"处理失败: {os.path.basename(file_path)}\n{str(e)}")
            return None

    def resize_image(self, img):
        size_map = {
            'A4 (210x297mm)': (2480, 3508),
            'Letter (216x279mm)': (2550, 3300)
        }
        target = self.settings['page_size']

        if target == '原始尺寸':
            return img

        if target in size_map:
            img.thumbnail(size_map[target])
        return img

    def get_checked_files(self):
        return [self.list_widget.item(i).text() 
               for i in range(self.list_widget.count())
               if self.list_widget.item(i).checkState() == Qt.Checked]

if __name__ == '__main__':
    # 高DPI设置
    if hasattr(Qt, 'AA_EnableHighDpiScaling'):
        QApplication.setAttribute(Qt.AA_EnableHighDpiScaling, True)
    if hasattr(Qt, 'AA_UseHighDpiPixmaps'):
        QApplication.setAttribute(Qt.AA_UseHighDpiPixmaps, True)

    app = QApplication(sys.argv)
    window = ImageToPDFConverter()
    window.show()
    sys.exit(app.exec_())
```

## 执行测试：

将上述代码，保存为`image2pdf.py`文件。



将如下命令写入`start.bat`脚本文件，双击即可自动调用这个文件：

```powershell
chcp 65001

@echo off
cd /d "%~dp0"
python image2pdf.py
```



## 简单教程如下：

选择对应的图片目录：

![](./../../assets/posts/image2pdf-20250507/1.png)

图片排序：

![](./../../assets/posts/image2pdf-20250507/2.png)

合成PDF文件：

![](./../../assets/posts/image2pdf-20250507/3.png)
