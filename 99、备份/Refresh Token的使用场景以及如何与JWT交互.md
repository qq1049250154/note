在这篇文章中，我们将探索由[OAuth2](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Ftools.ietf.org%2Fhtml%2Frfc6749)定义的Refresh Token的概念。我们将会明白为什么他们会这样做，以及他们如何与其他类型的Token进行比较。我们也将通过一个简单的例子来学习如何使用它们。

`更新：` 目前这篇文章写的Auth0还没有通过[OpenID Connect认证](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Fopenid.net%2Fcertification%2F)。本文中使用的某些术语（如`access token`）不符合此规范，但符合[OAuth2规范](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Ftools.ietf.org%2Fhtml%2Frfc6749%23section-1.4)。OpenID Connect在`access token`（用于访问授权服务器的API）和`id token`（用于针对资源服务器的客户端验证）之间建立明确的区别。

## 介绍

现代的认证或者授权的解决方案已经将`token`引入到了协议当中。`token`是一种特殊的数据片段，用来授权用户执行特定的操作，或允许客户获得关于授权过程的额外信息（然后完成）。换句话说，令牌是允许授权过程执行的信息。该信息是否可由客户端（或授权服务器以外的任何方）读取或解析，由该实现定义。重要的是：客户端获取这些信息，然后用来获取特定的资源。JSON Web Token(JWT)[规范](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Ftools.ietf.org%2Fhtml%2Frfc7519)定义了一种代表通用的`token`信息的方式。

## JWT简短回顾

JWT定义了可以表示与认证/授权过程有关的某些共同信息的方式。顾名思义，数据格式是JSON。JWT拥有subject，issuer，过期时间等通用属性。JWT与其他规范（如[JSON Web签名](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Ftools.ietf.org%2Fhtml%2Frfc7515)（JWS）和[JSON Web加密](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Ftools.ietf.org%2Fhtml%2Frfc7516)（JWE））结合使用时会变得非常有用。 这些规范不仅提供了授权`token`通常需要的所有信息，还提供了一种验证`token`内容的方法，以便它不会被篡改（JWS）和一种加密信息的方法，以使其对于客户端（JWE）。数据格式（及其他优点）的简单性已经帮助JWT成为最常见的`token`类型之一。如果您有兴趣学习如何在您的Web应用程序中实现JWT，请查看Ryan Chenkie撰写的优秀[文章](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fauth0.com%2Fblog%2F2015%2F09%2F28%2F5-steps-to-add-modern-authentication-to-legacy-apps-using-jwts%2F)。

## Token类型

为了这篇文章的目的，我们将重点讨论两种最常见的`token`类型：`access token`和`refresh token`。

- Access Token携带了直接访问资源的必要信息。换句话说，当客户端将`access token`传给管理资源的服务器时，该服务器可以使用`token`中包含的信息来决定是否授权给客户端。`access token`通常有一个过期时间，而且通常时间非常短暂。

