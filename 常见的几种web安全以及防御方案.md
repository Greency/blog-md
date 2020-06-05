#### 一、XSS（跨站脚攻击）

##### 1. 定义

XSS全称Cross Site Scripting。通过在HTML文件或者DOM中注入恶意代码，借此对用户实施攻击。

##### 2. 造成的影响

1. 窃取Cookie信息：恶意代码通过document.cookie可获取到所有的cookie信息，并通过CORS将数据发送到恶意的服务器。
2. 监听用户行为：使用addEventListener接口监听用户的输入。
3. 修改DOM：伪造假的登录窗口等。
4. 在页面中生成浮窗广告。

##### 3. 三种XSS类型

###### 1. 储存型XSS

1. 恶意代码储存在数据库中。
2. 用户请求包含恶意代码的页面。
3. 恶意代码将用户的Cookie等信息上传到服务器。

###### 2. 反射型XSS

在反射型XSS中，恶意代码属于用户发送给网站请求的一部分，随后网站又将恶意代码返回给用户。

1. 举例说明：

```javascript
//服务端
const express = require('express')
const router = express.Router()

router.get('/', (req, res, next) => {
    res.render('index', { title: 'Express', xss: res.query.xss })
})

module.exports = router;

//html模板
<!DOCTYPE html>
<html>
<head>
  <title><%= title %></title>
  <link rel='stylesheet' href='/stylesheets/style.css' />
</head>
<body>
  <h1><%= title %></h1>
  <p>Welcome to <%= title %></p>
  <div>
      <%- xss %>
  </div>
</body>
</html>
```

当输入如下地址时：

```javascript
http://localhost:3000/?xss=<script>alert('你被xss攻击了')</script>
```

html模板中的<%- xss %>就会被替换成<script>alert('你被xss攻击了')</script>，这时候浏览器会执行。

2. 常见场景
   1. 在QQ群或者邮件等渠道诱导用户点击恶意链接。

###### 3. 基于DOM的XSS

1. 不涉及web服务器。
2. 网络劫持：直接在传输过程中修改HTML页面的内容。
3. wifi路由劫持；本地恶意软件劫持等。

##### 4. 如何预防

###### 1. 过滤/转码

```javascript
code:<script>alert('你被xss攻击了')</script>

//过滤后
code:

//转义后：
code:&lt;script&gt;alert(&#39;你被xss攻击了&#39;)&lt;/script&gt;
```

###### 2. CSP

1. 限制加载其他域下的资源文件。
2. 禁止向第三方域提交数据，比如发送http请求等。
3. 禁止直行内联脚本和未授权的脚本。
4. 提供上报机制。

具体的可以查看 [[Content-Security-Policy文档](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy)

###### 3. HttpOnly

1. 设置了HttpOnly属性的Cookie无法被document.cookie读取。
2. 服务器通过HTTP响应头来设置。

#### 二、CSRF

##### 1. 定义

CSRF全称Cross-site request forgery。通过诱导用户点开恶意网站，并利用用户在目标网站的登录状态发起跨站请求（由于跨站请求时，浏览器会默认的带上目标网站的Cookie信息），达到攻击的目的。

##### 2. 举例说明

1. 假设有如下转账接口

   ```javascript
   #同时支持POST和Get
   #接口 
   https://xxx.com/sendcoin
   #参数
   ##目标用户
   user
   ##目标金额
   number
   ```

2. 恶意操作的方式

   ```javascript
   //1. 自动发起get请求
   <html>
     <body>
       <h1>黑客的站点：CSRF攻击演示</h1>
       <img src="https://time.geekbang.org/sendcoin?user=hacker&number=100">
     </body>
   </html>
   
   //2. 自动发起post请求
   <html>
   <body>
     <h1>黑客的站点：CSRF攻击演示</h1>
     <form id='hacker-form' action="https://time.geekbang.org/sendcoin" method=POST>
       <input type="hidden" name="user" value="hacker" />
       <input type="hidden" name="number" value="100" />
     </form>
     <script> document.getElementById('hacker-form').submit(); </script>
   </body>
   </html>
   
   //3. 引诱用户点击链接
   <div>
     <img width=150 src=http://images.xuejuzi.cn/1612/1_161230185104_1.jpg> </img> </div> <div>
     <a href="https://time.geekbang.org/sendcoin?user=hacker&number=100" taget="_blank">
       点击下载美女照片
     </a>
   </div>
   ```

##### 3. 如何预防

###### 1. 利用Cookie的sameSite属性

由于在B网站进行A网站的请求时，默认会带上A网站的cookie，所以黑客就利用了此漏洞；使用SameSite属性后，就可以防止这样的情况出现。

在HTTP响应中，通过set-cookie字段设置Cookie时，可以塞上SameSite选项：

```javascript
set-cookie: 1P_JAR=2019-10-20-06; expires=Tue, 19-Nov-2019 06:36:21 GMT; path=/; domain=.google.com; SameSite=none
```

SameSite有如下三种值：

1. Strict ：浏览器会完全禁止第三方 Cookie。
2. Lax ：在跨站点的情况下，从第三方站点的链接打开和从第三方站点提交 Get 方式的表单这两种方式都会携带 Cookie。但如果在第三方站点中使用 Post 方法，或者通过 img、iframe 等标签加载的 URL，这些场景都不会携带 Cookie。
3. None：任何情况下都会发送 Cookie 数据。

###### 2. 验证请求的来源

1. origin
2. referer：由于referer可以不用上传，所以它并不可靠。

###### 3. CSRF Token

由服务端生成一个token，在发送请求时需要带上token，且从第三方发送的请求是无法获取到token。

#### 三、点击劫持

##### 1. 定义

点击劫持利用视觉欺骗，通过iframe将目标网站嵌套到自己的网站，并将iframe设置为透明，并设置一些按钮诱导用户点击。

##### 2. 举例说明（诱导用户关注）

1. 通过iframe引入B站的视频，并将iframe设置为完全透明。
2. 在iframe上覆盖一张性感图片，并设置一个假的“点击下载”按钮（此按钮的位置与B站中”关注“按钮一致）。
3. 当用户点击按钮时，实际上是点击的“关注”按钮，这样就达成了关注的行为。

##### 3. 如何预防

###### 1. X-FRAME-OPTIONS

X-FRAME-OPTIONS是一个HTTP响应头，它的出现就是为了防止iframe嵌套的劫持攻击。

它有如下三种值：

1. DENY：表示页面不允许通过iframe展示。
2. SAMEORIGIN：表示页面可以在相同域名下通过iframe展示。
3. ALLOW-FROM：表示页面可以在指定来源的iframe中展示。

###### 2. JavaScript防御

```javascript
<head>
  <style id="hidden-style">
    html {
      display: none !important;
    }
  </style>
</head>
<body>
  <script>
    if (self === top) {
      var style = document.getElementById('hidden-style')
      document.body.removeChild(style)
    }
  </script>
</body>
```

原理是通过判断self与top是否相等，如果相等则去掉“隐藏整个html内容”的样式。