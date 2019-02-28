---
title: Google Colab 在线训练深度学习模型
date: 2019-02-28 21:05:49
categories:
- 工具
tags:
- Google Colab
- 深度学习
- 工具
---
# 简介
Google Colab是一个可以在线编译python的平台，用户程序运行在虚拟环境，并且提供一块GPU（Tesla K80）使用。
但是一个页面只能运行12个小时，超过后会重启虚拟环境，之前的程序之类的就都没了。
而且这块GPU是共享的，可能是采用轮循的方式吧。
一般是和云盘结合起来用，把数据放到云盘上，然后通过将云盘挂载到虚拟环境下，利用Colab在线运行。
云盘的大小是15GB，给钱的话可以升级容量。

# 将Colab与谷歌云盘结合！
第一步：授权访问
```
!apt-get install -y -qq software-properties-common python-software-properties module-init-tools
!add-apt-repository -y ppa:alessandro-strada/ppa 2>&1 > /dev/null
!apt-get update -qq 2>&1 > /dev/null
!apt-get -y install -qq google-drive-ocamlfuse fuse
from google.colab import auth
auth.authenticate_user()
from oauth2client.client import GoogleCredentials
creds = GoogleCredentials.get_application_default()
import getpass
!google-drive-ocamlfuse -headless -id={creds.client_id} -secret={creds.client_secret} < /dev/null 2>&1 | grep URL
vcode = getpass.getpass()
!echo {vcode} | google-drive-ocamlfuse -headless -id={creds.client_id} -secret={creds.client_secret}
```

第二步：挂载云盘
将谷歌云盘看作是虚拟机中的一个硬盘挂载
```
!mkdir -p drive
!google-drive-ocamlfuse drive
```

第三步：更换目录
```
import os
os.chdir('drive')
!ls
```
Colab里面不能用`cd`更换目录，只能用上述方法。
`drive`是我们将云盘挂载到Colab的盘
进入到`drive`中就可以看到云盘中的文件啦！

# 模型的断线保护
因为这个12小时强制重启的操作，我们必须要时刻保存训练的中间过程，这样在我们下一次重新训练的时候可以接着上一次成果继续。

Keras中的`ModelCheckpoint`函数就可以实现在每个`epoch`训练完后保存模型
```python
checkPoint = keras.callbacks.ModelCheckpoint(
    model_name, monitor='val_loss', verbose=0, save_best_only=True, mode='auto', period=1)
```
具体的参数解释可以去看文档
这里的保存是`model.save()`

这样下一次重新训练的时候只要加载上一次训练好的模型就可以啦！


---

参考文献：
[官方新手教程](https://medium.com/deep-learning-turkey/google-colab-free-gpu-tutorial-e113627b9f5d)
[参考文章：云盘挂载](https://www.jianshu.com/p/000d2a9d36a0)
[参考文章：断线保护](https://www.cnblogs.com/baiting/p/8420637.html)