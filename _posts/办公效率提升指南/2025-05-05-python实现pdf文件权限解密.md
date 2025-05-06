---
title: "python实现pdf文件的权限"
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
import sys
import os
from PyQt5.QtWidgets import (QApplication, QMainWindow, QWidget, QTabWidget, QVBoxLayout, QHBoxLayout,
                             QLabel, QPushButton, QLineEdit, QFileDialog, QMessageBox, QListWidget, 
                             QListWidgetItem, QAbstractItemView, QComboBox)
from PyQt5.QtCore import Qt, QMimeData
from PyPDF2 import PdfReader, PdfWriter, PdfMerger
from pikepdf import Pdf

class DragDropLineEdit(QLineEdit):
    """支持PDF拖放的自定义输入框（基于网页1/2实现）"""
    def __init__(self, parent=None):
        super().__init__(parent)
        self.setAcceptDrops(True)

    def dragEnterEvent(self, event):
        if event.mimeData().hasUrls():
            for url in event.mimeData().urls():
                if url.isLocalFile() and url.toLocalFile().lower().endswith('.pdf'):
                    event.acceptProposedAction()
                    return
        event.ignore()

    def dropEvent(self, event):
        for url in event.mimeData().urls():
            file_path = url.toLocalFile()
            if file_path.lower().endswith('.pdf'):
                self.setText(file_path)
                break

