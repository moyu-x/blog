---
title: Ubuntu移除多余的内核
updated: 2017-3-22 21:56:37
date: 2016-2-29 21:47:53
tags:
  - Linux
  - 内核
categories:
  - Linux
---

# Ubuntu移除多余的内核

在使用Ubuntu的时候，发现内核更新的频率比较高，所以需要删除一些内核来节省空间:

1. 查询当前系统所使用的内核`uname -r`
2. 查询系统中安装的内核`dpkg --get-selections|grep linux`
3. 删除内核：

```bash
sudo apt-get remove linux-image-*.*.*-**（*号用你想删除的实际情况改写）
sudo apt-get remove linux-headers-*.*.*-**（*号用你想删除的实际情况改写）
```

4. 再次执行`dpkg --get-selections|grep linux`查看内核是否都删除干净了,没干净继续删除,有的内核后面会显示是deinstall 那需要通过`dpkg --get-selections | grep deinstall | sed 's/deinstall/\lpurge/' | sudo dpkg --set-selections; sudo dpkg -Pa`来进行删除
