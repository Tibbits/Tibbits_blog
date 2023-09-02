# 基于OAuth2.0的第三方登录

> 配置docsify的gittalk似乎就比较简单，按着教程代码来就行。但当时在配置docsify的gittalk的时候没有第一时间看到docsify文档写的这方面的内容，后面发现原来自己查的是针对普通网站的配置gittalk，看到了基于OAuth2.0的第三方登录，看着看着发现有点感兴趣，便想找找资料深入了解一下，由于并不太了解前端的知识，因此一些基础的知识我也一同查阅了并放了在里面。

- docsify上的代码

```
<link rel="stylesheet" href="//cdn.jsdelivr.net/npm/gitalk/dist/gitalk.css">

<script src="//cdn.jsdelivr.net/npm/docsify/lib/plugins/gitalk.min.js"></script>
<script src="//cdn.jsdelivr.net/npm/gitalk/dist/gitalk.min.js"></script>
<script>
  const gitalk = new Gitalk({
    clientID: 'Github Application Client ID',
    clientSecret: 'Github Application Client Secret',
    repo: 'Github repo',
    owner: 'Github repo owner',
    admin: ['Github repo collaborators, only these guys can initialize github issues'],
    // facebook-like distraction free mode
    distractionFreeMode: false
  })
</script>
```

##### 这段代码是用于在网页中添加Gitalk评论系统的。Gitalk是一个基于GitHub Issue和Preact开发的评论系统，可以让用户在网页中进行评论和交流。

- 第一行代码引入了Gitalk的CSS文件，用于样式化评论系统的外观。
- 第二行代码引入了Docsify插件的Gitalk JavaScript文件，用于在Docsify文档中使用Gitalk评论系统。
- 第三行代码引入了Gitalk的JavaScript文件，用于实际初始化和运行评论系统。
- clientID: GitHub应用程序的客户端ID，需要在GitHub上注册应用程序来获取。
- clientSecret: GitHub应用程序的客户端密钥，也需要在GitHub上注册应用程序来获取。
- repo: GitHub仓库的名称，用于存储评论数据。
- owner: GitHub仓库的所有者，可以是个人或组织。
- admin: GitHub仓库的协作者，只有这些人才能初始化GitHub问题。
- distractionFreeMode: 是否开启类似Facebook的无干扰模式，如果设置为true，将隐藏页面上的其他元素。

##### 点开setting，从develop settings中点开OAuth app

![image-20230902162235204](../../_image/oauth.png)

##### 填好信息

![image-20230902163504377](../../_image/oauth2.png)

##### 现在是配置好的图，开始的时候就弄一个secret

![image-20230902163024490](../../_image/oauth1.png)

> 以上是初步配置



基于OAuth2.0的第三方登录是一种授权机制，它允许用户使用第三方平台的身份验证来登录其他网站或应用程序，而无需创建新的账户。就是将三方的帐号绑定到产品自身的帐号上，当查询到用户第三方的帐号已经绑定了平台的某个user_id时，直接登录对应的帐号。例如利用微信QQ快捷登陆，可以不用注册APP账号，先点击跳转到微信，再点击授权进去APP中。



------



> 以下内容转载自作者fishBugs

从用户点击`login with github`开始，中间有几次重要的HTTP报文传递？

与登录认证有关的http请求有几个？

**3个**！！！说清楚这一点，基本就明白了。

首先我们构造了这样的html页面

```xml
<body>
  <a href="https://github.com/login/oauth/authorize\
?client_id=47878c9a96cfc358eb6e">
    login with github</a>
</body>
```

关键是我们构造了一个url

- ```
  https://github.com/login/oauth/authorize
  ```

这一部分是固定的，只要是用GitHub登录，就得这样。你可以从官方文档上看到这个链接。

如果是微信登录，twitter登录，也是大同小异，具体的url可以在相关的官网上找到。

GitHub会监听这个路由，来做出登录处理，然后后面是一个查询字符串

```
client_id=47878c9a96cfc358eb6e
```

