---
title: "php QueryList采集微信公众号文章"
date: 2019-12-04
categories: ["后端","爬虫"]
keywords: ["php","queryList"]
tags: ["php"]
draft: true
---

## QueryList 是什么

> QueryList是一套用于内容采集的PHP工具，它使用更加现代化的开发思想，语法简洁、优雅，可扩展性强。相比传统的使用晦涩的正则表达式来做采集，QueryList使用了更加强大而优雅的**CSS选择器**来做采集，大大降低了PHP做采集的门槛，同时也让采集代码易读易维护，让你从此告别晦涩难懂且不易维护的正则表达式😀。

### QueryList 提供的一整套内容采集解决方案

- DOM内容选择：CSS选择器
- HTTP客户端：GuzzleHTTP
- 内容过滤：CSS选择器
- 额外功能：丰富的扩展插件

>具体详细可以去官方文档查看 [QueryList官方文档](https://querylist.cc/docs/guide/v4/overview)

## 安装

`QueryList`目前有2个支持的版本`V3`和`V4`，安装之前你需要根据实际环境来选择,它们的区别如下:

- V3
  - PHP版本要求PHP5.3以上；
  - 只有一个主文件，可直接引入无需使用Composer安装，使用便捷；
  - 只有一个主要的API，学习简单；
  - 支持V4版本的大多数功能特性
- V4
  - PHP版本要求PHP7.1以上;
  - 更加现代化的设计思想，文件结构复杂，需要使用Composer安装;
  - 更加丰富的富有表现力的API,功能更加强大;
  - 完全模块化的设计，更加强大的可扩展性;

**总的来说，如果条件允许请尽量使用最新版本。**

## [环境要求](https://querylist.cc/docs/guide/v4/installation#environment)

```bash
PHP >= 7.1
```

>QueryList 4.0 要求PHP版本7.1，如果你的PHP版本还停留在PHP5，或者不会使用Composer,你可以选择使用QueryList3,QueryList3支持php5.3以及手动安装。

## 开始

- 下面以thinkphp为例

- 在项目根目录执行composer命令安装QueryList

```bash
composer require jaeger/querylist
```

**具体代码:**

```php

        // origin_url为文章链接
        if (!empty($data['origin_url'])) {
            return false;
        }

        //验证文章是否存在
        $origin_url = self::articleDetail(['origin_url' => $url]);
        if ($origin_url) {
            $this->error = '该文章已存在';
            return false;
        }

        if (empty($url)) {
            $this->error = '请输入url';
            return false;
        }

        $_host = parse_url($url, PHP_URL_HOST);  //获取主机名
        //不是来自微信文章
        if ($_host !== 'mp.weixin.qq.com') {
            $this->error = '不是来自微信文章';
            return false;
        }


        $rules = array(   //设置QueryList的解析规则
            'thumbnail' => array('meta[property=og:image]', 'content'),//文章封面图
            "description" => array('meta[name=description]', 'content'),//获取文章描述
            'title' => array('#activity-name', 'text'),  //文章标题
            'content' => array('#js_content', 'html'),  //文章内容
        );

        $result = QueryList::get($url)->rules($rules)->query()->getData();
        $_res = $result[0];  //获取解析结果

        if (empty($_res)) { //解析失败
            $this->error = '文章爬取失败！';
            return false;
        }

        //aid代表是否添加/编辑
        $_res['aid'] = $aid;
        //qiniuFetch()这里处理了图片防盗链 因为微信图片有防盗链，需要把图片放去七牛云处理
        $_res['thumbnail'] = $fetch->qiniuFetch($_res['thumbnail']);
        //正则替换内容中的图片链接
        $pattern = '/<img([^>]*)data-src\s*=\s*([\' "])([\s\S]*?)([^>]*)/';
        $_res['content'] = preg_replace($pattern, '<img$1src=$2' . '$3$4', $_res['content']);
        $_res['category_id'] = $data['category_id'];    //文章分类ID
        $_res['origin_url'] = $data['origin_url'];      //文章链接
        //处理文章内部图片
        $doc = \phpQuery::newDocumentHTML($_res['content']);
        $imgSrc = pq($doc)->find('img');

        foreach ($imgSrc as $item) {
            $src = pq($item)->attr('src');
            //图片传入qiniuFetch处理
            $qiniuImg = $fetch->qiniuFetch($src);

            pq($item)->attr('src', $qiniuImg);
            $_res['content'] = str_replace($src, $qiniuImg, $_res['content']);
        }
        return $_res;

```

```php
    /**
     * 七牛云Fetch配置方法
     * @param $srcUrl
     * @return bool|string
     */
    public function qiniuFetch($url)
    {
        //七牛云配置
        $config = $this->engine;
		
        //写入32位随机数
        $str = "QWERTYUIOPASDFGHJKLZXCVBNM1234567890qwertyuiopasdfghjklzxcvbnm";
        $key = substr(str_shuffle($str), mt_rand(0, strlen($str) - 11), 32);

        $iovip = 'iovip-z2.qbox.me';
        $encodedURL = $this->base64_urlSafeEncode($url);
        $bucket = empty($key) ? $config['bucket'] : $config['bucket'] . ':' . $key;
        $encodedEntryURI = $this->base64_urlSafeEncode($bucket);
        $url = '/fetch/' . $encodedURL . '/to/' . $encodedEntryURI;
        $sign = hash_hmac('sha1', $url . "\n", $config['secret_key'], true);
        $token = $config['access_key'] . ':' . $this->base64_urlSafeEncode($sign);
        $header = array('Host: ' . $iovip, 'Content-Type:application/x-www-form-urlencoded', 'Authorization: QBox ' . $token);
        $curl = curl_init();
        curl_setopt($curl, CURLOPT_URL, 'http://' . $iovip . $url);
        curl_setopt($curl, CURLOPT_HTTPHEADER, $header);
        curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
        curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, false);
        curl_setopt($curl, CURLOPT_SSL_VERIFYHOST, false);
        curl_setopt($curl, CURLOPT_POSTFIELDS, "");
        $result = json_decode(curl_exec($curl), true);
        curl_close($curl);
        return !empty($result['key']) ? $config['domain'] . '/' . $result['key'] : false;
    }

    /**
     * base64处理
     * @param $data
     * @return mixed
     */
    public function base64_urlSafeEncode($data)
    {
        $find = array('+', '/');
        $replace = array('-', '_');
        return str_replace($find, $replace, base64_encode($data));
    }

```

以上为queryList 个人使用过程

