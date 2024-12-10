用这个来记录我学习去参加Kaggle的过程。

[TOC]

# git

```
git init
git clone https://github.com/username/repository-name.git
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
```

然后我使用了Tortoisegit. 不过之后还是要多熟悉命令行git的使用。

# prepare for Kaggle

检查了一下我的`anaconda`环境，有一个之前创建的`pytorch`环境，还有一个没用的环境（现在已经删除了），准备先用着。然后我现在电脑里没有`jupyter notebook`，等下安装一个。

```
conda env list
conda env remove --name envname
```

## jupyter notebook

直接在命令行安装就行

```
pip3 install jupyter
```

使用的时候，在想使用的那个路径框中输入`cmd`然后命令行输入`jupyter notebook`就可以。

![image-20241210144646577](readme.assets/image-20241210144646577.png)