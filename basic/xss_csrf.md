---
title: XSS和CSRF
date: 2019-7-11 10:46:00
tags: [basic]
---

# XSS

[XSS](https://zh.wikipedia.org/wiki/%E8%B7%A8%E7%B6%B2%E7%AB%99%E6%8C%87%E4%BB%A4%E7%A2%BC)（cross-site scripting）跨站脚本攻击，是代码注入的一种。

XSS利用的是用户对指定网站的信任。

恶意用户利用服务器的漏洞，通过用户输入接口注入恶意代码，当普通用户访问被攻击的服务器时，恶意代码可能会窃取cookie或者其他信息。

当Web服务器采用token的验证方式时，如果将token保存在HTML5 Web Storage (localStorage or sessionStorage)，任何在该网站运行的JS脚本都可以获得该token，如果网站遭受XSS攻击，用户token将会泄露。

## 防范措施
通过对用户的输入进行过滤。

```
><script>alert(document.cookie)</script>
='><script>alert(document.cookie)</script>
"><script>alert(document.cookie)</script>
<script>alert(document.cookie)</script>
<script>alert (vulnerable)</script>
%3Cscript%3Ealert('XSS')%3C/script%3E
<script>alert('XSS')</script>
<img src="javascript:alert('XSS')">
<img src="http://888.888.com/999.png" onerror="alert('XSS')">
<div style="height:expression(alert('XSS'),1)"></div>（这个仅于IE7(含)之前有效）
```

# CSRF

[CSRF](https://zh.wikipedia.org/wiki/%E8%B7%A8%E7%AB%99%E8%AF%B7%E6%B1%82%E4%BC%AA%E9%80%A0)（cross-site request forgery）跨站请求伪造，是一种挟制用户在已登录的Web应用程序上执行非本意的操作的攻击方法。

CSRF利用的是网站对用户网页浏览器的信任。

当Web服务器采用token的验证方式时，如果将token保存在cookie中，CSRF会引诱已登录的用户执行非本意的操作。

例如有这样一个页面
```
<body>

    hi
    
    <div style="display:none;">
        <form action="http://darharry:8080" method="POST">
            <input name="_xsrf" value="HFPl48uXMTGMs70B5imNTp6Tj11QgX87" />
            <input name="username" value="hacker" />
        </form>
    </div>

    <script type="text/javascript">
        function do_commit() {
            window.document.forms[0].submit();
        }

        document.onload(
            do_commit()
        )
    </script>
</body>
```

当用户打开这个页面时，浏览器会携带darharry网站的cookie向darharry网站递交请求，如果Web服务器验证cookie通过，本次请求会成功，从而被攻击。

## 防范措施
### 利用CSRF TOKEN
Web服务器向客户端的cookie写入一个csrf token，或者在渲染Form时，给出一个隐藏的csrf token，Web前端提交请求时必须获取csrf token与Form的内容一起提交，否则服务器拒绝该请求。因为其他网站攻击者无法获取到该token，因此可以防范CSRF攻击。

如果采用cookie的方式保存csrf，不需要设置该cookie为HTTP Only，因为HTTP Only主要用于使JS不能使用该值，以防范XSS攻击，但是csrf token必须能够以某种方式调用。

### 检查Origin和Referer
如果请求是从攻击网站传来，Origing和Referer字段将会是攻击网站的地址，因此可以通过检查该字段判断是否是恶意的请求。
