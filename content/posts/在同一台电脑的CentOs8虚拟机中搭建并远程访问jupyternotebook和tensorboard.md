---
title: "在同一台电脑的CentOs8虚拟机中搭建并远程访问jupyternotebook和tensorboard"
subtitle: "本文又名：<关于我也不知道为什么这标题能起的这么冗长拗口这件事>"
date: 2020-03-21
lastmod : 2020-03-21
author: "简单的I电脑"
tags: ["虚拟机","JupyterNotebook","TensorBoard"]
categories: ["文档"]
draft: false
---
## 一、概述

**本文 **用于在**同一台**电脑的 CentOs8 虚拟机中搭建并访问 jupyter notebook 和 tensoboard。

**初衷** ~~可能是闲的~~ 为了和某些教程中的操作环境**保持一致**。

## 二、准备

**1.VM 中搭建的CentOs8虚拟机并已经正确配置了网络环境[^1]**

**2.虚拟机中正确安装了python3，jupyter 和 tensorboard**

**3.主机安装有SecureCRT8（Xshell等也可）**

## 三、搭建 Jupyter Notebook 远程访问

### 1.设置远程访问

**1）骗出jupyter notebook配置文件的存储路径**

命令 `jupyter notebook --generate-config` 用于将配置**初始化**，但我们可以虚晃一枪，将它用于获取配置文件的存储路径

<div align=center><img src="/posts/2020/3-21/图 3-1-1.png" alt="" style="zoom:100%;" /></div>

**2）配置相关参数**

打开`jupyter_notebook_config.py`，在文件任意处键入：

```bash
c.NotebookApp.ip='*'                          # 配置任意ip皆可访问
c.NotebookApp.password = ''                   # 选填项，此为远程访问密码，将会在下文介绍
c.NotebookApp.open_browser = False            # 禁止自动打开浏览器
c.NotebookApp.port =6066                      # 指定远程访问端口
c.NotebookApp.notebook_dir = '/root/jupyter/' # 配置工作空间文件夹
c.NotebookApp.allow_remote_access = True      # 允许远程访问
```

### 2.设置密码

**Jupyter Notebook的访问密码实际上保存在同一目录下的两个文件中，且有优先级顺序：**

<div align=center><img src="/posts/2020/3-21/图 3-2-0.png" alt="" style="zoom:90%;" /></div>

#### (1) 低优先级：jupyter_notebook_config.py

**1）进入python环境中获取对应密码的密文**

```bash
$ py
```

```python
In [1]: from IPython.lib import passwd
In [2]: passwd()
Enter password:******                          # 输入密码
Verify password:******                         # 确认密码
Out[2]: 'sha1:0e422dfccef2:84cfbcbb3ef95872fb8e23be3999c123f862d856'     #输出密文
```

**2）将密钥填入至上文介绍的相关参数中**

```bash
c.NotebookApp.password = 'sha1:0e422dfccef2:84cfbcbb3ef95872fb8e23be3999c123f862d856'
```

#### (2) 高优先级：jupyter_notebook_config.json

**有三种方式配置此类密钥**

**1）忽视上文中的`c.NotebookApp.password`参数，根据提示在浏览器中进行密码初始化配置**

**2）通过命令行指定**

```bash
$jupyter notebook password
Enter password:******                          # 输入密码
Verify password:******                         # 确认密码
[NotebookPasswordApp] Wrote hashed password to /root/.jupyter/jupyter_notebook_config.json 
```

**3）手动填入**

向`jupyter_notebook_config.json`中填写/修改如下字段：

```bash
{
    NotebookApp:{								
    password:"******"                           # 此处填写经加密后的密文
	}											# 如何获取密文请见 低优先级配置方法
}
```

## 四、搭建 TensorBoard 远程访问

**因TensorBoard无法配置远程连接，直接远程访问会被拒绝，故配置端口转发[^2]**

**以SecureCRT8为示例**

### 1.获取TensorBoard连接端口

<div align=center><img src="/posts/2020/3-21/图 4-1-0.png" alt="" style="zoom:60%;" /></div>

### 2.配置远程连接

**1）点击`快速连接`，填入虚拟机ip地址并进行连接**

<div align=center><img src="/posts/2020/3-21/图 4-2-1.png" alt="" style="zoom:60%;" /></div>

### 3.配置端口转发

**1）右击已经配置好的`会话`，点击`属性`**

<div align=center><img src="/posts/2020/3-21/图 4-3-1.png" alt="" style="zoom:67%;" /></div>

**2）点击`端口转发`，然后点击`添加`**

<div align=center><img src="/posts/2020/3-21/图 4-3-2.png" alt="" style="zoom:67%;" /></div>

**3）配置`本地`以及`远程`所对应的端口**

<div align=center><img src="/posts/2020/3-21/图 4-3-3.png" alt="" style="zoom:67%;" /></div>

### 4.远程访问

**访问已经配置端口转发的本地端口即可**

<div align=center><img src="/posts/2020/3-21/图 4-4-0.png" alt="" style="zoom:67%;" /></div>

## 五、其他

**1.<关于我费了这么大劲把环境搭起来后才发现我的虚拟机和显卡跑不了 TF-gpu 这件事>**

泪目 QAQ，下次试试Docker或者购置一台支持虚化的显卡



[^1]:「NAT」，「Bridge」和「Host-Only」三种模式只要能ping通主机均可
[^2]:[https://baike.baidu.com/item/%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91/8327845?fr=aladdin](https://baike.baidu.com/item/端口转发/8327845?fr=aladdin)