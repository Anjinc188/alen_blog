---
title: "Vue3"
date: 2020-10-04
categories: ["前端"]
keywords: ["vue","javaScript"]
tags: ["vue","javaScript"]
draft: true
---

# Vue3.0 可以完全支持Vue2.0, 升级以后不需要更改关于Vue的代码

### vue3.0创建项目

```javascript
-- 使用vite
    npm init vite-app  你的项目名
    cd 项目名
    npm run dev
```

---

### 当然也可以使用npm

```javascript
    npm install -g @vue/cli
    vue create 你的项目名
    cd 项目名
    vue add vue-next 
    npm run serve
```

---

#### **变化一：main.js**

```javascript
import { createApp } from 'vue';
import App from './App.vue';
import router from './router';
import store from './store';
import './App.scss'

const app = createApp(App)
app.use(router).use(store).mount('#app');
```

---

#### 变化二：CompositionApi

- **setup函数**

```javascript
<script>
  import { ref, reactive } from 'vue'
  export default {
    setup(props, context) {
        // props父组件传的值
        // context是对参数的集合
      return {
       ....
      }
    },
  }
</script>
```

**数据声明**

- vue2.0 里数据都是在data()里声明， 而在3.0里也取消了data(), 引用了Reactive(), ref()
- ref() 声明单一基础数据类型时使用例如

```javascript
<template>
  <div>{{count}}</div>
</template>
<script>
 import { setup， ref } from 'vue'
 export default {
    setup(props, context) {
        let count = ref(0)
        console.log(count.value) //0 ，  内部获取时需要用.value来获取
     return {
         count  // ref返回的是一个响应式的对象
     }
    }
 }
</script>
<style>
</style>

```

##### Reactive() 声明单一对象时使用

```javascript
<template>
 <div>{{data1}}</div>
</template>
<script>
import { setup, Reactive } from 'vue'
export default {
   setup(props, context) {
        const state= reacttive({
        name: 'Vue3.0',
        data: 'hello wolrd'
    })
    
    return {
        state
    }
   }
}
</script>
<style>
</style>
```

**watchEffect() 监听props**

```javascript
// 父组件.vue
<template>
   <ZiComp val:'1'/>
</template>
<script>

import ziComp from './ziComp.vue'
export default {
  components： {
      ziComp
  }, //components方法也是和2.0相同
  setup(props, context) {
      
  }
}
</script>
-------------------------------------- 
//ziComp.vue
<template>
 <div>{{obj.data1}}</div>
</template>
<script>
import { Reactive } from 'vue'
export default {
  props: ['val'], //这里获取props和 2.0 一样,也可以用对象形式
  setup(props, context) {
     watchEffect(() => {  //首次和props改变才会执行这里面的代码
         console.log(props.val)
     })
  }
}
</script>
<style>
</style>

```

本章就讲先到这里吧,暂时了解到这么多
