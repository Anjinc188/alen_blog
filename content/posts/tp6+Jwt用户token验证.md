---
title: "Tp6+Jwt用户token验证"
date: 2020-06-04
categories: ["后端"]
keywords: ["php","thinkphp6.0"]
tags: ["php","thinkphp6.0"]
draft: true
---

# JSON Web Token是什么？

转载：https://www.liqingbo.cn/docs/jwt/

>JSON Web Token (JWT)是一个开放标准(RFC 7519)，它定义了一种紧凑的、自包含的方式，用于作为JSON对象在各方之间安全地传输信息。该信息可以被验证和信任，因为它是数字签名的。

# JWT组成

- JWT由三个部分组成：header.payload.signature

```sh
{
  "alg": "HS256",
  "typ": "JWT"
}
```

对应`base64UrlEncode`编码为：`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9`

说明：该字段为json格式。alg字段指定了生成signature的算法，默认值为 HS256，typ默认值为JWT

## payload部分：

```sh
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022
}
```

对应`base64UrlEncode`编码为：`eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ`

说明：该字段为json格式，表明用户身份的数据，可以自己自定义字段，很灵活。sub 面向的用户，name 姓名 ,iat 签发时间。例如可自定义示例如下：

```sh
{
    "iss": "admin",          //该JWT的签发者
    "iat": 1535967430,        //签发时间
    "exp": 1535974630,        //过期时间
    "nbf": 1535967430,         //该时间之前不接收处理该Token
    "sub": "www.admin.com",   //面向的用户
    "jti": "9f10e796726e332cec401c569969e13e"   //该Token唯一标识
}
```

## signature部分：

```sh
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  123456
)
```

对应的签名为：`keH6T3x1z7mmhKL1T3r9sQdAxxdzB6siemGMr_6ZOwU`

最终得到的JWT的Token为(header.payload.signature)：

```sh
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.keH6T3x1z7mmhKL1T3r9sQdAxxdzB6siemGMr_6ZOwU
```

说明：对header和payload进行base64UrlEncode编码后进行拼接。通过key（这里是123456）进行HS256算法签名。

## 有效载荷

有效载荷部分，是JWT的主体内容部分，也是一个JSON对象，包含需要传递的数据。 JWT指定七个默认字段供选择。

-  iss：发行人
-  exp：到期时间
- sub：主题
- aud：用户
-  nbf：在此之前不可用
- iat：发布时间
-  jti：JWT ID用于标识该JWT

除以上默认字段外，我们还可以自定义私有字段，如下例：

```sh
{

   “sub”: “1234567890”,

   “name”: “chongchong”,

   “admin”: true

} 
```

请注意，默认情况下JWT是未加密的，任何人都可以解读其内容，因此不要构建隐私信息字段，存放保密信息，以防止信息泄露。

JSON对象也使用Base64 URL算法转换为字符串保存。

## 签名哈希

签名哈希部分是对上面两部分数据签名，通过指定的算法生成哈希，以确保数据不会被篡改。

首先，需要指定一个密码（secret）。该密码仅仅为保存在服务器中，并且不能向用户公开。然后，使用标头中指定的签名算法（默认情况下为HMAC SHA256）根据以下公式生成签名。

HMACSHA256(base64UrlEncode(header) + “.” + base64UrlEncode(payload), secret) 在计算出签名哈希后，JWT头，有效载荷和签名哈希的三个部分组合成一个字符串，每个部分用".“分隔，就构成整个JWT对象。

## Base64URL算法

如前所述，JWT头和有效载荷序列化的算法都用到了Base64URL。该算法和常见Base64算法类似，稍有差别。

作为令牌的JWT可以放在URL中（例如api.example/?token=xxx）。 Base64中用的三个字符是”+"，"/“和”="，由于在URL中有特殊含义，因此Base64URL中对他们做了替换："=“去掉，"+“用”-“替换，"/“用”_“替换，这就是Base64URL算法，很简单把。

# 安装

通过composer安装

```sh
composer require lcobucci/jwt
```

- 在app/common/下创建一个trait目录
- 并且在trait目录下创建JWT.php文件
- 如：app/common/trait/JWT.php

JWT.php代码

```php
<?php

namespace app\common\traits;


use Lcobucci\JWT\Builder;
use Lcobucci\JWT\Parser;
use Lcobucci\JWT\Signer\Hmac\Sha256;

trait JWT
{

    public static $signKey = 'www.liqingbo.cn'; //签名KEY

    /**
     * 生成json web token 字符串
     * @param int $userId 用户id
     * @return string $token
     */
    protected function generateJWT($userId)
    {
        $signer = new Sha256();

        return (string)(new Builder())->setIssuer('liqingbo')
            ->setIssuedAt(time())
            ->set('user_id', $userId)
            ->sign($signer, self::$signKey)
            ->getToken();
    }

    /**
     * 验证token是否有效
     * @param string $token token字符串aaa.bbb.ccc
     * @return bool
     */
    protected function verifyJWT($token)
    {
        $tokenObj = $this->parseJWT($token);
        $signer = new Sha256();
        return $tokenObj->verify($signer, self::$signKey);
    }

    /**
     * 将jwt字符串解析成Token对象
     * @param $token
     * @return \Lcobucci\JWT\Token
     */
    protected function parseJWT($token)
    {
        return (new Parser())->parse((string) $token); // Parses from a string
    }

    /**
     * 从jwt字符串中获取用户ID
     * @param string $token
     * @return mixed
     */
    protected function getUserIdFromJWT($token)
    {
        $tokenObj = $this->parseJWT($token);
        return $tokenObj->getClaim('user_id');
    }
}
```

##### 通过上面两步：

- 安装JWT
- 通过traits公共调用代码

就可以开始调用了，这里建议放在中间件或者公共控制器里面

```php
<?php

declare (strict_types = 1);

namespace app\api\controller;
use app\BaseController;

use app\common\traits\JWT; //通过命名空间引入
use think\facade\Request;

class CommonController extends BaseController
{
    use JWT; //还有这里，别忘记了

    public function checkAuth()
    {
        $header = Request::header();
        $token = $header['token']; //获取通过header传过来的token

        // 1.先从header取出token字符串
        if (!empty($token)) {
            $tokenString = (string)$token;
            if (empty($tokenString)) {
                exception("token值为空", 100001);
            }

            // 2.验证token是否合法
            if ($this->verifyJWT($tokenString)) {
                // 3.将token字符串转换成Token对象然后取出user_id
                $data = $this->getUserIdFromJWT($tokenString);

                // 单点登录
                // 4.在redis里查找token查找数据，如果查不到则返回token不存在
                //$token = Cache::store('redis')->get('user_id:' . $userId);

            } else {
                exception('非法token', 100001);
            }
        } else {
            exception('请求头缺少token参数', 100001);
        }

    }

}
```

