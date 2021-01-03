---
title: 浏览器跨域问题
date: 2013-06-02 10:15:21
tags:
- 网络安全
---

## 同源策略
同源策略（Same origin policy）是一种约定，它是浏览器最核心也最基本的安全功能，如果缺少了同源策略，则浏览器的正常功能可能都会受到影响。可以说 Web 是构建在同源策略基础之上的，浏览器只是针对同源策略的一种实现。
同源策略分为以下两种：

DOM 同源策略：禁止对不同源页面 DOM 进行操作。这里主要场景是 iframe 跨域的情况，不同域名的 iframe 是限制互相访问的。
XMLHttpRequest 同源策略：禁止使用 XHR 对象向不同源的服务器地址发起 HTTP 请求。

如果没有 DOM 同源策略，也就是说不同域的 iframe 之间可以相互访问，那么黑客可以这样进行攻击：
1、 做一个假网站，里面用 iframe 嵌套一个银行网站 http://mybank.com。
2、 把 iframe 宽高啥的调整到页面全部，这样用户进来除了域名，别的部分和银行的网站没有任何差别。
3、 这时如果用户输入账号密码，我们的主网站可以跨域访问到 http://mybank.com 的 dom 节点，就可以拿到用户的账户密码了。

如果 XMLHttpRequest 同源策略，那么黑客可以进行 CSRF（跨站请求伪造） 攻击：
1、 用户登录了自己的银行页面 http://mybank.com，http://mybank.com 向用户的 cookie 中添加用户标识。
2、 用户浏览了恶意页面 http://evil.com，执行了页面中的恶意 AJAX 请求代码。
3、 http://evil.com 向 http://mybank.com 发起 AJAX HTTP 请求，请求会默认把 http://mybank.com 对应 cookie 也同时发送过去。
4、 银行页面从发送的 cookie 中提取用户标识，验证用户无误，response 中返回请求数据。此时数据就泄露了。而且由于 Ajax 在后台执行，用户无法感知这一过程。

## 跨域
上面知道到了浏览器同源策略的作用，也正是有了跨域限制，才使我们能安全的上网。但是在实际中，有时候我们需要突破这样的限制 这就是跨域。因此下面将介绍几种跨域的解决方法。

一、 CORS（Cross-origin resource sharing，跨域资源共享）
使用自定义的 HTTP 头部让浏览器与服务器进行沟通，从而决定请求或响应是应该成功，还是应该失败。
整个 CORS 通信过程，都是浏览器自动完成，不需要用户参与。对于开发者来说，CORS 通信与同源的 AJAX 通信没有差别，代码完全一样。浏览器一旦发现 AJAX 请求跨源，就会自动添加一些附加的头信息，有时还会多出一次附加的请求，但用户不会有感觉。
因此，实现 CORS 通信的关键是服务器。只要服务器实现了 CORS 接口，就可以跨源通信。
浏览器将CORS请求分成两类：简单请求（simple request）和非简单请求（not-so-simple request）。

只要同时满足以下两大条件，就属于简单请求。
请求方法是以下三种方法之一：
HEAD
GET
POST
HTTP的头信息不超出以下几种字段：
Accept
Accept-Language
Content-Language
Last-Event-ID
Content-Type：只限于三个值 application/x-www-form-urlencoded、multipart/form-data、text/plain
凡是不同时满足上面两个条件，就属于非简单请求。

简单请求
1、 在请求中需要附加一个额外的 Origin 头部，其中包含请求页面的源信息（协议、域名和端口），以便服务器根据这个头部信息来决定是否给予响应。例如：Origin: http://www.test.cn
2、如果服务器认为这个请求可以接受，就在 Access-Control-Allow-Origin 头部中回发相同的源信息（如果是公共资源，可以回发 * ）。例如：Access-Control-Allow-Origin：http://www.test.cn
3、 没有这个头部或者有这个头部但源信息不匹配，浏览器就会驳回请求。正常情况下，浏览器会处理请求。注意，请求和响应都不包含 cookie 信息。
4、 如果需要包含 cookie 信息，ajax 请求需要设置 xhr 的属性 withCredentials 为 true，服务器需要设置响应头部 Access-Control-Allow-Credentials: true。

非简单请求
浏览器在发送真正的请求之前，会先发送一个 Preflight 请求给服务器，这种请求使用 OPTIONS 方法，发送下列头部：
	Origin：与简单的请求相同。
	Access-Control-Request-Method: 请求自身使用的方法。
	Access-Control-Request-Headers: （可选）自定义的头部信息，多个头部以逗号分隔。
例如：
``` bash
Origin: http://www.testweb.cn
Access-Control-Request-Method: POST
Access-Control-Request-Headers: NCZ
```

