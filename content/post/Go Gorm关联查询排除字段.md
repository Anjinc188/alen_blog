---
title: "Go Gorm关联查询排除字段"
date: 2020-10-08
categories: ["后端"]
keywords: ["Go","Gorm"]
tags: ["Go","Gorm"]
draft: true
---

### 本章讲解关于gorm模型后如何筛选按需字段

##### 安装gorm

```sh
go get -u github.com/jinzhu/gorm
```

### 一对多使用：

```go
//日记模型
type Diary struct {
    Model
    Uid       uint32         `json:"uid"`                                                           //用户ID
    Content   string         `json:"content"`                                                       //日记内容
    DiaryImg  []DiaryImg     `gorm:"ForeignKey:diary_id;AssociationForeignKey:id" json:"diary_img"` //日记图片
    DiaryLike int            `json:"diary_like"`                                                    //点赞
    Collect   int            `json:"collect"`                                                       //收藏
}

//日记图片模型
type DiaryImg struct {
    Model
    DiaryId   int    `json:"diary_id"`   //日记ID
    DiaryUrls string `json:"diary_urls"` //日记图片路径
}       
```

### 外键：

- 为了定义一对多关系， 外键是必须存在的，默认外键的名字是所有者类型的名字加上它的主键。 就像上面的例子，为了定义一个属于 Diary的模型，外键就应该id

---

### **重点：**

```go
`gorm:"ForeignKey:diary_id;AssociationForeignKey:id" json:"diary_img"`
```

关联成功之后可以通过 Related 找到 has many 关联关系`

```go
db.Model(&diary).Related('DiaryImg')
```

那么问题来了，我们如何只需要查询指定的字段呢。我们可以这样使用，一步到位

```go
Preload("DiaryImg", func(db *gorm.DB) *gorm.DB {
         return db.Select("diary_urls")
}).
```

这样就解决了我们使用gorm查询按需字段的问题了
