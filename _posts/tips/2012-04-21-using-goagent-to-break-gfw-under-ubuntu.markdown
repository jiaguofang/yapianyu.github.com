---
layout: post
title: Ubuntu下使用goagent翻墙
category: tips
tags: ubuntu goagent gfw
---

在[Xan Peng](http://xanpeng.github.com)的强烈推荐下，终于用上了goagent，在追求真理的道路上又向前迈出了一步。

###准备工作###

1. 在GAE上架设服务器
2. 在用户电脑上的运行客户端
3. 安装代理切换工具，如SwitchSharp
4. 配置代理切换规则，即规定哪些URL使用goagent代理

###Step by step###
1. 在[Google App Engine](https://appengine.google.com/)上Create Application，并记住"app id"
2. 下载并解压[goagent稳定版](http://code.google.com/p/goagent/)和[GAE SDK For Python](https://developers.google.com/appengine/downloads)
3. 修改goagent/local/proxy.ini中\[gae\]段落下的appid字段为"app id"
4. 修改goagent/server/python/app.yaml中的application字段为"app id"
5. 用GAE上传服务器。PS：之前运行`python uploader.zip`一直不能成功，换成这种方法就OK了
        python google_appengine/appcfg.py update goagent/server/python/
6. 服务器上传成功之后，再运行客户端，此时即可使用代理127.0.0.1:8087 
        python goagent/local/proxy.py
7. 为Chrome安装[SwitchySharp](https://chrome.google.com/webstore/detail/dpplabbmogkhghncfbfdeeokoefdjegm?utm_source=chrome-ntp-icon)插件，并在Import/Export栏目下导入[SwitchSharp配置](http://goagent.googlecode.com/files/SwitchyOptions.bak)
8. 为Chrome安装goagent证书goagent/local/CA.crt，安装步骤为Chrome->Settings->HTTPS/SSL->Manage Certificates->Authorities->Import...
9. 开机自动启动goagent客户端，在~/.profile里添加`python /home/yapianyu/work/Installation/goagent/local/proxy.py > /dev/null 2>&1 &` 
