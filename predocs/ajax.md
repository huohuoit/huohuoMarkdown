## Ajax 基础

### 传统网站中存在的问题

网速慢的情况下，页面加载时间长，用户只能等待

表单提交后，如果一项内容不合格，需要重新填写所有表单内容

页面跳转，重新加载页面，造成资源浪费，增加用户等待时间

### Ajax

它是浏览器提供的一套方法，可以实现页面无刷新更新数据，提高用户浏览网站应用的体验。

`Ajax`技术需要运行在网站环境中才能生效

#### 应用

页面上拉加载更多数据

列表数据无刷新分页

表单项离开焦点数据验证

搜索框提示文字下拉列表

### Ajax 运行原理与实现

`Ajax`相当于浏览器发送请求与接收响应的代理人，以实现在不影响用户浏览页面的情况下，局部更新页面数据，从而提高用户体验。

#### Ajax 的实现步骤

1. 创建`Ajax`对象

```js
var xhr = new XMLHttpRequest();
```

2. 告诉`Ajax`请求地址以及请求方式

```js
xhr.open("get", "http://www.example.com");
```

3. 发送请求

```js
xhr.send();
```

4. 获取服务器端给与客户端的响应数据

```js
xhr.onload = function () {
  console.log(xhr.responseText);
};
```

#### 服务器端响应的数据格式

在真实的项目中，服务器端大多数情况下会以`JSON`对象作为响应数据的格式。当客户端拿到响应数据时，要将`JSON`数据和`HTML`字符串进行拼接，然后将拼接的结果展示在页面中。

在`http`请求与响应的过程中，无论是请求参数还是响应内容，如果是对象类型，最终都会被转换为对象字符串进行传输。

```js
JSON.parse(); // 将 json 字符串转换为 json 对象
```

#### 请求参数传递

GET 请求方式

```js
xhr.open("get", "http://www.example.com?name=zhangsan&age=20");
```

POST 请求方式

```js
xhr.setRequestHeader('Content-Type','application/x-www-form-urlencoded') xhr.send('name=zhangsan&age=20');
```

#### 请求报文

在`HTTP`请求和响应的过程中传递的数据块就叫报文，包括要传送的数据和一些附加信息，这些数据和信息要遵守规定好的格式。

#### 请求参数的格式

1. application/x-www-form-urlencoded

```js
name=zhangsan&age=20&sex=男
```

2. application/json

```js
{name: 'zhangsan', age: '20', sex: '男'}
```

3.在请求头中指定`Content-Type`属性的值是`application/json`，告诉服务器端当前请求参数的格式是`json`。

```js
JSON.stringify(); // 将 json 对象转换为 json 字符串
```

注意：`get`请求是不能提交`json`对象数据格式的，传统网站的表单提交也是不支持`json`对象数据格式的。

#### 获取服务器端的响应

1.`Ajax`状态码

在创建`ajax`对象，配置`ajax`对象，发送请求，以及接收完服务器端响应数据，这个过程中的每一个步骤都会对应一个数值，这个数值就是`ajax`状态码。

- 0：请求未初始化(还没有调用`open()`)

- 1：请求已经建立，但是还没有发送(还没有调用`send()`)

- 2：请求已经发送

- 3：请求正在处理中，通常响应中已经有部分数据可以用了

- 4：响应已经完成，可以获取并使用服务器的响应了

```js
xhr.readyState; // 获取 Ajax 状态码
```

2.`onreadystatechange`事件（当`Ajax`状态码发生变化时将自动触发该事件）改为用 `onload`

在事件处理函数中可以获取`Ajax`状态码并对其进行判断，当状态码为`4`时就可以通过`xhr.responseText`获取服务器端的响应数据了。

```js
// 当 Ajax 状态码发生变化时

xhr.onreadystatechange = function () {
  // 判断当 Ajax 状态码为 4 时

  if (xhr.readyState == 4) {
    // 获取服务器端的响应数据

    console.log(xhr.responseText);
  }
};
```

#### Ajax 错误处理

1. 网络畅通，服务器端能接收到请求，服务器端返回的结果不是预期结果。

可以判断服务器端返回的状态码，分别进行处理。`xhr.status`获取`http`状态码

