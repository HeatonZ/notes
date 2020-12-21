# HTTP缓存

## 缓存的优点
1. 减少了冗余的数据传递，节省宽带流量
2. 减少了服务器的负担，大大提高了网站性能
3. 加快了客户端加载网页的速度 这也正是HTTP缓存属于客户端缓存的原因。

## 缓存的类型
分为强制缓存和协商缓存
### 1. 强制缓存
客户端发起请求时，查找缓存是否有数据，有直接拿出，没有的话向服务器请求最新数据，存入缓存数据库
![image](https://raw.githubusercontent.com/HeatonZ/notes/main/HTTP%E7%BC%93%E5%AD%98/cache-force-1.png)
对于强制缓存，服务器响应的header中会用两个字段来表明——Expires和Cache-Control
#### Expires
HTTP1.0    

Exprires的值为服务端返回的数据到期时间。当再次请求时的请求时间小于返回的此时间，则直接使用缓存数据。  

但由于服务端时间和客户端时间可能有误差，这也将导致缓存命中的误差
#### Cache-Control
HTTP1.1 
- private：客户端可以缓存
- public：客户端和代理服务器都可以缓存 
- max-age=t：缓存内容将在t秒后失效 
- no-cache：需要使用协商缓存来验证缓存数据 no-store：所有内容都不会缓存。

### 2. 协商缓存
客户端发起请求时，查找缓存的标志，发送给服务器，服务器判断是否失效
![image](https://raw.githubusercontent.com/HeatonZ/notes/main/HTTP%E7%BC%93%E5%AD%98/cache-negotiate-1.png)
对于协商缓存缓存，服务器响应的header中会用两个字段来表明——Last-Modified和Etag

#### Last-Modified
HTTP1.0   

服务器在响应请求时，会告诉浏览器资源的最后修改时间。

请求对应的header是If-Modified-Since和if-Unmodified-Since
##### If-Modified-Since
- 如果真的被修改：那么开始传输响应一个整体，服务器返回：200 OK
- 如果没有被修改：那么只需传输响应header，服务器返回：304 Not Modified

##### if-Unmodified-Since
- 如果没有被修改:则开始`继续'传送文件: 服务器返回: 200 OK
- 如果文件被修改:则不传输,服务器返回: 412 Precondition failed (预处理错误)

一个资源如果改了之后回退，而缓存还是一样的，但是时间更改过却要重新获取，所以诞生Etag
#### Etag
HTTP1.1

服务器响应请求时，通过此字段告诉浏览器当前资源在服务器生成的唯一标识（生成规则由服务器决定）

请求对应的header是If-None-Match
- 不同，说明资源被改动过，则响应整个资源内容，返回状态码200。
- 相同，说明资源无修改，则响应header，浏览器直接从缓存中获取数据信息。返回状态码304.

但是实际应用中由于Etag的计算是使用算法来得出的，而算法会占用服务端计算的资源，所有服务端的资源都是宝贵的，所以就很少使用Etag了。

## 不同刷新的请求执行过程
1. 浏览器地址栏中写入URL，回车
浏览器发现缓存中有这个文件了，不用继续请求了，直接去缓存拿。（最快）
2. F5就是告诉浏览器，别偷懒，好歹去服务器看看这个文件是否有过期了。于是浏览器就胆胆襟襟的发送一个请求带上If-Modify-since。
3. Ctrl+F5
告诉浏览器，你先把你缓存中的这个文件给我删了，然后再去服务器请求个完整的资源文件下来。于是客户端就完成了强行更新的操作.
