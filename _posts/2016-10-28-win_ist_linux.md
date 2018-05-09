---
layout: post
title: Linux硬盘安装windows
category: other
---

以下方法适用win7/8/10

# 准备windows文件
找到windows的iso文件，解压到某**非C盘**的ntfs分区，**不一定要根目录，可以放在诸如win10的目录下**。

从这个名为win10的文件夹中把**bootmgr**文件和**boot**文件夹复制出来，放到分区的**根目录**下，在分区的**根目录**下新建一个文件夹名为**sources**, 然后从win10文件夹中的sources文件夹复制一个名为**boot.wim**的文件，把这个文件放到根分区下的sources文件夹内.

这样，根目录下的bootmgr文件和boot文件夹还有sources文件夹下的boot.wim文件，其实就构成了一个完整的winpe.

# 准备grub4dos
下载最新版的grub4dos，谷歌一搜即得。把它解压，从里面提取一个名为**grub.exe**的文件。把这个文件也放到存放win7安装文件的根目录下。

# 增加启动项
在文件**/etc/grub.d/40_custom**末尾增加如下内容
```
menuentry "Grub for Dos" {
insmod ntfs
set root=(hd0,9)
linux /grub.exe
}
```

这个(hd0,9)是指第一硬盘第九分区。硬盘是从0开始编号的，而分区则是从一开始编号的。主分区是一二三四，逻辑分区则是五六.....

把这个硬盘和分区编号换成你自己的。

运行如下命令应用修改
```
sudo update-grub
```

**若出错，可以在grub的启动界面修改，无需重启修改**

## 增加windows启动项
在存放win10安装程序的那个分区新建一个空白文件（其实也可以在任意分区），把它重命名为：menu.lst

在这个文件里面写入：
```
title win10
find --set-root /bootmgr
chainloader /bootmgr
boot
```

最后重启

# 参考
1. <http://blog.sina.com.cn/s/blog_49f914ab0100hcd7.html>
2. <http://dl.pconline.com.cn/download/90586-1.html>
3. <http://grub4dos.chenall.net/categories/downloads/>
