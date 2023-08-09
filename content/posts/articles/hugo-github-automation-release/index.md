---
title: "[Tech]基于Github的Hugo博客自动同步"
date: 2023-08-09T04:58:06+08:00
toc: true
---
## 前言

花了一晚上研究了一下如何把博客自动部署到服务器，作为记录。

总结一下需要的操作，最终实现的路径为：本地 -> GitHub -> 服务器，需要依赖git和WebHooks。

值得一提的是，默认是将public文件夹放入.gitignore目录，因此需要手动放出来，才能进行同步。

## 同步流程

首先将本地的博客内容上传到GitHub之中，实现本地到云端的同步。
随后在服务器中将Github的仓库内容clone到相应文件夹之中。以文件夹`www/wwwroot/Alkdis-site`为例。
之后需要实现服务器到Github的自动同步，这需要使用WebHooks功能。
这步需要两个操作，分别是启动WebHooks和写服务器自动化脚本。

## WebHooks

这个操作在仓库的 Settings/WebHooks之中。
`Payload URL`为`http://Setver Url/www/wwwroot/Alkdis-site`
`Content type`为`application/x-www-form-urlenconded`
最好选择`Just the push event`。

WebHooks设置完毕，接下来需要写自动同步脚本。

## 脚本

在`/www/wwwroot/Alkdis-site/`目录下，新建`sync.sh`，内容为：

~~~sh
#!/bin/bash
# 进入到代码仓库目录
cd /www/wwwroot/Alkdis-Site/
# 拉取最新的代码
git pull origin main

# 执行部署操作，将所有文件复制到指定位置
cp -R * /www/wwwroot/Alkdis-Site/
~~~

随后编辑crontab，实现自动调动。

~~~sh
crontab -e
~~~

如果是第一次使用，需要选择熟悉的编辑器。随后输入

~~~sh
* * * * * /bin/bash /www/wwwroot/Alkdis-Site/sync.sh
* * * * * sleep 1; /bin/bash /www/wwwroot/Alkdis-Site/sync.sh
* * * * * sleep 2; /bin/bash /www/wwwroot/Alkdis-Site/sync.sh
* * * * * sleep 3; /bin/bash /www/wwwroot/Alkdis-Site/sync.sh
~~~

这实现了每秒自动波动，如果想要调整时间，可以自行调整，调整规则如下：

~~~sh
* * * * *
| | | | |
| | | | +----- 日 (0 - 31)
| | | +------- 月 (1 - 12)
| | +--------- 星期 (0 - 7，其中 0 和 7 都表示周日)
| +----------- 小时 (0 - 23)
+------------- 分钟 (0 - 59)
~~~

脚本设置完毕。
至此，自动化同步进程设置完毕，之后只需要将博客写完了pull到Github，就会自动同步到服务器。

## 试错

在发现WebHooks之前，我寻找了很多方式，包括Github Actions，他是在仓库的.github/workflow/*.yaml 文件之中进行自动流部署，但是最终的分发都是基于Github Pages，我的博客并不是基于此而是直接部署在服务器，因此并不需要系列操作。