2. 网络畅通，服务器端没有接收到请求，返回`404`状态码。

检查请求地址是否错误。

3. 网络畅通，服务器端能接收到请求，服务器端返回`500`状态码。

服务器端错误，找后端程序员进行沟通。

4. 网络中断，请求无法发送到服务器端。

会触发 xhr 对象下面的`onerror`事件，在`onerror`事件处理函数中对错误进行处理。

#### 低版本 IE 浏览器的缓存问题

问题：在低版本的 IE 浏览器中，`Ajax`请求有严重的缓存问题，即在请求地址不发生变化的情况下，只有第一次请求会真正发送到服务器端，后续的请求都会从浏览器的缓存中获取结果。即使服务器端的数据更新了，客户端依然拿到的是缓存中的旧数据。

解决方案：在请求地址的后面加请求参数，保证每一次请求中的请求参数的值不相同

```js
xhr.open("get", "<http://www.example.com?t>=" + Math.random());
```

### Ajax 异步编程

#### 同步

一个人同一时间只能做一件事情，只有一件事情做完，才能做另外一件事情。

落实到代码中，就是上一行代码执行完成后，才能执行下一行代码，即代码逐行执行。

#### 异步

一个人一件事情做了一半，转而去做其他事情，当其他事情做完以后，再回过头来继续做之前未完成的事情。

落实到代码上，就是异步代码虽然需要花费时间去执行，但程序不会等待异步代码执行完成后再继续执行后续代码，而是直接执行后续代码，当后续代码执行完成后再回头看异步代码是否返回结果，如果已有返回结果，再调用事先准备好的回调函数处理异步代码执行的结果。

#### Ajax 封装

问题：发送一次请求代码过多，发送多次请求代码冗余且重复。

解决方案：将请求代码封装到函数中，发请求时调用函数即可。

```js
ajax({
  type: "get",
  url: "http://www.example.com",
  success: function (data) {
    console.log(data);
  },
});
```

## Ajax 编程

### 2.1 模板引擎

#### 2.1.1 作用

使用模板引擎提供的模板语法，可以将数据和`HTML`拼接起来

