---
title: "使用Logstash把mysql数据同步到elasticsearch"
date: 2020-07-04
categories: ["数据处理"]
keywords: ["Logstash","elasticsearch"]
tags: ["Logstash","elasticsearch"]
draft: true
---

## **首先 我们要知道elasticsearch是什么**

>elasticsearch简写es，es是一个高扩展、开源的全文检索和分析引擎，它可以准实时地快速存储、搜索、分析海量的数据

### **什么是全文检索**

全文检索是指计算机索引程序通过扫描文章中的每一个词，对每一个词建立一个索引，指明该词在文章中出现的次数和位置，当用户查询时，检索程序就根据事先建立的索引进行查找，并将查找的结果反馈给用户的检索方式。这个过程类似于通过字典中的检索字表查字的过程。全文搜索搜索引擎数据库中的数据。

---

### 准备工作

- 下面我们通过Logstash把mysql里面的数据同步到elasticsearch,关于mysql安装就不用说了

- 首先要安装elasticsearch [官方下载链接](https://www.elastic.co/cn/downloads/elasticsearch)

- logstash [官方下载链接](https://www.elastic.co/cn/downloads/logstash)

- kibana [官方下载链接](https://www.elastic.co/cn/webinars/getting-started-kibana?elektra=products-kibana&storm=hero&rogue=watch-video)

- 建议挂代理 如果不挂代理会很慢，在我[另一篇博文](http://blgo.ximi-edu.com/index.php/2020/03/16/first/)上有墙内资源 大家可去download

  环境搭建好下面开始

---

#### 1.安装logstash-input-jdbc

```sh
安装此插件以及依赖需要在ruby环境下，ruby默认的镜像在国外，所以很慢，需要手动将镜像网站https://rubygems.org/替换为国内的https://gems.ruby-china.com/
```

##### **1.11 安装ruby**

- 我这里是windows系统 [ruby官方下载链接](https://rubyinstaller.org/downloads/)
- 下载安装完之后 验证 ruby -v 返回版本既安装成功

---

##### 1.12安装gem

Gem是一个管理Ruby库和程序的标准包，MacOS系统自带gem环境(我这里是Windows，所以要安装一下)，可以用gem -v查看版本，gems.ruby-china建议gem版本最好>2.6.x，如果版本较低，可以用 gem update –system更新

```sh
gem -v
gem update --system
```

gem sources -l命令的作用是查看当前gem的源，在替换成功后，应该显示如下：

```sh
*** CURRENT SOURCES ***
https://gems.ruby-china.com/
```

确保只有 [http://gems.ruby-china.com](https://link.zhihu.com/?target=http%3A//gems.ruby-china.com)，即替换成功

---

##### 1.13**使用 Bundler 的[Gem 源代码镜像命令](https://link.zhihu.com/?target=http%3A//bundler.io/v1.5/bundle_config.html%23gem-source-mirrors)或修改Gemfile**

```sh
bundle config mirror.https://rubygems.org https://gems.ruby-china.com
```

在logstash-7.6.1(其它版本应该一致)根目录下打开Gemfile与Gemfile.lock两个文件 然后更改如下：

- Gemfile
- 修改第1行出现的source，将source “[https://rubygems.org](https://rubygems.org/)”，改为source “https://gems.ruby-china.com/"

Gemfile.lock

- 修改GEM下的remote，将remote: [https://rubygems.org](https://rubygems.org/)，改为remote: https://gems.ruby-china.com/

---

##### 1.14 **安装logstash-input-jdbc插件**

- 完成前面的操作之后我们就可以来安装logstash-input-jdbc了
- 打开logstash文件夹下的bin目录
- 打开终端输入

```sh
logstash-plugin install logstash-input-jdbc --回车进行安装
```

- 安装成功之后就可以开始导入配置了 前提装备还需要mysql-connector-java这个jar包
- [官方下载地址](https://mvnrepository.com/artifact/mysql/mysql-connector-java)
- 以上我们的第一步操作就完成了

#### **2.配置logstash的conf文件**

```sh
input {
stdin {
}
jdbc {
       jdbc_connection_string => "jdbc:mysql://localhost:3306/yt_rjjy?"
       jdbc_user => "root"
       jdbc_password => ""
       jdbc_driver_library => "D:\Elastic\logstash-7.6.1\mysql-connector-java-8.0.16.jar"
       jdbc_driver_class => "com.mysql.jdbc.Driver"
       jdbc_paging_enabled => "true"
       jdbc_page_size => "50000"
       jdbc_default_timezone => "UTC"
       statement => "SELECT * FROM `card`"
       schedule => "* * * * *"
   }
}
output {
elasticsearch {
       # 配置ES集群地址
         hosts => ["127.0.0.1:9200"]
         index => "card"
         document_id => "%{id}"

 }
stdout {

  }
}

```

配置完之后我们去bin目录下 执行cmd命令

```sh
logstash -f D:\Elastic\logstash-7.6.1\config\logstash-mysql.conf 执行无错误就成功了 如果你用的是mysql的最新的驱动8.多的版本会出现配置错误
```

具体解决方法：https://blog.csdn.net/u012976879/article/details/85261036

原文出处：https://zhuanlan.zhihu.com/p/46749126

