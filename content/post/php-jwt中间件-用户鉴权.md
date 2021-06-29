---
title: "Php Jwt中间件 用户鉴权"
date: 2020-06-10
categories: ["后端"]
keywords: ["php","thinkphp6.0","jwt"]
tags: ["PHP","thinkphp6.0","JWT"]
draft: true
---

**在上一篇文章中大概对jwt有了一个大概的了解，现在我们用tp6配合jwt做一个中间件，实现对用户签发token还有鉴权**

## 签发token

直接开始上代码：

```php
   /**
     * 生成用户token
     * @param $uid
     * @return mixed
     */
    public function signToken($uid)
    {
        $data['uid'] = $uid;
        //生成用户token
        $key = '******';
        $token = array(
            "iss" => $key,         //签发者 可以为空
            "aud" => '',           //面象的用户，可以为空
            "iat" => time(),       //签发时间
            "nbf" => time() + 3,   //在什么时候jwt开始生效  （这里表示生成100秒后才生效）
            "exp" => time() + 7200, //token 有效时间(两个小时)
            "uid" => $uid
        );
        $jwt = JWT::encode($token, $key, "HS256");  //根据参数生成了 token
        $data['token'] = $jwt;

        return $data;
    }

```

```php
    /**
     * 解密token
     * @param $token
     * @return object
     */
    public static function checkToken($token)
    {
        //解决时间不同步问题
        JWT::$leeway += 15;
        $key = "*****";
        $info = JWT::decode($token, $key, ["HS256"]); //解密jwt
        return $info;
    }

```

## 定义中间件

```sh
php think make:middleware Check
```

```php
    /**
     * 路由中间件拦截
     * @param $request
     * @param \Closure $next
     * @return array|mixed
     * @throws TokenException
     */
    public function handle($request, \Closure $next)
    {
        //JWT验证登录
        $token = $request->header('Authorization');
        if (!$token) {
            return $this->renderError(Status::Unauthorized, Status::NotTokenMsg);
        }
        try {
            //解密token
            JwtToken::checkToken($token);
        } catch (\Firebase\JWT\SignatureInvalidException $e) {  //签名不正确
            throw new TokenException('签名异常', Status::TokenExpired);
        } catch (\Firebase\JWT\BeforeValidException $e) {  // 签名在某个时间点之后才能用
            throw new TokenException('签名在某个时间点之后才能用', Status::NotEffect);
        } catch (\Firebase\JWT\ExpiredException $e) {  // token过期{}
            throw new TokenException('token已过期', Status::TokenExpired);
        } catch (\Exception $e) {  //其他错误
            throw new TokenException('token已过期', Status::ERROR);
        }
        return $next($request);
    }

```

## 使用路由中间件

```php
//验证token接口
Route::group(function ()
{
    //上传图片接口
    Route::rule('****/***', 'api/**/**');
    Route::get('***/***', 'api/**/**');
    Route::post('***/***', 'api/**/**');

})->middleware(\app\api\middleware\UserAuth::class)->allowCrossDomain([
    'Access-Control-Allow-Origin'        => '*',
    'Access-Control-Allow-Credentials'   => 'true'
]);

```

