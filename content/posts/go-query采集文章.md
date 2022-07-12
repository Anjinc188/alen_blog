---
title: "Go Query采集文章"
date: 2020-08-04
categories: ["后端"]
keywords: ["Go","Go-Query"]
tags: ["Go","Go-Query","Gin"]
draft: true

---

>相信很多小伙伴对爬虫有着很大的兴趣,我们今天来说下go语言的爬虫利器`goquery`，语法有`jquery`类似，我有一篇使用php-QueryList,使用与goquery类似，感兴趣的小伙伴可以去看看

本文前部分参考 飞雪无痕 的 《golang goquery selector(选择器) 示例大全》

## 安装

```go
go get github.com/PuerkitoBio/goquery
```

#### 基于HTML Element 元素的选择器

```go
func main() {
	html := `<body>
				<div>这是一个div里面的内容</div>
				<div>这是一个div2里面的内容</div>
				<span>这是一个span里面的内容</span>

			</body>
			`
	dom,err:=goquery.NewDocumentFromReader(strings.NewReader(html))
	if err!=nil{
		log.Fatalln(err)
	}
	dom.Find("div").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Text())
	})
}

```

以上示例，可以把`div`元素筛选出来，而`body`,`span`并不会被筛选

---

#### ID 选择器

这个是使用频次最多的，类似于上面的例子，有两个`div`元素，其实我们只需要其中的一个，那么我们只需要给这个标记一个唯一的`id`即可，这样我们就可以使用`id`选择器，精确定位了。

```go
func main() {
	html := `<body>
				<div>这是一个div里面的内容</div>
				<div>这是一个div2里面的内容</div>
				<span>这是一个span里面的内容</span>
			</body>	`
	dom,err:=goquery.NewDocumentFromReader(strings.NewReader(html))
	if err!=nil{
		log.Fatalln(err)
	}
	dom.Find("#div1").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Text())
	})
}

```

---

## 实战

下面我们已实战为例，爬取微信公众号文章的内容，直接上代码

```go
result, err := http.Get(urls)
	if err != nil {
		logger.Error(err)
	}
	dom, err := goquery.NewDocumentFromReader(result.Body)
	Article := Article{}

	dom.Find("#article").Each(func(i int, s *goquery.Selection) {
		Article.Title = s.Find(".article-title").Text()
		Article.Content, _ = s.Find(".article-content").Html()
	})

	dom.Find("head").Each(func(i int, s *goquery.Selection) {
		Article.Description, _ = s.Find(`meta[name="description"]`).Attr("content") //文章描述
	})

	doc, err := goquery.NewDocumentFromReader(strings.NewReader(Article.Content))
	doc.Find("img").Each(func(i int, s *goquery.Selection) {
		Src, _ := s.Attr("src")
        //qiniuFetch 处理微信图片防盗链
		imageUrl := qiniuFetch(Src)
		if imageUrl != "" {
			s.SetAttr("src", imageUrl)
		}
	})

	content, _ := doc.Html()
	content = strings.Replace(content, "<html><head></head><body>", "", -1)
	content = strings.Replace(content, "</body></html>", "", -1)

```

```go
func qiniuFetch(imgSrc string) (imageUrl string) {
	var (
		AccessKey = "******"  //ACCESS_KEY
		SecretKey = "******"  //SECRET_KEY
		domain    = "******"  //七牛域名
		bucket    = "******"  //存储空间名称
	)

	mac := qbox.NewMac(AccessKey, SecretKey)
	cfg := storage.Config{
		Zone:          &storage.ZoneHuanan,
		UseHTTPS:      false,
		UseCdnDomains: true,
	}

	bucketManager := storage.NewBucketManager(mac, &cfg)
	fetchRet, err := bucketManager.FetchWithoutKey(imgSrc, bucket)

	if err != nil {
		logger.Error("fetch error", err)
		return ""
	} else {
		return domain + fetchRet.Key
	}
}

```

剩余部分可自行根据需求存入数据库中