发送这个请求后，服务器可以决定是否允许这种类型的请求。服务器通过在响应中发送如下头部与浏览器进行沟通：

	Access-Control-Allow-Origin：与简单的请求相同。
	Access-Control-Allow-Methods: 允许的方法，多个方法以逗号分隔。
	Access-Control-Allow-Headers: 允许的头部，多个方法以逗号分隔。
	Access-Control-Max-Age: 应该将这个 Preflight 请求缓存多长时间（以秒表示）。
例如：
``` bash
Access-Control-Allow-Origin: http://www.testweb.cn
Access-Control-Allow-Methods: GET, POST
Access-Control-Allow-Headers: NCZ
Access-Control-Max-Age: 1728000
```
一旦服务器通过 Preflight 请求允许该请求之后，以后每次浏览器正常的 CORS 请求，就都跟简单请求一样了。

CORS 优点
1、 CORS 通信与同源的 AJAX 通信没有差别，代码完全一样，容易维护。
2、 支持所有类型的 HTTP 请求。
CORS 缺点
1、 存在兼容性问题，特别是 IE10 以下的浏览器。
2、 第一次发送非简单请求时会多一次请求。

二、 JSONP
由于 script 标签不受浏览器同源策略的影响，允许跨域引用资源。因此可以通过动态创建 script 标签，然后利用 src 属性进行跨域，这也就是 JSONP 跨域的基本原理。

直接通过下面的例子来说明 JSONP 实现跨域的流程：
1、 使用script标签的src属性来完成JSONP跨域请求

``` bash
// 前端代码
// 1. 定义一个 回调函数 handleResponse 用来接收返回的数据
function handleResponse(data) {
    console.log(data);
};

// 2. 动态创建一个 script 标签，并且告诉后端回调函数名叫 handleResponse
var body = document.getElementsByTagName('body')[0];
var script = document.getElement('script');
script.src = 'http://www.testweb.cn/json?callback=handleResponse';
body.appendChild(script);

// 3. 通过 script.src 请求 ‘http://www.testweb.cn/json?callback=handleResponse’，
// 4. 后端能够识别这样的 URL 格式并处理该请求，然后返回 handleResponse({"name": "testweb"}) 给浏览器
// 5. 浏览器在接收到 handleResponse({"name": "testweb"}) 之后立即执行 ，也就是执行 handleResponse 方法，获得后端返回的数据，这样就完成一次跨域请求了。

// 后端代码
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    response.setCharacterEncoding("UTF-8");
    response.setContentType("text/html;charset=UTF-8");

    //数据
    List<Student> studentList = getStudentList();

    JSONArray jsonArray = JSONArray.fromObject(studentList);
    String result = jsonArray.toString();

    //前端传过来的回调函数名称
    String callback = request.getParameter("callback");
    //用回调函数名称包裹返回数据，这样，返回数据就作为回调函数的参数传回去了
    result = callback + "(" + result + ")";

    response.getWriter().write(result);
}
```
2、使用jquery的JSONP方式实现跨域

``` bash
// 前端代码
<script>
    function showData (data) {
        console.info("调用showData");

        var result = JSON.stringify(data);
        $("#text").val(result);
    }
    $(document).ready(function () {
        $("#btn").click(function () {
            $.ajax({
                url: "http://localhost:9090/student",
                type: "GET",  // 此处即使改为 POST, jquery 也会自动会转为GET方式
                dataType: "jsonp",  //指定服务器返回的数据类型
                jsonp: "theFunction",   //指定参数名称，默认是callback
                jsonpCallback: "showData",  //指定回调函数名称
                success: function (data) {
                    console.info("调用success");
                }
            });
        });

    });
</script>

// 执行结果是，先调用callbak毁掉函数，
// 再调用 success

// 后端代码
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    response.setCharacterEncoding("UTF-8");
    response.setContentType("text/html;charset=UTF-8");

    //数据
    List<Student> studentList = getStudentList();

    JSONArray jsonArray = JSONArray.fromObject(studentList);
    String result = jsonArray.toString();

    //前端传过来的回调函数名称
    String callback = request.getParameter("theFunction");
    //用回调函数名称包裹返回数据，这样，返回数据就作为回调函数的参数传回去了
    result = callback + "(" + result + ")";

    response.getWriter().write(result);
}
```

JSONP 优点
使用简便，没有兼容性问题，目前最流行的一种跨域方法。
JSONP 缺点
只支持 GET 请求。
由于是从其它域中加载代码执行，因此如果其他域不安全，很可能会在响应中夹带一些恶意代码。
要确定 JSONP 请求是否失败并不容易。虽然 HTML5 给 script 标签新增了一个 onerror 事件处理程序，但是存在兼容性问题。

