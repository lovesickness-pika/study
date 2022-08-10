### 使用Token实现用户登录

#### 为什么要使用Token

##### session

- 使用session验证cookie对，当用户数量较多的时候，对服务器的负担太大
- 当有多台服务器时，服务器还需要同步sessionId
- cookie不支持跨域访问

##### Token的优势

- 防止跨域访问的问题
- 服务端不用存储状态，减轻服务器的负担

## 什么是 JSON Web Token 结构？

在其紧凑形式中，JSON Web Tokens 由用点 ( `.`)分隔的三个部分组成，它们是：

- 标题
- 有效载荷
- 签名

因此，JWT 通常如下所示。

```
xxxxx.yyyyy.zzzzz
```

### 标头

标头*通常*由两部分组成：令牌的类型，即 JWT，以及正在使用的签名算法，例如 HMAC SHA256 或 RSA。

例如：

```
{
  "alg": "HS256",
  "typ": "JWT"
}
```

然后，这个 JSON 被**Base64Url**编码以形成 JWT 的第一部分。

### 有效载荷

令牌的第二部分是负载，其中包含声明。声明是关于实体（通常是用户）和附加数据的声明。共有三种类型的声明：*注册声明*、*公共*声明和*私人*声明。

- [**注册声明**](https://tools.ietf.org/html/rfc7519#section-4.1)：这些是一组预定义的声明，这些声明不是强制性的，而是推荐的，以提供一组有用的、可互操作的声明。其中一些是： **iss**（发行者）、 **exp**（到期时间）、 **sub**（主题）、 **aud**（受众）[等](https://tools.ietf.org/html/rfc7519#section-4.1)。

  > 请注意，声明名称只有三个字符，因为 JWT 是紧凑的。

- [**公共声明**](https://tools.ietf.org/html/rfc7519#section-4.2)：这些可以由使用 JWT 的人随意定义。但是为了避免冲突，它们应该在[IANA JSON Web Token Registry](https://www.iana.org/assignments/jwt/jwt.xhtml)中定义或定义为包含抗冲突命名空间的 URI。

- [**私人权利**](https://tools.ietf.org/html/rfc7519#section-4.3)：这些都是使用它们同意并既不是当事人之间建立共享信息的自定义声明*注册*或*公众*的权利要求。

一个示例有效载荷可能是：

```
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

然后对有效负载进行**Base64Url**编码以形成 JSON Web 令牌的第二部分。

> 请注意，对于已签名的令牌，此信息虽然受到防篡改保护，但任何人都可以读取。除非加密，否则不要将机密信息放在 JWT 的负载或标头元素中。

### 签名

要创建签名部分，您必须获取编码的标头、编码的有效载荷、密匙、标头中指定的算法，并对其进行签名。

例如，如果要使用 HMAC SHA256 算法，则签名将通过以下方式创建：

```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

签名用于验证消息在此过程中没有更改，并且在使用私钥签名的令牌的情况下，它还可以验证 JWT 的发送者是它所说的那个人。

### 放在一起

输出是三个由点分隔的 Base64-URL 字符串，可以在 HTML 和 HTTP 环境中轻松传递，同时与基于 XML 的标准（如 SAML）相比更加紧凑。





#### token的使用

##### 编写token的工具类

