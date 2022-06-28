> `URL`参数（也称为查询参数或查询字符串）是附加到`URL`末尾的一组键值解析。它们用于在页面之间或通过`URL`从客户端向服务器发送少量数据。

```js
const queryString = window.location.search;
const urlParams = new URLSearchParams(queryString);

urlParams.get("color"); // purple
urlParams.getAll("size"); // ['M', 'L']
```

## URL 参数结构

- 查询参数与带有（问号）的`URL`路径分开
- 第一个参数之后的每个参数都用&（与号）连接

在这种情况下，查询字符串是`color=purple`。

有些字符不能成为 URL 的一部分（例如空格），而其他一些字符在`URL`中具有特殊含义（字符`#`）。这种类型的字符需要编码（例如空格编码为`%20`）。

## 获取 URL 参数

您可以使用`window.location.search`

```js
const queryString = window.location.search;
```

然后，您可以使用以下方法解析查询字符串的参数 URLSearchParams：

```js
const urlParams = new URLSearchParams(queryString);
```

现在您可以使用`URLSearchParams.get()`来获取与给定搜索参数关联的第一个值

```js
urlParams.get("color"); // purple
urlParams.get("size"); // M
urlParams.get("nothing"); // empty string
```

## 获取 URL 参数的所有值

现在您可以使用`URLSearchParams.getAll()`来获取与给定搜索参数关联的所有值

```js
urlParams.getAll("color"); // ['purple']
urlParams.getAll("size"); // ['M', 'L']
```

## 检查 URL 参数是否存在

您可以使用`URLSearchParams.has()`来检查给定参数是否存在。

```js
urlParams.has("color"); // true
urlParams.has("size"); // true
urlParams.has("nothing"); // false
```

## 遍历 URL 参数

遍历所有键：

```js
const keys = urlParams.keys();

for (const key of keys) {
  console.log(key);
}
```

遍历所有值：

```js
const values = urlParams.values();

for (const value of values) {
  console.log(value);
}
```

遍历所有键/值对：

```js
const entries = urlParams.entries();

for (const entry of entries) {
  console.log(`${entry[0]} = ${entry[1]}`);
}
```

## 对于 Internet Explorer

IE 不支持，因此`URLSearchParams`您需要解析`URL`并获取查询参数。

```js
function getParameterByName(name) {
  cont url = window.location.search;
  name = name.replace(/[\[\]]/g, '\\$&');

  let regex = new RegExp('[?&]' + name + '(=([^&#]*)|&|#|$)');
  let results = regex.exec(url);

  if (!results) {
    return null;
  }

  if (!results[2]) {
    return '';
  }

  return decodeURIComponent(results[2]);
}
```

现在您可以使用`getParameterByName`来获取与给定搜索参数关联的第一个值：

```js
getParameterByName("color"); // purple
getParameterByName("size"); // M
getParameterByName("nothing"); // null
```