三、 图像 Ping 跨域
由于 img 标签不受浏览器同源策略的影响，允许跨域引用资源。因此可以通过 img 标签的 src 属性进行跨域，这也就是图像 Ping 跨域的基本原理。

直接通过下面的例子来说明图像 Ping 实现跨域的流程：
``` bash
var img = new Image();

// 通过 onload 及 onerror 事件可以知道响应是什么时候接收到的，但是不能获取响应文本
img.onload = img.onerror = function() {
    console.log("Done!");
}

// 请求数据通过查询字符串形式发送
img.src = 'http://www.testweb.cn/test?name=testweb';
```

图像 Ping 跨域,用于实现跟踪用户点击页面或动态广告曝光次数有较大的优势。
但是，同样只支持 GET 请求，且只能浏览器与服务器的单向通信，因为浏览器不能访问服务器的响应文本

四、 服务器代理
浏览器有跨域限制，但是服务器不存在跨域问题，所以可以由服务器请求所要域的资源再返回给客户端。服务器代理是万能的，但是因为经过了两次请求，所以效率不高。
以 nginx 为例，通过nginx配置一个代理服务器（域名与domain1相同，端口不同）做跳板机，反向代理访问domain2接口，并且可以顺便修改cookie中domain信息，方便当前域cookie写入，实现跨域登录。
``` bash 
// proxy服务器
server {
    listen       81;
    server_name  www.domain1.com;
    location / {
        proxy_pass   http://www.domain2.com:8080;  #反向代理
        proxy_cookie_domain www.domain2.com www.domain1.com; #修改cookie里域名
        index  index.html index.htm;

        # 当用webpack-dev-server等中间件代理接口访问nignx时，此时无浏览器参与，故没有同源限制，下面的跨域配置可不启用
        add_header Access-Control-Allow-Origin http://www.domain1.com;  #当前端只跨域不带cookie时，可为*
        add_header Access-Control-Allow-Credentials true;
    }
}
```

五、 document.domain 跨域
对于主域名相同，而子域名不同的情况，可以使用 document.domain 来跨域。这种方式非常适用于 iframe 跨域的情况。

比如，有一个页面，它的地址是 http://www.testweb.cn/a.html，在这个页面里面有一个 iframe，它的 src 是 http://testweb.cn/b.html。很显然，这个页面与它里面的 iframe 框架是不同域的，所以我们是无法通过在页面中书写 js 代码来获取 iframe 中的东西的。

这个时候，document.domain 就可以派上用场了，我们只要把 http://www.testweb.cn/a.html 和 http://testweb.cn/b.html 这两个页面的 document.domain 都设成相同的域名就可以了。但要注意的是，document.domain 的设置是有限制的，我们只能把 document.domain 设置成自身或更高一级的父域，且主域必须相同。例如：a.b.testweb.cn 中某个文档的 document.domain 可以设成 a.b.testweb.cn、b.testweb.cn 、testweb.cn 中的任意一个，但是不可以设成 c.a.b.testweb.cn ，因为这是当前域的子域，也不可以设成 baidu.com，因为主域已经不相同了。

例如，在页面 http://www.testweb.cn/a.html 中设置document.domain：
``` bash
<iframe src="http://testweb.cn/b.html" id="myIframe" onload="test()">
<script>
    document.domain = 'testweb.cn'; // 设置成主域
    function test() {
        console.log(document.getElementById('myIframe').contentWindow);
    }
</script>
```
在页面 http://testweb.cn/b.html 中也设置 document.domain，而且这也是必须的，虽然这个文档的 domain 就是 testweb.cn，但是还是必须显式地设置 document.domain 的值：
``` bash
<script>
    document.domain = 'testweb.cn'; // document.domain 设置成与主页面相同
</script>
```
这样，http://www.testweb.cn/a.html 就可以通过 js 访问到 http://testweb.cn/b.html 中的各种属性和对象了。

六、 window.name 跨域
window 对象有个 name 属性，该属性有个特征：即在一个窗口（window）的生命周期内，窗口载入的所有的页面（不管是相同域的页面还是不同域的页面）都是共享一个 window.name 的，每个页面对 window.name 都有读写的权限，window.name 是持久存在一个窗口载入过的所有页面中的，并不会因新页面的载入而进行重置。
通过下面的例子介绍如何通过 window.name 来跨域获取数据的。

