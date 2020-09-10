# 点击劫持

## 什么是点击劫持？

点击劫持是一种视觉欺骗的攻击手段，攻击方诱导用户点击一个看不见的网页或者经过伪装的元素，实际上用户点击的并不是自己看到的网页，而是这个网页上层的一个透明网页，这种攻击可能会导致用户下载恶意软件、泄露个人信息、或者在不知情的情况下进行转账、购物等等。

一般情况下，攻击方是把需要攻击的网站通过 iframe 嵌入到自己的网页中，再设置其为透明或伪装成其他元素。用户访问攻击方的网页，可能会看到一个“领取免费礼物”的按钮，但实际上用户点击这个按钮，却是点击了这个按钮上方的透明网页，这个看不见的网页可能是用户的银行转账页面，而用户点击了按钮之后，就同意转账了。

> p.s. 因为浏览器可能会对透明的 iframe 做一些安全检测，所以攻击方可能会把网页设置成 `opacity: 0.0001;` 之类的，越过浏览器的检测阈值。

## 如何防御点击劫持？

1. 客户端防御：JS 防御
2. 服务器防御：X-Frame-Options 和 Content Security Policy (CSP)

### JS 防御

-   检查当前网页的 window 是否 top-window，如果不是，那就把它改成 top-window
-   把所有 frame 都设为不透明的
-   阻止透明 frame 上的点击事件
-   ...

但 JS 防御很容易被绕过，比如浏览器可能设置了禁止执行 JS 脚本，或者攻击方可能会利用 HTML5 iframe 的 `sandbox` 属性。

### X-Frame-Options

`X-Frame-Options` 是一个 HTTP 响应头部字段，用来控制浏览器是否可以在 `<frame>` 或者 `<iframe>` 标签中展示当前页面，有以下几个可以设置的值：

-   DENY: 不允许任何网站在 iframe 中展示当前网页
-   SAMEORIGIN: 在域名相同的情况下，可以允许将当前页面在 iframe 中展示
-   ALLOW-FROM URI: 在指定的 URI 中，允许当前页面在 iframe 中展示

**缺点**

1. 如果要使用 SAMEORIGIN，需要在网站的每一个单独的页面中都返回 `X-Frame-Options` 头部
2. 使用 ALLOW-FROM URI 的时候，不支持设置多个 URI
3. `X-Frame-Options` 只能设置一个值
4. 大部分浏览器都不支持 ALLOW-FROM 选项
5. `X-Frame-Options` 在大部分浏览器中已经被丢弃

### Content Security Policy (CSP)

`Content Security Policy` 也是一个 HTTP 响应头部，我们可以通过设置 `frame-ancestors` 指令来指定哪些源可以通过 `<iframe>` 等方式来展示当前页面，可以指定一个或多个源。

**用法**

`Content-Security-Policy: frame-ancestors <source> <source>;`

e.g.

```
Content-Security-Policy: frame-ancestors 'none';

Content-Security-Policy: frame-ancestors 'self' https://www.example.org;
```

-   `none`: 是一个关键字，跟 `X-Frame-Options` 的 `DENY` 一样
-   `self`: 是一个关键字，跟 `X-Frame-Options` 的 `SAMEORIGIN` 一样
-   此外还可以指定多个 URI

**READING**

-   [x] https://www.imperva.com/learn/application-security/clickjacking/
-   [x] https://portswigger.net/web-security/clickjacking (notes: read the extension readings in this blog when you got time)
