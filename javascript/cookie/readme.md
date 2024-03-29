# 前端基础 -- 全面了解Cookie

### 一、Cookie的出现

  &emsp;&emsp;浏览器和服务器之间的通信少不了HTTP协议，但是因为HTTP协议是无状态的，所以服务器并不知道上一次浏览器做了什么样的操作，这样严重阻碍了交互式Web应用程序的实现。

  &emsp;&emsp;针对上述的问题，网景公司的程序员创造了Cookie。


### 二、Cookie的传输

  &emsp;&emsp;服务器端在实现Cookie标准的过程中，需要对任意HTTP请求发送Set-Cookie HTTP头作为响应的一部分：

```s
  Set-Cookie: name=value; expires=Tue, 03-Sep-2019 14:10:21 GMT; path=/; domain=.xxx.com;
```

  &emsp;&emsp;浏览器端会存储这样的Cookie，并且为之后的每个请求添加Cookie HTTP请求头发送回服务器：

```s
  Cookie: name=value
```

  &emsp;&emsp;服务器通过验证Cookie值，来判断浏览器发送请求属于哪一个用户。


### 三、浏览器中的Cookie

  &emsp;&emsp;浏览器中的Cookie主要由以下几部分组成：

  - 名称：Cookie**唯一**的名称，必须经过URL编码处理。（同名会出现覆盖的情况）
  - 值：必须经过URL编码处理。
  - 域（domain）：默认情况下cookie在当前域下有效，你也可以设置该值来确保对其子域是否有效。
  - 路径（path）：指定Cookie在哪些路径下有效，默认是当前路径下。
  - 失效时间（expires）：默认情况下，浏览器会话结束时会自动删除Cookie；也可以设置一个GMT格式的日期，指定具体的删除日期；如果设置的日期为以前的日期，那么Cookie会立即删除。
  - 安全标志（secure）：指定之后只允许Cookie发送给https协议。

  &emsp;&emsp;浏览器在发送请求时，只会将名称与值添加到请求头的Cookie字段中，发送给服务端。

  &emsp;&emsp;浏览器提供了一个非常蹩脚的API来操作Cookie：

```JavaScript
  document.cookie
```

  &emsp;&emsp;通过上述方法可以对该Cookie进行写操作，每一次只能写入一条Cookie字符串：

```JavaScript
  document.cookie = 'a=1; secure; path=/'
```

  &emsp;&emsp;通过该方法还可以进行Cookie的读操作：

```JavaScript
  document.cookie
  // "a=1"
```

  &emsp;&emsp;由于上述方法操作Cookie非常的不直观，一般都会写一些函数来简化Cookie读取、设置和删除操作。

  &emsp;&emsp;对于Cookie的设置操作中，需要以下几点：

  - 对于名称和值进行URL编码处理，也就是采用JavaScript中的encodeURIComponent()方法；
  - expires要求传入GMT格式的日期，需要处理为更易书写的方式，比如：设置秒数的方式；
  - 注意只有的属性名的secure；
  - 每一段信息需要采用**分号加空格**。

```JavaScript
function setCookie (key, value, attributes) {
  if (typeof document === 'undefined') {
    return
  }
  attributes = Object.assign({}, {
    path: '/'
  }, attributes)
  
  let { domain, path, expires, secure } = attributes

  if (typeof expires === 'number') {
    expires = new Date(Date.now() + expires * 1000)
  }
  if (expires instanceof Date) {
    expires = expires.toUTCString()
  } else {
    expires = ''
  }
  
  key = encodeURIComponent(key)
  value = encodeURIComponent(value)

  let cookieStr = `${key}=${value}`

  if (domain) {
    cookieStr += `; domain=${domain}`
  }

  if (path) {
    cookieStr += `; path=${path}`
  }

  if (expires) {
    cookieStr += `; expires=${expires}`
  }
  
  if (secure) {
    cookieStr += `; secure`
  }

  return (document.cookie = cookieStr)
}
```

  &emsp;&emsp;Cookie的读操作需要注意的是将名称与值进行URL解码处理，也就是调用JavaScript中的decodeURIComponent()方法：

```JavaScript
function getCookie (name) {
  if (typeof document === 'undefined') {
    return
  }
  let cookies = []
  let jar = {}
  document.cookie && (cookies = document.cookie.split('; '))

  for (let i = 0, max = cookies.length; i < max; i++) {
    let [key, value] = cookies[i].split('=')
    key = decodeURIComponent(key)
    value = decodeURIComponent(value)
    jar[key] = value
    if (key === name) {
      break
    }
  }

  return name ? jar[name] : jar
}
```

  &emsp;&emsp;最后一个清除的方法就更加简单了，只要将失效日期（expires）设置为过去的日期即可：