页面 http://www.testweb.cn/a.html 的代码：
``` bash
<iframe src="http://testweb.cn/b.html" id="myIframe" onload="test()" style="display: none;">
<script>
    // 2. iframe载入 "http://testweb.cn/b.html" 页面后会执行该函数
    function test() {
        var iframe = document.getElementById('myIframe');
        
        // 重置 iframe 的 onload 事件程序，
        // 此时经过后面代码重置 src 之后，
        // http://www.testweb.cn/a.html 页面与该 iframe 在同一个源了，可以相互访问了
        iframe.onload = function() {
            var data = iframe.contentWindow.name; // 4. 获取 iframe 里的 window.name
            console.log(data); // hello world!
        };
        
        // 3. 重置一个与 http://www.testweb.cn/a.html 页面同源的页面
        iframe.src = 'http://www.testweb.cn/c.html';
    }
</script>
```
页面 http://testweb.cn/b.html 的代码
``` bash
<script type="text/javascript">
    // 1. 给当前的 window.name 设置一个 http://www.testweb.cn/a.html 页面想要得到的数据值 
    window.name = "hello world!";
</script>
```

七、 location.hash 跨域
location.hash 方式跨域，是子框架具有修改父框架 src 的 hash 值，通过这个属性进行传递数据，且更改 hash 值，页面不会刷新。但是传递的数据的字节数是有限的。

页面 http://www.testweb.cn/a.html 的代码：
``` bash
<iframe src="http://testweb.cn/b.html" id="myIframe" onload="test()" style="display: none;">
<script>
    // 2. iframe载入 "http://testweb.cn/b.html" 页面后会执行该函数
    function test() {
        // 3. 获取通过 http://testweb.cn/b.html 页面设置 hash 值
        var data = window.location.hash;
        console.log(data);
    }
</script>
```
页面 http://testweb.cn/b.html 的代码：
``` bash
<script type="text/javascript">
    // 1. 设置父页面的 hash 值
    parent.location.hash = "world";
</script>
```

八、 postMessage 跨域
window.postMessage(message，targetOrigin) 方法是 HTML5 新引进的特性，可以使用它来向其它的 window 对象发送消息，无论这个 window 对象是属于同源或不同源。这个应该就是以后解决 dom 跨域通用方法了。

调用 postMessage 方法的 window 对象是指要接收消息的那一个 window 对象，该方法的第一个参数 message 为要发送的消息，类型只能为字符串；第二个参数 targetOrigin 用来限定接收消息的那个 window 对象所在的域，如果不想限定域，可以使用通配符 *。

需要接收消息的 window 对象，可是通过监听自身的 message 事件来获取传过来的消息，消息内容储存在该事件对象的 data 属性中。

页面 http://www.testweb.cn/a.html 的代码：
``` bash
<iframe src="http://testweb.cn/b.html" id="myIframe" onload="test()" style="display: none;">
<script>
    // 1. iframe载入 "http://testweb.cn/b.html" 页面后会执行该函数
    function test() {
        // 2. 获取 http://testweb.cn/b.html 页面的 window 对象，
        // 然后通过 postMessage 向 http://testweb.cn/b.html 页面发送消息
        var iframe = document.getElementById('myIframe');
        var win = iframe.contentWindow;
        win.postMessage('我是来自 http://www.testweb.cn/a.html 页面的消息', '*');
    }
</script>
```
页面 http://testweb.cn/b.html 的代码：
``` bash
<script type="text/javascript">
    // 注册 message 事件用来接收消息
    window.onmessage = function(e) {
        e = e || event; // 获取事件对象
        console.log(e.data); // 通过 data 属性得到发送来的消息
    }
</script>
```
九、 websocket 跨域
Websocket是HTML5的一个持久化的协议，它实现了浏览器与服务器的全双工通信，同时也是跨域的一种解决方案。WebSocket和HTTP都是应用层协议，都基于 TCP 协议。但是 WebSocket 是一种双向通信协议，在建立连接之后，WebSocket 的 server 与 client 都能主动向对方发送或接收数据。同时，WebSocket 在建立连接时需要借助 HTTP 协议，连接建立好了之后 client 与 server 之间的双向通信就与 HTTP 无关了。

原生WebSocket API使用起来不太方便，我们使用Socket.io，它很好地封装了webSocket接口，提供了更简单、灵活的接口，也对不支持webSocket的浏览器提供了向下兼容。

我们先来看个例子：本地文件socket.html向localhost:3000发生数据和接受数据
``` bash
// socket.html
<script>
	// 高级api  不兼容  但是有一个socket.io这个库，是兼容的(一般用这个)
    let socket = new WebSocket('ws://localhost:3000');
    socket.onopen = function () {
      socket.send('hello');//向服务器发送数据
    }
    socket.onmessage = function (e) {
      console.log(e.data);//接收服务器返回的数据
    }
</script>
// server.js
let express = require('express');
let app = express();
let WebSocket = require('ws');//记得安装ws
let wss = new WebSocket.Server({port:3000});
wss.on('connection',function(ws) {
  ws.on('message', function (data) {
    console.log(data);
    ws.send('this is server')
  });
})
```