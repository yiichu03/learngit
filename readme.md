用这个来记录我的机器学习课程结课大作业的完成

[TOC]

# 目录

- [参考链接](#参考链接)
  - [Fast.ai的PASCAL单/多目标检测](#fastai的pascal单多目标检测)
  - [PASCAL数据集讲解](#pascal数据集讲解)
- [作业思路](#作业思路)
  - [项目要求](#项目要求)
  - [对象/人识别相关方法](#对象人识别相关方法)
    - [PCA](#pca-principal-component-analysis)
    - [HOG vs. SIFT](#hog-vs-sift)
- [数据集选择](#数据集选择)
  - [PASCAL2007](#pascal2007)
- [实现方法](#实现方法)
  - [PCA (Principal Component Analysis)](#pca-principal-component-analysis)
  - [HOG (Histogram of Oriented Gradients)](#hog-histogram-of-oriented-gradients)
  - [SIFT (Scale-Invariant Feature Transform)](#sift-scale-invariant-feature-transform)
  - [路径推荐](#路径推荐)
    - [路径A（经典CV）](#路径a经典cv)
    - [路径B（混合方法）](#路径b混合方法)
- [代码示例](#代码示例)
  - [PASCAL VOC示例代码](#pascal-voc示例代码)
  - [扩展与优化建议](#扩展与优化建议)
- [小组合作建议](#小组合作建议)
  - [代码部分](#代码部分)
    - [共享部分](#共享部分)
    - [独立部分](#独立部分)
  - [报告部分](#报告部分)
- [Report建议](#report建议)
  - [结构](#结构)
  - [精简方法](#精简方法)
  - [LaTeX模板](#latex模板)


# 找的参考链接：

## 两个fast.ai的PASCAL单/多目标检测

https://www.shenxiaohai.me/fastai-8-objctDetct/

https://www.shenxiaohai.me/fastai-9-objctDetctMulti/

https://forums.fast.ai/t/deeplearning-lec9-notes/14113?u=cedric

## PASCAL数据集的讲解

https://arleyzhang.github.io/articles/1dc20586/

# 作业思路（gpt）

- Each Project: A group of two
- Topic:
  - Based on the class material
  - Focus on learning but not feature extraction
  - Can be related to your research, but it has to be extended
- What to submit:
  - A report
    - FontSize 12, single column, 2-3 pages.
  - The Code
    - Explicitly label, which part of the code is implemented yourself, by labeling
      - START: OWNCODE, right before your own implemented code
      - END: OWNCODE, right after your own implemented code

==Object/person recognition==

- PCA: Eigenfaces, eigendogs, etc.
- HOG vs. SIFT
- Data: Caltech 101/256, PASCAL, MIT Labelme, Yale face database, …

## 选择数据集

感觉选择PASCAL2007比较合适（好吧，也是gpt推荐的，gpt推荐了caltech 101和PASCAL，然后caltech好像一般都是用于分类，而PASCAL可以分类也可以检测，扩展性更强？我们可以根据我们进度或者怎样再决定，不过我个人觉得可以试试检测？虽然不是眼睛嘴巴检测，但也是检测了hhh）。

## 实现方法

关于老师给的：PCA, HOG vs. HIFT

### **PCA (Principal Component Analysis)**

- **作用**：PCA对高维特征进行降维，去除冗余，并在低维子空间中保留数据最大方差方向，有助于提升分类器的效率和可能的泛化性。
- **用法**：可以在提取完HOG或SIFT特征后对其进行PCA处理，将几千维的特征降到几十或上百维，以便更快的训练和更容易的可视化。

### **HOG (Histogram of Oriented Gradients)**

- **特征属性**：捕捉局部边缘和梯度分布信息，对形状变化敏感，计算简单且较为快速。对人类检测等任务常见于早期传统计算机视觉方法中（如经典的人体检测算法Dalat & Triggs使用HOG特征）。
- **优点**：实现与理解相对简单，可视化梯度方向特征较直观。
- **缺点**：对图像中尺度变化、视角变化的鲁棒性相对较弱。

### **SIFT (Scale-Invariant Feature Transform)**

- **特征属性**：具有尺度和旋转不变性，对光照和角度变化鲁棒性更好。SIFT特征往往适用于更复杂的场景识别，因为对图像中关键点的描述更丰富。
- **优点**：相比HOG，更加强大与稳定，当你的图像变化较大时更有优势。
- **缺点**：计算相对耗时，得到的特征可能需要进一步处理（如构建Bag-of-Visual-Words）以生成固定长度特征向量。

### 路径推荐（gpt）

在PASCAL VOC上，你可以尝试两条路径：

- **路径A（经典CV）**：对标注物体的bounding box进行裁剪，对该区域提取HOG特征，再用PCA降维，并使用SVM进行分类，看能否正确识别bounding box中的对象类别（如person vs. others）。然后再尝试SIFT特征，比较HOG与SIFT的性能与特性差异。
- **路径B（混合方法）**：在特征基础上加入深度学习预训练模型提取的深度特征，与HOG/SIFT进行对比。这会增加你的工作量和技术深度，但也会在报告中提供丰富的对比点。

### 代码示例（PASCAL VOC）

- PASCAL VOC数据集（例如VOC2007）可从 http://host.robots.ox.ac.uk/pascal/VOC/voc2007/ 下载。
- 下载后会有`Annotations/`、`JPEGImages/`和`ImageSets/Main/`文件夹。
- 下面的示例以简单分类任务为例：
  1. 从`ImageSets/Main`中找到一个特定类别（如person）的trainval.txt文件，它包含图像id和标签(1表示包含person，-1表示不包含)。
  2. 从对应的`JPEGImages`中读取图像，并在此示例中不做严格的对象检测，而是简化处理（如：直接使用整张图像提取特征，或根据标注裁剪出person bounding box区域，然后对该区域提特征）。
  3. 对该区域提取HOG特征，然后PCA降维，用SVM分类person vs. non-person。

请根据自己的需求对代码进行修改和扩展。例如，如果要使用SIFT，需要对ROI区域的特征进行SIFT关键点提取，然后构建bag-of-features向量化。

```python
# START: OWNCODE
import os
import cv2
import numpy as np
from skimage.feature import hog
from sklearn.decomposition import PCA
from sklearn.svm import SVC
from sklearn.metrics import classification_report, confusion_matrix
from sklearn.model_selection import train_test_split

# 假设你已经下载并解压了VOC2007数据集到'./VOC2007/'
voc_root = './VOC2007/'

# 选择"person"类别的annotation列表文件
# 此文件中每行格式为：<image_id> <label(1或-1)>
# 如：'ImageSets/Main/person_trainval.txt'
label_file = os.path.join(voc_root, 'ImageSets', 'Main', 'person_trainval.txt')

with open(label_file, 'r') as f:
    lines = f.readlines()

image_ids = []
labels = []
for line in lines:
    parts = line.strip().split()
    image_id = parts[0]
    label = int(parts[1])
    image_ids.append(image_id)
    labels.append(label)

X = []
y = []

img_size = (128, 128)  # 图像缩放尺寸，以便统一处理
for img_id, lbl in zip(image_ids, labels):
    img_path = os.path.join(voc_root, 'JPEGImages', img_id+'.jpg')
    if not os.path.exists(img_path):
        continue
    img = cv2.imread(img_path, cv2.IMREAD_COLOR)
    if img is None:
        continue
    
    # 简化处理：使用整张图像缩放并转灰度提取HOG（如需要可根据person的box做裁剪）
    gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
    gray = cv2.resize(gray, img_size)
    
    # 提取HOG特征
    hog_feat = hog(gray, 
                   orientations=9, 
                   pixels_per_cell=(8, 8), 
                   cells_per_block=(2, 2), 
                   block_norm='L2-Hys', 
                   transform_sqrt=True, 
                   feature_vector=True)
    X.append(hog_feat)
    # label由1/-1组成，可将-1转换为0作为负类
    y.append(1 if lbl == 1 else 0)

X = np.array(X)
y = np.array(y)

# 划分训练测试集
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42, stratify=y)

# PCA降维
pca = PCA(n_components=50, whiten=True, random_state=42)
X_train_pca = pca.fit_transform(X_train)
X_test_pca = pca.transform(X_test)

# 使用SVM分类
svm_clf = SVC(kernel='rbf', C=1.0, gamma='scale', class_weight='balanced', random_state=42)
svm_clf.fit(X_train_pca, y_train)

y_pred = svm_clf.predict(X_test_pca)

print("Classification Report:")
print(classification_report(y_test, y_pred, target_names=['non-person', 'person']))
print("Confusion Matrix:")
print(confusion_matrix(y_test, y_pred))
# END: OWNCODE

```

### 扩展和优化建议

**使用SIFT特征**：
对于SIFT，需要安装`opencv-contrib-python`，通过`cv2.SIFT_create()`提取关键点和描述子，然后需要构建一个Bag-of-Visual-Words模型来生成定长特征向量，再送入PCA和SVM中。

**可视化**：

- 可视化HOG特征：`skimage.feature.hog`有`visualize=True`的选项，可返回HOG图像用于展示。
- 可视化PCA主成分：展示前几个主成分对应的数据变化方向。
- 显示SIFT关键点：在图像上绘制SIFT特征点位置与方向。

**可解释性分析**：

- 使用LIME或SHAP对训练好的模型进行解释，查看模型判定person与非person的区域与特征依据。
- 对误分类的样本进行深度分析，研究特征提取与模型结构的改进空间。

**提高难度与深度学习特征对比**：
使用预训练卷积神经网络（例如ResNet）提取深度特征，与HOG/SIFT特征进行对比，将结果与PCA降维后的经典特征进行比较。

### 参考链接推荐（gpt）

**PASCAL VOC数据集主页**：
http://host.robots.ox.ac.uk/pascal/VOC/

**scikit-learn PCA文档与示例**：
https://scikit-learn.org/stable/modules/generated/sklearn.decomposition.PCA.html

**HOG特征介绍与示例**：
https://scikit-image.org/docs/dev/auto_examples/features_detection/plot_hog.html

**SIFT特征(OpenCV)**：
https://docs.opencv.org/4.x/da/df5/tutorial_py_sift_intro.html

**Bag-of-Visual-Words示例**：
GitHub上有许多关于BOVW的实现示例，例如：
https://github.com/kushalvyas/BOVW

**可解释性工具**：

- [LIME官方GitHub](https://github.com/marcotcr/lime)
- [SHAP官方GitHub](https://github.com/slundberg/shap)

**PASCAL VOC相关GitHub项目**：

- https://github.com/tzutalin/ImageNet_Utils（包含VOC数据读取与处理的脚本）
- 其他：直接在GitHub搜索“PASCAL VOC HOG SIFT”可发现若干教程与代码参考。

# 关于小组合作的理解与建议(gpt)

## 代码标注

### 代码开头可以写

```python
# This file contains shared code collaboratively developed by [Alice] and [Bob].
# The sections marked with START: OWNCODE and END: OWNCODE are independently implemented.

```

### 共享部分

数据加载与预处理（如图像裁剪、灰度化、特征提取的基础步骤）。

这部分代码可以完全相同，但要注明是 **小组共同完成的代码**，例如：

```python
# START: SHARED CODE
# This section was collaboratively developed by [Your Name] and [Teammate Name]
def preprocess_data(...):
    # 数据加载与预处理逻辑
    ...
# END: SHARED CODE

```

### 独立部分

核心算法的实现、模型的训练、调参部分。

这部分代码需要 **每人独立实现**，并标注 `OWNCODE`，例如：

```python
# START: OWNCODE
# Implemented by [Your Name]
model = SVC(kernel='rbf', C=1.0, gamma='scale')
model.fit(X_train, y_train)
y_pred = model.predict(X_test)
# END: OWNCODE

```

## 报告标注

报告中增加一个 **"Collaboration Statement"**： 在报告的最后说明：

- 哪些部分是团队共同完成的（如数据预处理）。
- 哪些部分是自己独立完成的（如模型实现和分析）。

```latex
Collaboration Statement:
The data preprocessing code and feature extraction pipeline were developed collaboratively with [Teammate Name].
The model implementation, parameter tuning, and analysis in this report are independently completed by [Your Name].

```



# Report

In the report, you should:

- Define the problem
  - What is the problem you are trying to solve?
- Show connection to class material
  - What is being classified, what are the classes, etc.
- Describe data
  - Train/test splits, etc.
- Show results of at least two learning models
  - Why do you see such results?
  - What results will you get if you tune the parameters?
  - What insights can you obtain?

## 结构（gpt推荐）

==这个结构我觉得gpt的回答仅供参考，我们可以最后自己调整，gpt说的要写的内容比较全比较杂，我们最后肯定是要挑重点（比如回答上面老师那三个问题的）去写在3页里面==

**精简后的结构（两到三页）**

### **Abstract**（100字以内）

用一段简洁的文字总结你的项目：

- 问题描述。
- 使用的主要方法（如特征提取方法和模型）。
- 关键结果和发现。
- 你工作的创新点。

### **Introduction & Related Work**（0.5页）

- 描述你解决的问题（Problem Definition）。

- 说明问题的重要性和实际应用背景。

- 明确你项目的研究目标（如：对图像分类问题进行研究，探索传统方法和深度学习方法的性能差异）。

- 总结课程中涉及的相关内容（如：PCA降维、HOG特征提取、SVM分类器等）。

- 强调你如何将课程中学到的技术应用于本项目。

### **Data & Methodology**（1页，包括数据描述、特征提取、模型介绍和参数设置）

**数据集描述**：

- 数据集来源（如PASCAL VOC 2007），类别数量，数据标注类型等。
- 数据预处理步骤（如图像裁剪、灰度化、特征提取）。

**训练与测试划分**：

- 描述你的训练集与测试集的划分比例（如70:30），或交叉验证方法。
- 提供一个简要统计（如数据分布表）。

**整体流程**：

- 用文字或简单流程图描述你的整体方法（数据预处理 -> 特征提取 -> 模型训练 -> 模型评估）。

**特征提取**：

- 说明你使用的特征提取方法（如HOG, SIFT），并简要描述其原理和优劣点。
- 例如：
  - HOG特征如何提取梯度方向直方图。
  - SIFT如何检测关键点并生成描述子。

**降维技术（PCA）**：

- 简要描述PCA的作用及其对高维特征的处理。

**分类模型**：

- 描述你使用的分类器（如SVM、Logistic Regression），以及选择这些模型的原因。

**参数设置**：

- 列出关键参数设置（如SVM的核函数类型、PCA主成分数目等）。

### **Results and Discussion**（0.75页，含表格和图示）

- 结果对比表
  - 用表格或图表展示不同模型/参数的性能对比，例如：
    - HOG + SVM vs. SIFT + SVM。
    - 不同PCA主成分数量对结果的影响。
- 性能指标
  - 提供准确率（Accuracy）、召回率（Recall）、精确率（Precision）和F1分数（F1-score）等指标。
- 错误分析
  - 对模型的错误案例进行分析，探讨其可能的原因。
- 可视化
  - 可视化HOG特征、SIFT关键点，或通过SHAP/LIME分析模型对图像的关注区域。

### **Conclusion**（0.25页，总结关键点）

**键发现**：

- 讨论你的实验结果表明了什么（如：HOG在简单分类任务中效果更好，而SIFT对复杂场景表现更优）。

**不足与改进**：

- 说明当前工作的不足之处（如数据量小，特征提取的局限性）。
- 提出未来改进方向（如尝试深度学习模型、使用更复杂的数据增强方法）。

总结你的主要工作和成果。

强调你的创新点和项目的意义。

### **References**（0.25页，简单列出引用文献）

==这个我觉得我们可以把那个https://arleyzhang.github.io/articles/1dc20586/里面提到的两个论文放上去，然后如果我们参考了一些博客之类的也可以标一下。不过应该就三四行，要不了四分之一面==

列出你引用的所有参考文献、教程、代码库等，格式统一（如IEEE格式或APA格式）。

示例：

- Dalal, N., & Triggs, B. (2005). Histograms of Oriented Gradients for Human Detection. In *IEEE CVPR*.
- scikit-learn: Machine Learning in Python. Journal of Machine Learning Research (2011).

### 精简内容方法

**图表优化**：

- 使用图表代替大段文字，例如结果对比表格、性能指标曲线图。
- 图表不要过多，2-3个关键图表足够。

**排版技巧**：

- 使用单栏格式，但尽量紧凑，例如减小段间距（在LaTeX中调整`\parskip`）。

## Report模板格式建议（gpt）

**字体**：Times New Roman, 字体大小 12。

**间距**：单倍行距。

**页边距**：默认设置即可（1英寸）。

**排版工具**：推荐使用LaTeX，模板更加规范；若使用Word，注意一致性。

==我感觉用latex和word都行，我有点想试试latex，好像都说他比较好排版。latex我之前大创学长有推荐**Overleaf**网站==

### LaTeX 初始模板

以下是一个简单的LaTeX模板，专为机器学习/计算机视觉课程项目设计。你可以直接复制并在LaTeX编辑器（如Overleaf）中使用。

```latex
\documentclass[12pt, a4paper]{article}

% 页边距设置
\usepackage[margin=1in]{geometry}

% 其他包
\usepackage{graphicx} % 用于插入图片
\usepackage{amsmath} % 数学公式
\usepackage{caption} % 图表标题
\usepackage{hyperref} % 超链接

% 文档标题信息
\title{Object Recognition Using HOG and PCA Features on PASCAL VOC Dataset}
\author{Your Name}
\date{\today}

\begin{document}

% 标题页
\maketitle
\begin{abstract}
This report presents an object recognition approach using HOG features and PCA dimensionality reduction on the PASCAL VOC dataset. We compare the performance of SVM classifiers trained with different feature extraction methods and analyze the impact of parameter tuning. Results show that HOG features combined with PCA significantly reduce computation time while maintaining competitive accuracy.
\end{abstract}

\section{Introduction}
Object recognition is a key problem in computer vision with applications ranging from surveillance to autonomous systems. In this project, we aim to classify objects in the PASCAL VOC dataset using traditional feature extraction methods (HOG, PCA) and compare their performance with a baseline approach.

\section{Data and Methodology}
The PASCAL VOC 2007 dataset contains 9,963 images labeled with 20 object categories. We used the "person" and "non-person" binary classification task. Images were resized to 128x128 pixels and converted to grayscale. HOG features were extracted, followed by PCA dimensionality reduction. An SVM classifier with RBF kernel was trained on the extracted features.

\subsection{Feature Extraction}
HOG features were computed with 9 orientations, a cell size of 8x8, and a block size of 2x2. PCA was applied to reduce the feature dimensions from 3,600 to 50.

\subsection{Model Training}
The SVM classifier was trained with a grid search for hyperparameter tuning (C: 0.1, 1, 10; gamma: scale).

\section{Results and Discussion}
\begin{figure}[h!]
\centering
\includegraphics[width=0.8\textwidth]{example-result.png}
\caption{Classification accuracy for HOG+PCA vs baseline methods.}
\end{figure}

The HOG+PCA+SVM model achieved an accuracy of 85\%, compared to 80\% with raw features. PCA reduced computation time by 40\%. Errors mainly occurred in images with occlusions or complex backgrounds.

\section{Conclusion}
We demonstrated the effectiveness of HOG and PCA features for object recognition on the PASCAL VOC dataset. Future work includes extending this approach to multi-class detection.

\begin{thebibliography}{9}
\bibitem{dalal2005} Dalal, N., \& Triggs, B. (2005). Histograms of Oriented Gradients for Human Detection. \textit{IEEE CVPR}.
\bibitem{pascal} PASCAL VOC Dataset. \url{http://host.robots.ox.ac.uk/pascal/VOC/}.
\bibitem{sklearn} Scikit-learn documentation: \url{https://scikit-learn.org}.
\end{thebibliography}

\end{document}

```

你说得对，上述设计如果按照详细描述可能会超过三页，为了保证在两到三页的范围内，我建议你精简内容，并使用以下策略优化篇幅：

### 如何控制篇幅

1. **精简内容**：
   - **合并部分章节**：将 `Introduction` 和 `Related Work` 合并为一章，减少重复背景描述。
   - **精简数据描述**：只包含数据集来源、类别和训练/测试划分比例，不需要过多赘述数据预处理细节。
   - **压缩结果分析**：用图表代替冗长文字描述，只突出关键对比结果。
2. **图表优化**：
   - 使用图表代替大段文字，例如结果对比表格、性能指标曲线图。
   - 图表不要过多，2-3个关键图表足够。
3. **排版技巧**：
   - 使用单栏格式，但尽量紧凑，例如减小段间距（在LaTeX中调整`\parskip`）。

------

### 精简后的结构（两到三页）

1. **Abstract**（100字以内）
2. **Introduction & Related Work**（0.5页）
3. **Data & Methodology**（1页，包括数据描述、特征提取、模型介绍和参数设置）
4. **Results and Discussion**（0.75页，含表格和图示）
5. **Conclusion**（0.25页，总结关键点）
6. **References**（0.25页，简单列出引用文献）

------

### LaTeX 初始模板

以下是一个简单的LaTeX模板，专为机器学习/计算机视觉课程项目设计。你可以直接复制并在LaTeX编辑器（如Overleaf）中使用。

```
latex复制代码\documentclass[12pt, a4paper]{article}

% 页边距设置
\usepackage[margin=1in]{geometry}

% 其他包
\usepackage{graphicx} % 用于插入图片
\usepackage{amsmath} % 数学公式
\usepackage{caption} % 图表标题
\usepackage{hyperref} % 超链接

% 文档标题信息
\title{Object Recognition Using HOG and PCA Features on PASCAL VOC Dataset}
\author{Your Name}
\date{\today}

\begin{document}

% 标题页
\maketitle
\begin{abstract}
This report presents an object recognition approach using HOG features and PCA dimensionality reduction on the PASCAL VOC dataset. We compare the performance of SVM classifiers trained with different feature extraction methods and analyze the impact of parameter tuning. Results show that HOG features combined with PCA significantly reduce computation time while maintaining competitive accuracy.
\end{abstract}

\section{Introduction}
Object recognition is a key problem in computer vision with applications ranging from surveillance to autonomous systems. In this project, we aim to classify objects in the PASCAL VOC dataset using traditional feature extraction methods (HOG, PCA) and compare their performance with a baseline approach.

\section{Data and Methodology}
The PASCAL VOC 2007 dataset contains 9,963 images labeled with 20 object categories. We used the "person" and "non-person" binary classification task. Images were resized to 128x128 pixels and converted to grayscale. HOG features were extracted, followed by PCA dimensionality reduction. An SVM classifier with RBF kernel was trained on the extracted features.

\subsection{Feature Extraction}
HOG features were computed with 9 orientations, a cell size of 8x8, and a block size of 2x2. PCA was applied to reduce the feature dimensions from 3,600 to 50.

\subsection{Model Training}
The SVM classifier was trained with a grid search for hyperparameter tuning (C: 0.1, 1, 10; gamma: scale).

\section{Results and Discussion}
\begin{figure}[h!]
\centering
\includegraphics[width=0.8\textwidth]{example-result.png}
\caption{Classification accuracy for HOG+PCA vs baseline methods.}
\end{figure}

The HOG+PCA+SVM model achieved an accuracy of 85\%, compared to 80\% with raw features. PCA reduced computation time by 40\%. Errors mainly occurred in images with occlusions or complex backgrounds.

\section{Conclusion}
We demonstrated the effectiveness of HOG and PCA features for object recognition on the PASCAL VOC dataset. Future work includes extending this approach to multi-class detection.

\begin{thebibliography}{9}
\bibitem{dalal2005} Dalal, N., \& Triggs, B. (2005). Histograms of Oriented Gradients for Human Detection. \textit{IEEE CVPR}.
\bibitem{pascal} PASCAL VOC Dataset. \url{http://host.robots.ox.ac.uk/pascal/VOC/}.
\bibitem{sklearn} Scikit-learn documentation: \url{https://scikit-learn.org}.
\end{thebibliography}

\end{document}
```

------

#### 模板功能说明

1. **紧凑的结构**：此模板针对两到三页报告设计，包含必备章节。
2. **可扩展性**：添加子章节（如`Feature Comparison`）或图表即可丰富内容。
3. **自动引用**：使用`\bibitem`管理参考文献，便于自动编号和格式化。

------

#### 使用方法

1. **本地编辑**：安装LaTeX工具（如TeX Live或MikTeX）并用编辑器（如Texmaker）运行。
2. **在线编辑**：将代码粘贴到 [Overleaf](https://www.overleaf.com/) 中，新建一个项目并编译。
3. **添加图片**：将图表保存为`example-result.png`并放在同目录，LaTeX会自动插入

#### 更多LaTeX资源(无🔗，可以搜索)

- **Overleaf模板库**：Overleaf Templates
- **LaTeX入门教程**：Overleaf Tutorial
- **LaTeX参考文档**：LaTeX Project

# 评分点

- Novelty of Application
- Models Chosen
- Experimental Results and Analysis
- Report Quality