它的值就是我们之前申请到的client id。这是个必须存在的字段，用户点击登录后，GitHub就是通过这个字段来确认用户究竟是想登录哪个网站。然后GitHub会返回这样的页面，确认用户是否真的要登录，这就是第一次HTTP请求和响应然后用户点击确认后发生了什么？

- 用户点击确认，就是向GitHub发送一个报文，确认自己确实到登录某网站
- GitHub收到这个用户的确认消息，**然后会返回一个状态码为301的报文**

这个是第二次HTTP请求和响应。因为浏览器收到301重定向后，会直接前往新的网址了，所以你要是没仔细看的话，可能就忽略了。

这个状态码为301的报文，它的Location字段大概长这样：`https://github.com/fish56/OAuth?code=a1aee8cacf7560825665>`

- ```
  https://github.com/fish56/OAuth
  ```

这个字段就是我们之前填写的callback url，正常情况下这个应该是我们云服务器的网址。但是这里为了演示方便，我这里就随便填写了我的github地址，没关系的

- ```
  code=a1aee8cacf7560825665
  ```

这个就是我们OAuth登录的一个核心信息。之前说过，**我们OAuth登录的核心目的就是让第三方网站能够安全的拿到用户的token**。用户的浏览器收到之前的301的HTTP响应后，就会向我们服务器发起请求，请求的同时服务器就拿到了这个code。服务器就可以通过这个code从github拿到用户的token。

这就是第三次http请求。

然后又有同学可能会问了，为什么要返回一个code，而不是直接返回一个token呢？

答：为了安全，如果token经过用户的手里走一遍，就可能会被其他恶意的人窃取。

OAuth协议下，GitHub会返回给用户一个code，然后用户浏览器通过重定向携带这个code来访问我们的服务器，这样服务器就拿到了这个code。

服务器拿到这个code 之后，通过结合之前的client secret向GitHub申请token，这样会安全一点。

之前我们只做了一半， 接下来我们通过postman来演示一下如何通过code拿到token。

我们要向GitHub申请用户的token，需要

- code（这个只有在用户同意登录后，服务器才能拿到。）
- client secret + client id

```
https://github.com/login/oauth/access_token
```

向以上发起POST请求，并且携带上面的三个字段

这个url是GitHub规定的，你可以在它的官方文档中找到

然后我们在postman中构造这样的请求：

可以看到，这我们确实拿到了用户的token：

`access_token=9094eb58a23093fd593d43eb28c1f06ce7904ed5&scope=&token_type=bearer`。

只不过真实情况下上面的操作都是由线上的服务器完成的，我这样操作是方便大家的理解。



------



> 以下内容转载自作者程序人生Jeffrey

- #### 几个重要概念

  - ##### 外部标识

    用来使用用户身份的标志，可以是用户名，手机号，邮箱等，每一个外部标识一定和一个内部标识相关联用以确定一个用户。
    外部标识的作用有两个

    - 让用户通过自己熟知且占有的外部标识来登录产品
    - 可以通过校验外部标识来实现找回或转移数据资产

  - ##### 内部标识

    即产品中用于标识用户唯一性的标志，例如user_id，必须有，不可更改且唯一，用户一般接触不到内部标识。
    当一个内部标识建立后，用户所有的数据资产都会绑定到这个内部标识上。

  - ##### user_id

    一个常用的内部标识，类似你的18位身份证ID

  - ##### app_id

    用于区别不同APP的ID，具有唯一性。

  - ##### open_id

    第三方平台为了用户信息的安全，一般不会直接将用户的内部标识给到其他产品，而是选择了给一个外部标识，这个open_id就是微信给各个APP用以区分微信用户身份的外部标识。

  - ##### union_id

    不同的产品的可以使用同一个union_id来确认用户的身份。

  - ##### access_token

    可以理解为通行证，有了这个通行证，就能获取到第三方平台指定用户的有限信息。

  - ##### OAuth 2.0

    OAuth2.0就是客户端和认证服务器之间由于相互不信任而产生的一个授权协议，只要授权方和被授权方遵守这个协议去写代码提供服务，那双方就是实现了OAuth2.0模式。

