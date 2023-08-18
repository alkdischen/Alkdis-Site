---
title: "[DataSet] ImageNet-1k处理"
date: 2023-08-06T11:45:42Z
toc: true
categories: ["CV"]
tags: ["DataSet"]
---

## 前言

由于每次在云服务器上处理数据都需要进行数据集下载与处理，因此写一篇post来记录ImageNet的处理过程。

ImageNet的重要性不遑多言，官方下载网站为：https://image-net.org/download.php，需要提前注册号账号，且下载速度较慢，建议使用云盘下载。

这里列出百度云下载链接：

~~~
https://pan.baidu.com/s/1Nhd7YB-xBXPYj7y9cNOedQ
提取码：ufxe
~~~

## 数据集类别

目前主流数据集又ImageNet-1k和ImageNet-22k，数字代表类别数目不同。

ImageNet有不同年份的数据集，一般使用ILSVRC2012版本，为2012年的ILSVR大赛数据集。

观察下载数据，可以发现分为三大类：

1. bbox
2. devkit
3. img

如果只是用于日常训练，只需要考虑img数据集，本文也只会探讨img数据集的处理。

观察img数据，发现分为四个文件，分别为：

1. train
2. ~~train_t3~~
3. val
4. test

它们分别是训练集，验证集与测试集。特别的，train_t3为新增的训练任务，具体内容可以去官网查看，一般不用考虑使用。

因此，我们最终需要使用的文件名称为

1. ILSVRC2012_img_train.tar
2. ILSVRC2012_img_val.tar
3. ILSVRC2012_img_test.tar

下载到服务器即可，建议存放到单独的文件夹，本文以文件夹 `/ImageNet1k/ILSVRC2012/`作为根目录为例。

## 数据集处理

tar文件的解压很简单，使用指令

~~~
tar -xvf 文件名
~~~

即可解压到当前目录，因此建议分别解压到`/ImageNet1k/ILSVRC2012/train`，`/ImageNet1k/ILSVRC2012/val`和`/ImageNet1k/ILSVRC2012/test`目录即可，将文件mv到相应目录，然后解压即可。

值得一提的是，val和test文件全都是图片，只需要正常解压就行，但是train不是，内部是1000个压缩包，每个压缩包代表一个类别的图片压缩集合，需要特别处理。

## 训练集处理

进入到训练集的目录`/ImageNet1k/ILSVRC2012/train`，确定好压缩包数目为1000个，随后进行批量解压缩，指令为

~~~
for file in /ImageNet1k/ILSVRC2012/train*.tar; do
  dir=$(basename "$file" .tar)
  mkdir "/ImageNet1k/ILSVRC2012/train$dir"
  tar -xvf "$file" -C "ImageNet1k/ILSVRC2012/train/$dir"
done
~~~

随后会将所有的压缩包解压到单独的以自身文件名称命名的文件夹之中。

如果空间不充足，可以边解压边删除，增加一句`rm "$file"`即可：

~~~for file in /ImageNet1k/ILSVRC2012/train*.tar; do
for file in /ImageNet1k/ILSVRC2012/train*.tar; do  
  dir=$(basename "$file" .tar)
  mkdir "/ImageNet1k/ILSVRC2012/train$dir"
  tar -xvf "$file" -C "ImageNet1k/ILSVRC2012/train/$dir"
  rm "$file"
done
~~~

至此ImageNet1k数据集的处理结束。

## 检验

检验数据解压数据量是非常重要的步骤，在此罗列常用的检验操作。

检验当前目录下文件夹数目：

~~~
ls -l | grep '^d' | wc -l
~~~

检验当前目录下压缩文件数目：

~~~
ls -l | grep '\.zip$\|\.rar$\|\.tar$\|\.gz$\|\.bz2$\|\.7z$\|\.xz$' | wc -l
~~~

检验当前文件下图片数目：

~~~
ls -l | grep -i '\.JPEG$\|\.jpg$\|\.jpeg$\|\.png$\|\.gif$\|\.bmp$' | wc -l
~~~

对于`ImageNet1k/ILSVRC2012`而言

`/ImageNet1k/ILSVRC2012/train`有1000个类别，存放在1000个文件夹之内，每个文件夹内图片数目不固定。

`/ImageNet1k/ILSVRC2012/val`有5000张图片。

`/ImageNet1k/ILSVRC2012/train`有10000张图片。

以供检查校验。

## 其他

发现了val的转换脚本，如果直接解压val，得到了5000张图片，在val文件夹使用脚本可以把图片分到1000个文件夹中，实现和train同等结构的转换。

脚本下载链接：[val2tr.sh](./val2tr.sh)