[官方地址](https://aui.github.io/art-template/zh-cn/index.html)

#### 使用步骤

1. 下载`art-template`模板引擎库文件并在`HTML` 页面中引入库文件

```js
<script src="./js/template-web.js"></script>
```

2. 准备`art-template`模板

```js
<script id="tpl" type="text/html">
  <div class="box"></div>
</script>
```

3. 告诉模板引擎将哪一个模板和哪个数据进行拼接

```js
var html = template("tpl", { username: "zhangsan", age: "20" });
```

4. 将拼接好的`html`字符串添加到页面中

```js
document.getElementById("container").innerHTML = html;
```

5. 通过模板语法告诉模板引擎，数据和`html`字符串要如何拼接

```js
<script id="tpl" type="text/html">
  <div class="box"> {{ username }} </div>
</script>
```

### FormData

#### FormData 对象的作用

模拟`HTML`表单，相当于将`HTML`表单映射成表单对象，自动将表单对象中的数据拼接成请求参数的格式。

异步上传二进制文件

#### FormData 对象的使用

1. 准备`HTML`表单

``html

<form id="form">

<input type="text" name="username" />

<input type="password" name="password" />

<input type="button"/>

</form>
```

2. 将`HTML` 表单转化为`formData`对象

```js
var form = document.getElementById("form");

var formData = new FormData(form);
```

3. 提交表单对象

```js
xhr.send(formData);
```

注意：

`Formdata`对象不能用于`get`请求，因为对象需要被传递到`send`方法中，而`get`请求方式的请求参数只能放在请求地址的后面。服务器端`bodyParser`模块不能解析`formData`对象表单数据，我们需要使用`formidable`模块进行解析。

#### FormData 对象的实例方法

1. 获取表单对象中属性的值

```js
formData.get("key");
```

2. 设置表单对象中属性的值

```js
formData.set("key", "value");
```

3. 删除表单对象中属性的值

```js
formData.delete("key");
```

4. 向表单对象中追加属性值

```js
formData.append("key", "value");
```

注意：`set`方法与`append`方法的区别是，在属性名已存在的情况下，`set`会覆盖已有键名的值，`append`会保留两个值。

#### FormData 二进制文件上传

```js
<input type="file" id="file" />;

var file = document.getElementById("file");
// 当用户选择文件的时候
file.onchange = function () {
  // 创建空表单对象
  var formData = new FormData();
  // 将用户选择的二进制文件追加到表单对象中
  formData.append("attrName", this.files[0]);
  // 配置 ajax 对象，请求方式必须为 post
  xhr.open("post", "www.example.com");
  xhr.send(formData);
};
// 当用户选择文件的时候
file.onchange = function () {
  // 文件上传过程中持续触发 onprogress 事件
  xhr.upload.onprogress = function (ev) {
  // 当前上传文件大小/文件总大小 再将结果转换为百分数
  // 将结果赋值给进度条的宽度属性
  bar.style.width = (ev.loaded / ev.total) \* 100 + '%';
  }
}
```

#### FormData 文件上传图片即时预览

```js
xhr.onload = function () {
  var result = JSON.parse(xhr.responseText);
  var img = document.createElement("img");
  img.src = result.src;
  img.onload = function () {
    document.body.appendChild(this);
  };
};
```

### 同源政策

#### Ajax 请求限制

Ajax 只能向自己的服务器发送请求。比如现在有一个 A 网站、有一个 B 网站，A 网站中的 HTML 文件只能向 A 网站服务器中发送 Ajax 请求，B 网站中的 HTML 文件只能向 B 网站中发送 Ajax 请求，但是 A 网站是不能向 B 网站发送 Ajax 请求的，同理，B 网站也不能向 A 网站发送 Ajax 请求。

#### 同源

如果两个页面拥有相同的协议、域名和端口，那么这两个页面就属于同一个源，其中只要有一个不相同，就是不同源。

#### 同源政策的目的

同源政策是为了保证用户信息的安全，防止恶意的网站窃取数据。最初的同源政策是指 A 网站在客户端设置的 Cookie，B 网站是不能访问的。

随着互联网的发展，同源政策也越来越严格，在不同源的情况下，其中有一项规定就是无法向非同源地址发送 Ajax 请求，如果请求，浏览器就会报错。

#### 使用 JSONP 解决同源限制问题

`jsonp`是 json with padding 的缩写，它不属于 Ajax 请求，但它可以模拟 Ajax 请求。

1. 将不同源的服务器端请求地址写在 script 标签的`src`属性中

```js
<script src="www.example.com"></script>

<script src=“<https://cdn.bootcss.com/jquery/3.3.1/jquery.min.js>"></script>
```

2. 服务器端响应数据必须是一个函数的调用，真正要发送给客户端的数据需要作为函数调用的参数

```js
const data = 'fn({name: "张三", age: "20"})';

res.send(data);
```

3. 在客户端全局作用域下定义函数 fn

```js
function fn(data) {}
```

4. 在 fn 函数内部对服务器端返回的数据进行处理

```js
function fn(data) {
  console.log(data);
}
```

代码优化：

客户端需要将函数名称传递到服务器端。

将 script 请求的发送变成动态请求。

封装 jsonp 函数，方便请求发送。

服务器端代码优化之`res.jsonp`方法。

#### CORS 跨域资源共享

1. CORS：

全称为`Cross-origin resource sharing`，即跨域资源共享，它允许浏览器向跨域服务器发送 Ajax 请求，克服了 Ajax 只能同源使用的限制。

```bash
Access-Control-Allow-Origin: 'http://localhost:3000'

Access-Control-Allow-Origin: '\*'
```

2. Node 服务器端设置响应头示例代码：

```js
app.use((req, res, next) => {
  res.header("Access-Control-Allow-Origin", "*");
  res.header("Access-Control-Allow-Methods", "GET, POST");
  next();
});
```

3. 访问非同源数据 服务器端解决方案

同源政策是浏览器给予 Ajax 技术的限制，服务器端是不存在同源政策限制。

4. `withCredentials`属性

在使用 Ajax 技术发送跨域请求时，默认情况下不会在请求中携带 cookie 信息。

`withCredentials`：指定在涉及到跨域请求时，是否携带 cookie 信息，默认值为 false

`Access-Control-Allow-Credentials`：true 允许客户端发送请求时携带 cookie