![img](https://user-gold-cdn.xitu.io/2018/1/14/160f2bcf5950e9cd?w=1280&h=800&f=png&s=68325)

- Refresh Token携带了用来获取新的access token的必要信息。换句话说，当客户端需要使用access token来访问特定资源的时候，客户端可以使用refresh token来向认证服务器请求下发新的access token。通常情况下，当旧的access token失效之后，才需要获得新的access token，或者是在第一次访问资源的时候。refresh token也有过期时间但是时间相对较长。refresh token对存储的要求通常会非常严格，以确保它不会被泄漏。它们也可以被授权服务器列入黑名单。

![img](https://user-gold-cdn.xitu.io/2018/1/14/160f2c332aa4c34a?w=1280&h=800&f=png&s=74512)

通常由具体的实现来定义token是透明的还是不透明的。通用实现允许对access token进行直接授权检查。也就是说，当access token传递给管理资源的服务器时，服务器可以读取token中包含的信息，并决定用户是否被授权（不需要对授权服务器进行检查）。这就是token必须签名的原因之一（例如使用JWS）。另一方面，refresh token通常需要对授权服务器进行检查。处理授权检查的这种分离方式有以下3个优点：

1. 改进了对授权服务器的访问模式（更低的负载，更快的检查）
2. 泄露access token的访问窗口更短（这些access token会很快过期，从而减少泄露的token访问受保护资源的机会）
3. 滑动session（见下文）

## 滑动session

滑动session是只一段时间不活动后过期的session。正如你想到的，使用access token和refresh token可以很容易实现这个功能。当用户执行操作时，会发出一个新的access token。如果用户使用过期的access token，则session被认为是不活动的，并且需要新的access token。这个token是否可以通过access token获得，或者是否需要新的认证轮回，由开发团队的要求来定义。

## 安全考虑

refresh token的存活时间较长。这意味着当客户端获取refresh token时，必须安全的存储此token以防止潜在攻击者使用此token。如果refresh token泄露，它可能会被用来获取新的access token（并访问受保护的资源），直到它被列入黑名单或到期（可能需要很长时间）。refrsh token必须发给单个经过身份验证的客户端，以防止其他方使用泄漏的token。访问令牌必须保密，但是正如你所想象的那样，安全考虑因其寿命较短而不那么严格。

## 实例：Refresh Token发放服务器

为了这个例子的目的，我们使用一个基于[node-oauth2-server](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fthomseddon%2Fnode-oauth2-server)的简单的服务器来发布access token和refresh token。访问受保护的资源需要access token。客户端使用简单的curl命令。此示例中的代码基于[node-oauth2-server](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fthomseddon%2Fnode-oauth2-server%2Ftree%2Fmaster%2Fexamples)中的示例。我们已经修改了基本示例，access token使用JWT格式。Node-oauth2-server为模型使用预定义的API。你可以点[这里](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fthomseddon%2Fnode-oauth2-server%2Fblob%2Fmaster%2FReadme.md)查看文档。以下代码展示了如何实现JWT格式的access token模型。

`免责声明`：请注意以下示例中的代码不是为生产环境准备的。

```js
model.generateToken = function(type, req, callback) {
  //Use the default implementation for refresh tokens
  console.log('generateToken: ' + type);
  if(type === 'refreshToken') {
    callback(null, null);
    return;
  }

  //Use JWT for access tokens
  var token = jwt.sign({
    user: req.user.id
  }, secretKey, {
    expiresIn: model.accessTokenLifetime,
    subject: req.client.clientId
  });

  callback(null, token);
}

model.getAccessToken = function (bearerToken, callback) {
  console.log('in getAccessToken (bearerToken: ' + bearerToken + ')');

  try {
    var decoded = jwt.verify(bearerToken, secretKey, {
        ignoreExpiration: true //handled by OAuth2 server implementation
    });
    callback(null, {
      accessToken: bearerToken,
      clientId: decoded.sub,
      userId: decoded.user,
      expires: new Date(decoded.exp * 1000)
    });
  } catch(e) {    
    callback(e);
  }
};

model.saveAccessToken = function (token, clientId, expires, userId, callback) {
  console.log('in saveAccessToken (token: ' + token +
              ', clientId: ' + clientId + ', userId: ' + userId.id +
              ', expires: ' + expires + ')');

  //No need to store JWT tokens.
  console.log(jwt.decode(token, secretKey));

  callback(null);
};
```

OAuth2 token端点（/oauth/token）处理所有类型的授权（密码和refresh token）的发放。其它所有端点都是受保护的，需要检查access token。

```js
// Handle token grant requests
app.all('/oauth/token', app.oauth.grant());

app.get('/secret', app.oauth.authorise(), function (req, res) {
  // Will require a valid access_token
  res.send('Secret area');
});
```

因此，例如，假设有一个用户'test'，密码'test'和一个客户端'testclient'，客户端密码'secret'，可以请求一个新的access token/refresh token对，如下所示：

```shell
$ curl -X POST -H 'Authorization: Basic dGVzdGNsaWVudDpzZWNyZXQ=' -d 'grant_type=password&username=test&password=test' localhost:3000/oauth/token

{
    "token_type":"bearer",
    "access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiVlx1MDAxNcKbwoNUwoonbFPCu8KhwrYiLCJpYXQiOjE0NDQyNjI1NDMsImV4cCI6MTQ0NDI2MjU2M30.MldruS1PvZaRZIJR4legQaauQ3_DYKxxP2rFnD37Ip4",
    "expires_in":20,
    "refresh_token":"fdb8fdbecf1d03ce5e6125c067733c0d51de209c"
}
```

授权头包含以BASE64（testclient:secret）编码的客户端ID和密钥。

使用该access token访问受保护的资源：

```shell
$ curl 'localhost:3000/secret?access_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiVlx1MDAxNcKbwoNUwoonbFPCu8KhwrYiLCJpYXQiOjE0NDQyNjI1NDMsImV4cCI6MTQ0NDI2MjU2M30.MldruS1PvZaRZIJR4legQaauQ3_DYKxxP2rFnD37Ip4'

Secret area
```

由于JWT，访问”安全区域“（就是受保护资源）不需要通过查询数据库来校验access token。

一旦token过期：

```shell
$ curl 'localhost:3000/secret?access_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiVlx1MDAxNcKbwoNUwoonbFPCu8KhwrYiLCJpYXQiOjE0NDQyNjI2MTEsImV4cCI6MTQ0NDI2MjYzMX0.KkHI8KkF4nmi9z6rAQu9uffJjiJuNnsMg1DC3CnmEV0'

{
    "code":401,
    "error":"invalid_token",
    "error_description":"The access token provided has expired."
}
```

现在我们可以通过refresh token来获取新的access token，如下所示：

```shell
$ curl -X POST -H 'Authorization: Basic dGVzdGNsaWVudDpzZWNyZXQ=' -d 'refresh_token=fdb8fdbecf1d03ce5e6125c067733c0d51de209c&grant_type=refresh_token' localhost:3000/oauth/token

{
    "token_type":"bearer",
    "access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiVlx1MDAxNcKbwoNUwoonbFPCu8KhwrYiLCJpYXQiOjE0NDQyNjI4NjYsImV4cCI6MTQ0NDI2Mjg4Nn0.Dww7TC-d0teDAgsmKHw7bhF2THNichsE6rVJq9xu_2s",
    "expires_in":20,
    "refresh_token":"7fd15938c823cf58e78019bea2af142f9449696a"
}
```

在[这里](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fauth0%2Fblog-refresh-tokens-sample)查看完整代码。

## 另外：在你的auth0应用中使用refresh token

在auth0应用中我们为你解决了认证的难点。一旦你[配置](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fauth0.com%2Fdocs)了我们的应用程序，你就可以根据这里的[文档](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fauth0.com%2Fdocs%2Frefresh-token)来获取refresh token。

## 结论

refresh token提升了安全性，并缩短了授权时间，提供了访问授权服务器的更好的一种新模式。使用JWT + JWS等工具可以简化实现。如果您有兴趣了解更多关于token（和cookies）的信息，请查看我们的[文章](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fauth0.com%2Fblog%2F2014%2F01%2F27%2Ften-things-you-should-know-about-tokens-and-cookies%2F)。

原文地址：[https://auth0.com/blog/refresh-tokens-what-are-they-and-when-to-use-them/](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fauth0.com%2Fblog%2Frefresh-tokens-what-are-they-and-when-to-use-them%2F)