```JavaScript
function removeCookie (key) {
  setCookie(key, '', { expires: -1 })
}
```

  &emsp;&emsp;介绍Cookie基本操作的封装之后，还需要了解浏览器为了限制Cookie不会被恶意使用，规定了Cookie所占磁盘空间的大小以及每个域名下Cookie的个数。

  &emsp;&emsp;为了绕开单域名下Cookie个数的限制，开发人员还创造了一种称为subcookie的概念，这里就不在赘述了，可以参考【JavaScript高级程序设计第23章 p633】。

### 四、服务端的Cookie

  &emsp;&emsp;相比较浏览器端，服务端执行Cookie的写操作时，是将拼接好的Cookie字符串放入响应头的Set-Cookie字段中；执行Cookie的读操作时，则是解析HTTP请求头字段Cookie中的键值对。

  &emsp;&emsp;与浏览器最大的不同，在于服务端对于Cookie的安全性操碎了心。

##### signed

  &emsp;&emsp;当设置signed=true时，服务端会对该条Cookie字符串生成两个Set-Cookie响应头字段：

```s
  Set-Cookie: lastTime=2019-03-05T14:31:05.543Z; path=/; httponly
  Set-Cookie: lastTime.sig=URXREOYTtMnGm0b7qCYFJ2Db400; path=/; httponly
```

  &emsp;&emsp;这里通过再发送一条以.sig为后缀的名称以及对值进行加密的Cookie，来验证该条Cookie是否在传输的过程中被篡改。

##### httpOnly

  &emsp;&emsp;服务端Set-Cookie字段中新增httpOnly属性，当服务端在返回的Cookie信息中含有httpOnly字段时，开发者是不能通过JavaScript来操纵该条Cookie字符串的。

  &emsp;&emsp;这样做的好处主要在于面对XSS（Cross-site scripting）攻击时，黑客无法拿到设置httpOnly字段的Cookie信息。

  &emsp;&emsp;此时，你会发现localStorage相比较Cookie，在XSS攻击的防御上就略逊一筹了。

##### sameSite

  &emsp;&emsp;在介绍这个新属性之前，首先你需要明白：当用户从a.com发起b.com的请求也会携带上Cookie，而从a.com携带过来的Cookie称为第三方Cookie。

  &emsp;&emsp;虽然第三方Cookie有一些好处，但是给CSRF（Cross-site request forgrey）攻击的机会。

  &emsp;&emsp;为了从根源上解决CSRF攻击，sameSite属性便闪亮登场了，它的取值有以下几种：

  - strict：浏览器在任何跨域请求中都不会携带Cookie，这样可以有效的防御CSRF攻击，但是对于有多个子域名的网站采用主域名存储用户登录信息的场景，每个子域名都需要用户重新登录，造成用户体验非常的差。
  - lax：相比较strict，它允许从三方网站跳转过来的时候使用Cookie。

  &emsp;&emsp;为了方便大家理解sameSite的实际效果，可以看这个例子：

```s
  // a.com 服务端会在访问页面时返回如下Cookie

  cookies.set('foo', 'aaaaa')
  cookies.set('bar', 'bbbbb')
  cookies.set('name', 'cccccc')

  // b.com 服务端会在访问页面时返回如下Cookie
  cookies.set('foo', 'a', { sameSite: 'strict' })
  cookies.set('bar', 'b', { sameSite: 'lax' })
  cookies.set('baz', 'c')
```

  &emsp;&emsp;如何现在用户在a.com中点击链接跳转到b.com，它的请求头是这样的：

```s
  Request Headers
  Cookie: bar=b; baz=c
```

### 五、网站性能优化

  &emsp;&emsp;Cookie在服务端和浏览器的通信中，主要依靠HTTP的响应头和请求头传输的，所以Cookie会占据一定的带宽。

  &emsp;&emsp;前面提到浏览器会为每一次HTPP请求自动携带上Cookie信息，但是对于同站内的静态资源，服务器并不需要处理其携带的Cookie，这无形中便浪费了带宽。

  &emsp;&emsp;在最佳实践中，一般都会将静态资源部署到独立的域名上，从而可以避免无效Cookie的影响。
