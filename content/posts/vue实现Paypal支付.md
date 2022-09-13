---
title: "vue实现PayPal"
date: 2022-05-08
categories: ["前端"]
keywords: ["vue"]
tags: ["vue","支付"]
draft: true
---

### PayPal我使用的是官方封装继承的集成

- PayPal官网有一个内部封装得流程，用户点击按钮触发创建订单事件，然后监听用户成功的回调，把订单ID传到服务端，再服务端进行校验。

- paypal官方文档：https://developer.paypal.com/sdk/js/reference/

关于PayPal账号注册流程我这里就不举例了，直接开始讲解代码，首先需要准备的是：

- ClientId：XXX3AX在PayPal后台获取。

- Secret：XXX3AXJnfUqNX在PayPal后台获取。

### 代码实战：

- 笔者这里使用的是vue3.0框架

- 安装paypal.js 开源库
  - ```JavaScript
    npm install @paypal/paypal-js
    // github地址：https://github.com/paypal/paypal-js
    ```

- 整体代码示例：

```JavaScript
<template>
    <div>
        <div class="paypal-box"><div id="paypal-button-container"></div></div>
    </div>
</template>

<script>
import { reactive, toRefs, onMounted } from 'vue'
import { loadScript } from '@paypal/paypal-js'

export default {
    setup(props) {
        const state = reactive({ paypal: {} })

        onMounted(() => {
            createdPaypal()
        })

        const createdPaypal = () => {
            loadScript({
                'client-id': 'xxxx你的client-id',
                'data-client-token': 'xxxx你的data-client-token'
            })
                .then(paypal => {
                    paypal
                        .Buttons({
                            style: {
                                layout: 'vertical',
                                color: 'gold',
                                shape: 'pill',
                                label: 'buynow'
                            },
                            // 初始化
                            onInit: function (data, actions) {
                                console.log(data)
                            },
                            // 创建订单
                            createOrder: function (data, actions) {
                                const order = actions.order.create({
                                    purchase_units: [
                                        {
                                            amount: {
                                                value: 100,
                                                currency_code: 'USD'
                                            },
                                            // 这个ID可以用做数据库里的uid 识别订单用
                                            reference_id: '2022xxxxxxxxxxx04xxxx'
                                        }
                                    ]
                                })

                                // 这里有一个paypal生成的orderId
                                // console.log(order.value)

                                return order
                            },
                            // 支付成功
                            onApprove: async (data, actions) => {
                                console.log('用户支付成功')
                                console.log('--->data', JSON.stringify(data))
                                console.log('--->actions', JSON.stringify(actions))
                            },
                            // 取消支付
                            onCancel: data => {
                                console.log(data, '支付取消')
                            }
                        })
                        .render('#paypal-button-container')
                })
                .catch(err => {
                    console.error('failed to load the PayPal JS SDK script', err)
                })
        }

        return { ...toRefs(state) }
    }
}
</script>

<style lang="scss">
.paypal-box {
    width: 80%;
    margin: 0 auto;
}
</style>
```

- 业务逻辑一般是onApprove支付成功后把订单ID传输给服务端，然后服务端去进行校验。

- 像paypal .Buttons.style 这些参数可以在paypal官网文档中查看说明。

- data-client-token这个参数是请求服务端传输过来的。