class PDFToolsUI(QMainWindow):
    """PDF全能处理工具主界面"""
    def __init__(self):
        super().__init__()
        self.setWindowTitle("PDF解密小工具 v1.0")
        self.setGeometry(300, 300, 800, 500)
        self.init_ui()

    def init_ui(self):
        """界面初始化"""
        tabs = QTabWidget()
        self.setCentralWidget(tabs)

        # 解密模块
        decrypt_tab = QWidget()
        self.decrypt_ui(decrypt_tab)
        tabs.addTab(decrypt_tab, "密码解除")

        # 加密模块
        encrypt_tab = QWidget()
        self.encrypt_ui(encrypt_tab)
        tabs.addTab(encrypt_tab, "文件加密")

        # 合并模块
        merge_tab = QWidget()
        self.merge_ui(merge_tab)
        tabs.addTab(merge_tab, "文件合并")

    def decrypt_ui(self, tab):
        """密码解除界面"""
        layout = QVBoxLayout()

        # 文件选择（支持拖放）
        file_layout = QHBoxLayout()
        self.decrypt_input = DragDropLineEdit()
        btn_choose = QPushButton("选择文件")
        btn_choose.clicked.connect(lambda: self.choose_file(self.decrypt_input))

        file_layout.addWidget(QLabel("输入文件:"))
        file_layout.addWidget(self.decrypt_input)
        file_layout.addWidget(btn_choose)

        # 密码输入
        self.decrypt_pass = QLineEdit()
        self.decrypt_pass.setEchoMode(QLineEdit.Password)

        # 操作按钮
        btn_decrypt = QPushButton("开始解密")
        btn_decrypt.clicked.connect(self.handle_decrypt)

        layout.addLayout(file_layout)
        layout.addWidget(QLabel("输入密码:"))
        layout.addWidget(self.decrypt_pass)
        layout.addWidget(btn_decrypt)
        tab.setLayout(layout)

    def encrypt_ui(self, tab):
        """文件加密界面"""
        layout = QVBoxLayout()

        # 输入文件选择（支持拖放）
        input_layout = QHBoxLayout()
        self.encrypt_input = DragDropLineEdit()
        btn_input = QPushButton("选择文件")
        btn_input.clicked.connect(lambda: self.choose_file(self.encrypt_input))
        input_layout.addWidget(QLabel("输入文件:"))
        input_layout.addWidget(self.encrypt_input)
        input_layout.addWidget(btn_input)

        # 输出路径选择
        output_layout = QHBoxLayout()
        self.encrypt_output = QLineEdit()
        btn_output = QPushButton("另存为")
        btn_output.clicked.connect(self.choose_output)
        output_layout.addWidget(QLabel("输出路径:"))
        output_layout.addWidget(self.encrypt_output)
        output_layout.addWidget(btn_output)

        # 加密参数
        param_layout = QHBoxLayout()
        self.encrypt_pass = QLineEdit()
        self.encrypt_pass.setEchoMode(QLineEdit.Password)
        self.encrypt_algo = QComboBox()
        self.encrypt_algo.addItems(["AES-128", "AES-256"])

        param_layout.addWidget(QLabel("加密密码:"))
        param_layout.addWidget(self.encrypt_pass)
        param_layout.addWidget(QLabel("算法:"))
        param_layout.addWidget(self.encrypt_algo)

        # 操作按钮
        btn_encrypt = QPushButton("开始加密")
        btn_encrypt.clicked.connect(self.handle_encrypt)

        layout.addLayout(input_layout)
        layout.addLayout(output_layout)
        layout.addLayout(param_layout)
        layout.addWidget(btn_encrypt)
        tab.setLayout(layout)

    def merge_ui(self, tab):
        """文件合并界面（支持拖放添加文件）"""
        layout = QVBoxLayout()

        # 文件列表（基于网页3实现拖放）
        self.merge_list = QListWidget()
        self.merge_list.setDragDropMode(QAbstractItemView.InternalMove)
        self.merge_list.setSelectionMode(QAbstractItemView.ExtendedSelection)
        self.merge_list.setAcceptDrops(True)
        self.merge_list.dragEnterEvent = self.merge_drag_enter
        self.merge_list.dropEvent = self.merge_drop

        # 操作按钮
        btn_layout = QHBoxLayout()
        btn_add = QPushButton("添加文件")
        btn_add.clicked.connect(self.add_merge_files)
        btn_remove = QPushButton("移除选中")
        btn_remove.clicked.connect(lambda: self.remove_items(self.merge_list))
        btn_up = QPushButton("上移")
        btn_up.clicked.connect(lambda: self.move_item(self.merge_list, -1))
        btn_down = QPushButton("下移")
        btn_down.clicked.connect(lambda: self.move_item(self.merge_list, 1))

        btn_layout.addWidget(btn_add)
        btn_layout.addWidget(btn_remove)
        btn_layout.addWidget(btn_up)
        btn_layout.addWidget(btn_down)

        # 输出路径
        output_layout = QHBoxLayout()
        self.merge_output = QLineEdit()
        btn_output = QPushButton("选择路径")
        btn_output.clicked.connect(self.choose_merge_output)
        output_layout.addWidget(QLabel("输出文件:"))
        output_layout.addWidget(self.merge_output)
        output_layout.addWidget(btn_output)

        # 合并按钮
        btn_merge = QPushButton("开始合并")
        btn_merge.clicked.connect(self.handle_merge)

        layout.addWidget(self.merge_list)
        layout.addLayout(btn_layout)
        layout.addLayout(output_layout)
        layout.addWidget(btn_merge)
        tab.setLayout(layout)

    # 核心功能方法（确保所有槽函数存在）
    def handle_decrypt(self):
        """解密处理（网页2验证）"""
        input_path = self.decrypt_input.text()
        password = self.decrypt_pass.text()

        if not os.path.exists(input_path):
            QMessageBox.critical(self, "错误", "文件不存在！")
            return

        output_path = f"{os.path.splitext(input_path)[0]}_unlocked.pdf"

        try:
            with Pdf.open(input_path, password=password) as pdf:
                pdf.save(output_path)
            QMessageBox.information(self, "成功", f"文件已保存至:\n{output_path}")
        except Exception as e:
            QMessageBox.critical(self, "错误", f"解密失败:\n{str(e)}")

    def handle_encrypt(self):
        """加密处理"""
        input_path = self.encrypt_input.text()
        output_path = self.encrypt_output.text()
        password = self.encrypt_pass.text()

        if not all([input_path, output_path, password]):
            QMessageBox.critical(self, "错误", "请填写所有字段！")
            return

        try:
            with Pdf.open(input_path) as pdf:
                encrypt_kwargs = {
                    'allow': Pdf.EncryptPermissions(print_lowres=True),
                    'R6': True if "256" in self.encrypt_algo.currentText() else False
                }
                pdf.save(output_path, encryption=encrypt_kwargs, password=password)
            QMessageBox.information(self, "成功", "文件加密完成！")
        except Exception as e:
            QMessageBox.critical(self, "错误", f"加密失败:\n{str(e)}")

    def handle_merge(self):
        """合并处理"""
        output_path = self.merge_output.text()
        if not output_path:
            QMessageBox.critical(self, "错误", "请选择输出路径！")
            return

        merger = PdfMerger()
        try:
            for i in range(self.merge_list.count()):
                file_path = self.merge_list.item(i).text()
                merger.append(file_path)

            merger.write(output_path)
            QMessageBox.information(self, "成功", "文件合并完成！")
        except Exception as e:
            QMessageBox.critical(self, "错误", f"合并失败:\n{str(e)}")
        finally:
            merger.close()

    # 通用工具方法
    def choose_file(self, target_field):
        """文件选择对话框"""
        path, _ = QFileDialog.getOpenFileName(
            self, "选择PDF文件", "", "PDF文件 (*.pdf)")
        if path:
            target_field.setText(path)

    def choose_output(self):
        """加密输出路径选择"""
        path, _ = QFileDialog.getSaveFileName(
            self, "保存文件", "", "PDF文件 (*.pdf)")
        if path:
            self.encrypt_output.setText(path)

    def choose_merge_output(self):
        """合并输出路径选择"""
        path, _ = QFileDialog.getSaveFileName(
            self, "保存合并文件", "", "PDF文件 (*.pdf)")
        if path:
            self.merge_output.setText(path)

    def add_merge_files(self):
        """添加合并文件"""
        files, _ = QFileDialog.getOpenFileNames(
            self, "选择PDF文件", "", "PDF文件 (*.pdf)")
        if files:
            for f in files:
                item = QListWidgetItem(f)
                self.merge_list.addItem(item)

    def remove_items(self, list_widget):
        """移除选中项"""
        for item in list_widget.selectedItems():
            list_widget.takeItem(list_widget.row(item))

    def move_item(self, list_widget, direction):
        """调整顺序"""
        current_row = list_widget.currentRow()
        if current_row >= 0:
            new_row = current_row + direction
            if 0 <= new_row < list_widget.count():
                item = list_widget.takeItem(current_row)
                list_widget.insertItem(new_row, item)
                list_widget.setCurrentRow(new_row)

    # 合并列表拖放事件（基于网页3实现）
    def merge_drag_enter(self, event):
        if event.mimeData().hasUrls():
            for url in event.mimeData().urls():
                if url.isLocalFile() and url.toLocalFile().lower().endswith('.pdf'):
                    event.acceptProposedAction()
                    return
        event.ignore()

    def merge_drop(self, event):
        for url in event.mimeData().urls():
            file_path = url.toLocalFile()
            if file_path.lower().endswith('.pdf'):
                self.merge_list.addItem(QListWidgetItem(file_path))

if __name__ == '__main__':
    app = QApplication(sys.argv)
    window = PDFToolsUI()
    window.show()
    sys.exit(app.exec_())
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