#### 为什么需要加入第三方登录

- 提高登录转化率，登录更加快捷，不需要输入密码
- 提高注册转化率，注册更加快速，方便获取用户信息
- 信赖感（让用户觉得这个产品和大厂是有合作的，提高对产品的信赖感）

#### OAuth2.0协议规范流程

![](../../_image/oauth3.png)

- Client请求RO的授权，请求中一般包含：要访问的资源路径，操作类型，Client的身份等信息。
- RO批准授权，并将“授权证据”发送给Client。至于RO如何批准，这个是协议之外的事情。典型的做法是，AS提供授权审批界面，让RO显式批准。这个可以参考下一节实例化分析中的描述。
- Client向AS请求“访问令牌(Access Token)”。此时，Client需向AS提供RO的“授权证据”，以及Client自己身份的凭证。
-  AS验证通过后，向Client返回“访问令牌”。访问令牌也有多种类型，若为bearer类型，那么谁持有访问令牌，谁就能访问资源。
-  Client携带“访问令牌”访问RS上的资源。在令牌的有效期内，Client可以多次携带令牌去访问资源。
-  RS验证令牌的有效性，比如是否伪造、是否越权、是否过期，验证通过后，才能提供服务。

#### 最典型的Authorization Code 授权模式

![](../../_image/oauth4.png)

##### 核心思想：

- oauth 的核心思想就是要让第三方在不知道用户名密码的情况下完成鉴权，但是没有密码用户名组合根本不可能有效鉴权， oauth 实际的过程是一个李代桃疆的手法。在第一方用你的原始用户名和密码组合，生成另外一对名称密码组合，这个阶段叫做获取 code 和 state,这对组合送到第二方也就是你的资源所在地，同样较验一遍，如果合格，给你生成一个带有时效性的 access token, 第三方在有效期内拿着这个 access token 跳过第一方直接请求第二方的资源
- 至于为什么不直接返回 access token? 是因为如果使用 code 方式的话，服务器获得用户授权后通过 302 跳转到你的 callback URI 上，并在 url query 上带上用于交换 accesd token 的 code ，你在浏览器地址栏就可以看到这个code ，已经暴露有可能被不法应用，所以在 url 上直接返回 access token 是不安全的，而client拿到code以后换取access token是client后台对认证服务器的访问，并且需要clientID和client secret，不依赖浏览器，access token不会暴露出去。

#### 为何引入authorization_code？

##### 		因为单从OAuth2.0的授权过程来看，如果直接返回access_token，协议将变得更加简洁，而且少一次Client与AS之间的交互，性能也更优，其实不然。引入authorization_code有很多妙处，主要原因如下：

- 浏览器的redirect_uri是一个不安全信道，此方式不适合于传递敏感数据（如access_token），会显著扩大access_token被泄露的风险。
  但authorization_code可以通过redirect_uri方式来传递，是因为authorization_code并不像access_token一样敏感。
  即使authorization_code被泄露，攻击者也无法直接拿到access_token，因为拿authorization_code去交换access_token是需要验证Client的真实身份。
- 由于协议需要验证Client的身份，如果不引入authorization_code，这个Client的身份认证只能通过第1步的redirect_uri来传递。同样由于redirect_uri是一个不安全信道，这就额外要求Client必须使用数字签名技术来进行身份认证，而不能用简单的密码或口令认证方式。
  引入authorization_code之后，AS可以直接对Client进行身份认证（见步骤4和5），而且可以支持任意的Client认证方式（比如，简单地直接将Client端密钥发送给AS）。

​		OAuth 协议设计不同于简单的网络安全协议的设计，因为OAuth需要考虑各种Web攻击，比如CSRF (Cross-Site Request Forgery), XSS (Cross Site Script), Clickjacking。在redirect_uri中引入state参数就是从浏览器安全角度考虑的，有了它就可以抵制CSRF攻击。

![](../../_image/oauth5.png)



------

