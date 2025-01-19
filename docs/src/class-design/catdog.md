---
title: 基于深度学习的猫狗识别系统【深度学习课设】
shortTitle: 1.猫狗识别系统
date: 2025-01-17
sticky: true
start: true
category:
  - 课程设计
tag:
  - 课程设计
---

## 作品演示

<img src="https://cdn.golangcode.cn/images/202501182209931.png" alt="在这里插入图片描述"  />
![在这里插入图片描述](https://cdn.golangcode.cn/images/202501182209097.png)
![在这里插入图片描述](https://cdn.golangcode.cn/images/202501182209889.png)
![在这里插入图片描述](https://cdn.golangcode.cn/images/202501182209061.png)

## 代码
采用模型VGG16、ALEXNet、Resnet18，训练测试。python版本3.10.11 。

数据集：和鲸社区猫狗图像数据集。[https://www.heywhale.com/mw/project/631aedb893f47b16cb062b2a](https://www.heywhale.com/mw/project/631aedb893f47b16cb062b2a)

### 1.train_and_test.py

```python
# 导入 PyTorch 库和相关模块
import torch                                 # PyTorch 的核心库，提供张量计算和自动求导功能
import torchvision.transforms as transforms  # 提供图像数据增强和预处理的功能
from torch.utils.data import Dataset         # 用于自定义数据集
from torch import nn, optim                  # nn 用于构建神经网络，optim 用于优化算法
from PIL import Image                        # 用于加载和处理图像文件
import time                                  # 用于记录训练时长和其他时间相关操作
import torchvision.models as models          # 包含一些预训练模型，如 AlexNet、ResNet 等
import os                                    # 用于与操作系统交互，如文件路径处理、创建目录等
import matplotlib.pyplot as plt              # 用于绘制图表，如准确率曲线、损失曲线等
from tqdm import tqdm                        # 用于显示训练过程中的进度条
from sklearn.metrics import confusion_matrix # 用于计算混淆矩阵，评估分类性能
import seaborn as sns                        # 用于绘制混淆矩阵的热图，提供美观的图表风格


device = torch.device('cpu')

# 数据预处理：缩放到224x224大小，并转换为Tensor
transformer = transforms.Compose([transforms.Resize((224, 224)), transforms.ToTensor()])

# 加载训练数据集
DogTrainImageList = os.listdir(r"./catsdogs/train/Dog")  # 加载训练集中的狗图片列表
CatTrainImageList = os.listdir(r"./catsdogs/train/Cat")  # 加载训练集中的猫图片列表
train_label = []  # 存储训练数据的标签
train_data = []  # 存储训练数据的图像数据
dog_train_data_dir = r"./catsdogs/train/Dog/"  # 狗的图片目录路径
cat_train_data_dir = r"./catsdogs/train/Cat/"  # 猫的图片目录路径

# 将狗的图片加载进训练数据集
for i in range(len(DogTrainImageList)):
    train_label.append(1)  # 狗的标签为1
    dog_img = Image.open(dog_train_data_dir + DogTrainImageList[i]).convert('RGB')  # 打开图片并转换为RGB
    dog_img = transformer(dog_img)  # 进行预处理
    train_data.append(dog_img)  # 添加到训练数据

# 将猫的图片加载进训练数据集
for i in range(len(CatTrainImageList)):
    train_label.append(0)  # 猫的标签为0
    cat_img = Image.open(cat_train_data_dir + CatTrainImageList[i]).convert('RGB')  # 打开图片并转换为RGB
    cat_img = transformer(cat_img)  # 进行预处理
    train_data.append(cat_img)  # 添加到训练数据

# 加载测试数据集（与训练集类似）
DogTestImageList = os.listdir(r"./catsdogs/train/Dog")
CatTestImageList = os.listdir(r"./catsdogs/train/Cat")
test_label = []  # 存储测试数据的标签
test_data = []  # 存储测试数据的图像数据
dog_test_data_dir = r"./catsdogs/train/Dog/"  # 狗的测试图片目录路径
cat_test_data_dir = r"./catsdogs/train/Cat/"  # 猫的测试图片目录路径

# 将狗的测试图片加载进测试数据集
for i in range(len(DogTestImageList)):
    test_label.append(1)  # 狗的标签为1
    dog_img = Image.open(dog_test_data_dir + DogTestImageList[i]).convert('RGB')
    dog_img = transformer(dog_img)
    test_data.append(dog_img)

# 将猫的测试图片加载进测试数据集
for i in range(len(CatTestImageList)):
    test_label.append(0)  # 猫的标签为0
    cat_img = Image.open(cat_test_data_dir + CatTestImageList[i]).convert('RGB')
    cat_img = transformer(cat_img)
    test_data.append(cat_img)

# 自定义的数据集类，用于加载图像数据
class DealDataset(Dataset):
    def __init__(self, data, label, transform=None):
        self.data = data  # 图像数据
        self.label = label  # 图像标签
        self.transform = transform  # 图像预处理

    def __getitem__(self, index):
        data, label = self.data[index], int(self.label[index])  # 获取指定索引的数据和标签
        return data, label  # 返回数据和标签

    def __len__(self):
        return len(self.data)  # 返回数据集的大小

# 将训练数据集和测试数据集包装为DealDataset对象
TrainDataSet = DealDataset(train_data, train_label, transform=transformer)
TestDataSet = DealDataset(test_data, test_label, transform=transformer)

# 定义AlexNet模型
class AlexNet(nn.Module):
    def __init__(self):
        super(AlexNet, self).__init__()
        # 定义卷积层部分
        self.conv = nn.Sequential(
            nn.Conv2d(3, 64, kernel_size=11, stride=4),
            nn.ReLU(),
            nn.MaxPool2d(kernel_size=3, stride=2),
            nn.BatchNorm2d(64),
            nn.Conv2d(64, 192, kernel_size=5, padding=2),
            nn.ReLU(),
            nn.MaxPool2d(kernel_size=3, stride=2),
            nn.BatchNorm2d(192),
            nn.Conv2d(192, 384, kernel_size=3, padding=1),
            nn.ReLU(),
            nn.Conv2d(384, 256, kernel_size=3, padding=1),
            nn.ReLU(),
            nn.Conv2d(256, 256, kernel_size=3, padding=1),
            nn.ReLU(),
            nn.MaxPool2d(kernel_size=3, stride=2),
            nn.BatchNorm2d(256)
        )
        # 定义全连接层部分
        self.fc = nn.Sequential(
            nn.Linear(256 * 5 * 5, 4096),
            nn.ReLU(),
            nn.Dropout(0.5),
            nn.Linear(4096, 4096),
            nn.ReLU(),
            nn.Dropout(0.5),
            nn.Linear(4096, 2)  # 输出2个类别：猫或狗
        )

    def forward(self, img):
        feature = self.conv(img)  # 通过卷积层提取特征
        output = self.fc(feature.view(img.shape[0], -1))  # 展开特征并通过全连接层进行分类
        return output

# 使用预训练的VGG16模型，并修改最后的全连接层以适应2个输出类别
class VGG16(nn.Module):
    def __init__(self, num_classes=2):
        super(VGG16, self).__init__()
        self.model = models.vgg16(pretrained=True)  # 加载预训练的VGG16模型
        self.model.classifier[-1] = nn.Linear(self.model.classifier[-1].in_features, num_classes)  # 修改输出层

    def forward(self, x):
        return self.model(x)  # 返回模型的输出

# 使用ResNet18模型，并修改最后的全连接层以适应2个输出类别
class ResNet18(nn.Module):
    def __init__(self):
        super(ResNet18, self).__init__()
        self.model = models.resnet18(pretrained=False)  # 加载ResNet18模型
        self.model.fc = nn.Linear(self.model.fc.in_features, 2)  # 修改输出层为2个类别

    def forward(self, x):
        return self.model(x)  # 返回模型的输出

# 绘制混淆矩阵的函数
def plot_combined_confusion_matrix(true_labels_dict, predicted_labels_dict, classes,
                                   save_path='combined_confusion_matrix.png'):
    # 创建一个子图，用来显示多个模型的混淆矩阵
    fig, axes = plt.subplots(1, len(true_labels_dict), figsize=(15, 5))

    # 遍历每个模型并绘制其混淆矩阵
    for i, (model_name, true_labels) in enumerate(true_labels_dict.items()):
        predicted_labels = predicted_labels_dict[model_name]
        cm = confusion_matrix(true_labels, predicted_labels)  # 计算混淆矩阵

        # 使用Seaborn绘制热图
        sns.heatmap(cm, annot=True, fmt="d", cmap="Blues", xticklabels=classes, yticklabels=classes,
                    ax=axes[i], cbar=False, annot_kws={"size": 14})
        axes[i].set_xlabel('Predicted labels', fontsize=12)
        axes[i].set_ylabel('True labels', fontsize=12)
        axes[i].set_title(f'{model_name} Confusion Matrix', fontsize=14)

    # 调整布局并保存图像
    plt.tight_layout()
    plt.savefig(save_path)
    plt.show()

# 计算模型在测试集上的准确率
def evaluate_accuracy(data_iter, net, device=None):
    if device is None and isinstance(net, torch.nn.Module):
        device = list(net.parameters())[0].device  # 获取模型的设备
    acc_sum, n = 0.0, 0
    predicted_labels = []
    true_labels = []
    with torch.no_grad():  # 在测试时不需要计算梯度
        for X, y in tqdm(data_iter, desc="加载中：", leave=True):
            net.eval()  # 将模型设置为评估模式
            outputs = net(X.to(device))  # 获取模型输出
            predicted = outputs.argmax(dim=1)  # 获取预测的标签
            true_labels.extend(y.cpu().numpy())  # 存储真实标签
            predicted_labels.extend(predicted.cpu().numpy())  # 存储预测标签
            acc_sum += (predicted == y.to(device)).float().sum().cpu().item()  # 累加准确的样本数
            n += y.shape[0]  # 累加样本总数

    return acc_sum / n, true_labels, predicted_labels  # 返回准确率，真实标签和预测标签

# 训练和评估模型
def train_and_evaluate_models(models, model_names, train_iter, test_iter, batch_size, optimizer_dict, device,
                              num_epochs, save_model_paths, plot_path):
    train_acc_history = {name: [] for name in model_names}  # 存储训练过程中每个模型的训练准确率
    test_acc_history = {name: [] for name in model_names}  # 存储测试过程中每个模型的测试准确率
    train_loss_history = {name: [] for name in model_names}  # 存储每个模型的训练损失

    # 存储每个模型的混淆矩阵数据
    true_labels_dict = {name: [] for name in model_names}
    predicted_labels_dict = {name: [] for name in model_names}

    # 迭代训练周期
    for epoch in range(num_epochs):
        for model, model_name in zip(models, model_names):  # 遍历每个模型
            model.train()
            optimizer = optimizer_dict[model_name]  # 获取当前模型的优化器
            loss_fn = torch.nn.CrossEntropyLoss()  # 定义损失函数
            scheduler = torch.optim.lr_scheduler.StepLR(optimizer, step_size=3, gamma=0.7)  # 学习率衰减策略
            train_l_sum, train_acc_sum, n, batch_count, start = 0.0, 0.0, 0, 0, time.time()

            # 训练每个模型
            for X, y in train_iter:
                X, y = X.to(device), y.to(device)
                y_hat = model(X)  # 获取模型预测
                loss = loss_fn(y_hat, y)  # 计算损失

                optimizer.zero_grad()  # 清空梯度
                loss.backward()  # 反向传播
                optimizer.step()  # 更新参数

                train_l_sum += loss.item()  # 累加损失
                train_acc_sum += (y_hat.argmax(dim=1) == y).sum().cpu().item()  # 累加准确的样本数
                n += y.shape[0]
                batch_count += 1

            scheduler.step()  # 学习率衰减

            # 计算训练集和测试集的准确率
            train_acc = train_acc_sum / n
            test_acc, true_labels, predicted_labels = evaluate_accuracy(test_iter, model, device)

            # 存储每个模型的混淆矩阵数据
            true_labels_dict[model_name].extend(true_labels)
            predicted_labels_dict[model_name].extend(predicted_labels)

            train_acc_history[model_name].append(train_acc)
            test_acc_history[model_name].append(test_acc)
            train_loss_history[model_name].append(train_l_sum / batch_count)

            print(f'{model_name} epoch {epoch + 1}, loss {train_l_sum / batch_count:.4f}, '
                  f'train acc {train_acc:.3f}, test acc {test_acc:.3f}, time {time.time() - start:.1f} sec')

            # 保存模型
            torch.save(model.state_dict(), save_model_paths[model_name])  # 保存模型的权重
            print(f"{model_name} Model saved to {save_model_paths[model_name]} after epoch {epoch + 1}")

    # 在所有训练完成后生成混淆矩阵的综合图
    plot_combined_confusion_matrix(true_labels_dict, predicted_labels_dict, ['Cat', 'Dog'],
                                   save_path=os.path.join(plot_path, 'combined_confusion_matrix.png'))

    return train_acc_history, test_acc_history, train_loss_history

# 可视化训练结果并保存
def plot_and_save_results(train_acc_history, test_acc_history, train_loss_history, num_epochs, save_plots_path):
    plt.figure(figsize=(10, 5))
    # 绘制每个模型的训练与测试准确率曲线
    for model_name in ['AlexNet', 'ResNet18', 'VGG16']:
        if model_name in train_acc_history and model_name in test_acc_history:
            plt.plot(range(num_epochs), train_acc_history[model_name], label=f'{model_name} Train Accuracy')
            plt.plot(range(num_epochs), test_acc_history[model_name], label=f'{model_name} Test Accuracy')

    plt.xlabel('Epochs')
    plt.ylabel('Accuracy')
    plt.title('AlexNet, ResNet18, VGG16 - Training and Test Accuracy Comparison')
    plt.legend()
    plt.grid(True)
    plt.savefig(os.path.join(save_plots_path, 'accuracy_plot.png'))  # 保存准确率图像
    plt.show()

    plt.figure(figsize=(10, 5))
    # 绘制每个模型的训练损失曲线
    for model_name in ['AlexNet', 'ResNet18', 'VGG16']:
        if model_name in train_loss_history:
            plt.plot(range(num_epochs), train_loss_history[model_name], label=f'{model_name} Train Loss')

    plt.xlabel('Epochs')
    plt.ylabel('Loss')
    plt.title('Training Loss Comparison')
    plt.legend()
    plt.grid(True)
    plt.savefig(os.path.join(save_plots_path, 'loss_plot.png'))  # 保存损失图像
    plt.show()

if __name__ == '__main__':
    # 设置训练参数
    num_epochs = 25  # 设置为可配置参数
    batch_size = 16  # 设置为可配置参数
    learning_rate = 0.009  # 设置为可配置参数

    save_model_paths = {
        'AlexNet': 'AlexNet.pth',
        'ResNet18': 'ResNet18.pth',
        'VGG16': 'VGG16.pth'
    }
    save_plots_path = './python'
    os.makedirs(save_plots_path, exist_ok=True)  # 创建保存模型和图像的文件夹

    # 创建模型实例
    alexnet_model = AlexNet().to(device)
    resnet_model = ResNet18().to(device)
    vgg_model = VGG16().to(device)

    # 创建数据加载器
    train_iter = torch.utils.data.DataLoader(TrainDataSet, batch_size=batch_size, shuffle=True, num_workers=2)
    test_iter = torch.utils.data.DataLoader(TestDataSet, batch_size=batch_size, shuffle=False, num_workers=2)

    # 优化器字典
    optimizer_dict = {
        'AlexNet': torch.optim.SGD(alexnet_model.parameters(), lr=learning_rate),
        'ResNet18': torch.optim.SGD(resnet_model.parameters(), lr=learning_rate),
        'VGG16': torch.optim.SGD(vgg_model.parameters(), lr=learning_rate)
    }

    # 训练并评估
    models = [alexnet_model, resnet_model, vgg_model]
    model_names = ['AlexNet', 'ResNet18', 'VGG16']
    train_acc_history, test_acc_history, train_loss_history = train_and_evaluate_models(
        models, model_names, train_iter, test_iter, batch_size, optimizer_dict, device, num_epochs, save_model_paths, save_plots_path)

    # 绘制并保存准确率和损失曲线
    plot_and_save_results(train_acc_history, test_acc_history, train_loss_history, num_epochs, save_plots_path)

```

### 2、view.py（可视化界面）

```python
import sys
from PyQt5.QtCore import Qt
from PyQt5.QtWidgets import QApplication, QWidget, QLabel, QPushButton, QFileDialog, QVBoxLayout, QGridLayout, \
    QTextEdit, QComboBox, QSpacerItem, QSizePolicy
from PyQt5.QtGui import QPixmap, QFont, QTextCursor
import torch
import torch.nn as nn
import torchvision.transforms as transforms
from PIL import Image
import torchvision.models as models


class AnimalClassifierApp(QWidget):
    def __init__(self):
        super().__init__()

        self.initUI()

    def initUI(self):
        self.setWindowTitle('猫狗识别系统')
        self.resize(600, 400)  # 更小的窗口尺寸

        # 创建布局
        grid = QGridLayout()
        grid.setContentsMargins(10, 10, 10, 10)  # 设置间距
        grid.setSpacing(5)  # 设置控件间距

        # 显示图像的标签
        self.image_label = QLabel(self)
        self.image_label.setFixedSize(250, 250)  # 调整图像显示尺寸
        self.image_label.setAlignment(Qt.AlignCenter)
        grid.addWidget(self.image_label, 1, 0, 2, 1)

        # 识别结果的标签
        self.result_label = QTextEdit(self)
        self.result_label.setFixedSize(250, 80)
        self.result_label.setReadOnly(True)
        self.result_label.setStyleSheet("color: red; font-size: 14px;")
        self.result_label.setAlignment(Qt.AlignCenter)
        grid.addWidget(self.result_label, 1, 1, 1, 2)

        # 模型选择下拉框
        self.model_selector = QComboBox(self)
        self.model_selector.addItem("AlexNet")
        self.model_selector.addItem("VGG16")
        self.model_selector.addItem("ResNet18")
        grid.addWidget(self.model_selector, 2, 0, 1, 2)

        # 按钮布局
        button_layout = QVBoxLayout()
        button_layout.setSpacing(5)  # 设置按钮间距

        # 上传图像按钮
        upload_btn = QPushButton('上传', self)
        upload_btn.clicked.connect(self.load_image)
        button_layout.addWidget(upload_btn)

        # 识别按钮
        recognize_btn = QPushButton('识别', self)
        recognize_btn.clicked.connect(self.classify_image)
        button_layout.addWidget(recognize_btn)

        # 添加按钮布局
        button_layout.addSpacerItem(QSpacerItem(10, 10, QSizePolicy.Minimum, QSizePolicy.Expanding))
        grid.addLayout(button_layout, 3, 1, 1, 2)

        self.setLayout(grid)

        # 加载模型
        self.device = torch.device('cpu')

        # 定义数据转换
        self.transform = transforms.Compose([
            transforms.Resize((148, 148)),
            transforms.ToTensor(),
            transforms.Normalize(mean=[0.4, 0.4, 0.4], std=[0.2, 0.2, 0.2])
        ])

        self.image_path = ''
        self.model = None  # 模型初始化为空

    def load_image(self):
        options = QFileDialog.Options()
        options |= QFileDialog.ReadOnly
        file_name, _ = QFileDialog.getOpenFileName(self, "上传图片", "", "图片文件 (*.jpg *.jpeg *.png)",
                                                   options=options)
        if file_name:
            self.image_path = file_name
            pixmap = QPixmap(file_name)
            pixmap = pixmap.scaled(self.image_label.width(), self.image_label.height(), Qt.KeepAspectRatio)
            self.image_label.setPixmap(pixmap)
            self.result_label.setText('识别结果: ')

    def classify_image(self):
        if self.image_path:
            # 根据选择的模型加载相应的模型
            selected_model = self.model_selector.currentText()
            if selected_model == "AlexNet":
                self.model = self.load_alexnet_model()
            elif selected_model == "VGG16":
                self.model = self.load_vgg16_model()
            elif selected_model == "ResNet18":
                self.model = self.load_resnet18_model()

            image = Image.open(self.image_path).convert('RGB')
            image_tensor = self.transform(image).unsqueeze(0).to(self.device)

            with torch.no_grad():
                output = self.model(image_tensor)
                probabilities = torch.nn.functional.softmax(output, dim=1)
                confidence, predicted = torch.max(probabilities, 1)
                label = 'cat' if predicted.item() == 0 else 'dog'
                confidence = confidence.item()

            # 将图像转换为QPixmap
            pixmap = QPixmap(self.image_path)
            pixmap = pixmap.scaled(self.image_label.width(), self.image_label.height(), Qt.KeepAspectRatio)
            self.image_label.setPixmap(pixmap)

            # 设置识别结果字体颜色和对齐方式
            self.result_label.setText(f'识别结果: {label} \n\n置信度: {confidence:.2f}')
            self.result_label.setAlignment(Qt.AlignCenter)
            cursor = self.result_label.textCursor()
            cursor.select(QTextCursor.Document)
            self.result_label.setTextCursor(cursor)

    def load_alexnet_model(self):
        model = models.alexnet(pretrained=True)
        model.classifier[6] = nn.Linear(model.classifier[6].in_features, 2)  # 修改最后一层
        model = model.to(self.device)
        model.eval()
        return model

    def load_vgg16_model(self):
        model = models.vgg16(pretrained=True)
        model.classifier[6] = nn.Linear(model.classifier[6].in_features, 2)  # 修改最后一层
        model = model.to(self.device)
        model.eval()
        return model

    def load_resnet18_model(self):
        model = models.resnet18(pretrained=True)
        model.fc = nn.Linear(model.fc.in_features, 2)  # 修改最后一层
        model = model.to(self.device)
        model.eval()
        return model


if __name__ == '__main__':
    app = QApplication(sys.argv)
    ex = AnimalClassifierApp()
    ex.show()
    sys.exit(app.exec_())

```

## 报告+源码获取地址

详细内容请关注微信公众号：**GolangCode**，输入“**课程设计报告**” 获取详细内容。如果对你有小小的帮助，也请给我点个小赞赞。

![GolangCode](https://cdn.golangcode.cn/images/202501171944968.png)
