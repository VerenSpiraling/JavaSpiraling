## Session Cookie 和 Token



### 发展史

web刚刚出来的时候，基本上就是文档的浏览，不需要记录谁在某一时间段里浏览量什么文件，每次请求都是一个新的HTTP协议。就是请求加响应。



随着交互式的Web应用兴起，出现了很多需要登录的网站，这就需要我们去记住哪些人注册或登录系统，记住购物车中放的商品，也就是说我们得把每个人都区分开，况且，HTTP请求是无状态的，所以会话标识（session id）就出来了，给每个人分发一个随机的字符串，每个人收到的都不一样，这样，当发起HTTP请求的时候，把这个字符串一并传过去，这样就能区分开谁是谁了。



这样会引发一个问题：我们自己只需要保存自己的session id，但服务器需要保存所有人的session id，当访问服务器多了，服务器就会超负荷。



这对服务器来说是一个巨大的开销，严重限制了服务器的扩展能力。假设，我用两个机器组成了一个集群，zx通过机器A登录了系统，那么session id就会保存在机器A上，如果zx的下一次请求被转发到机器B了，这怎么办？机器B可没有zx的session id啊。



有时候会采用一种办法：session sticky，就是让zx的请求一直粘连在机器A上，但保不齐哪天机器A突然挂掉了，还得转到机器B上。



做session的复制？ 把session id在两个机器之间搬来搬去，怕不是要累死了。



直到后来，出现了一个叫Memcached的，它把session id集中存储到一个地方，所有的机器都来访问这个地方的数据，这样一来，就不用复制了，**但是**，这增加了单点失败的可能性，要是那个负责session 的机器挂了，所有人就得重新登录一遍，估计要被人唾弃死。



不是没尝试过把这个单点的机器也搞出集群来，增加可靠性，但无论如何，这小小的session 对我来说就是一个沉重的负担。



于是呢，就有人在思考，我们为什么要保存这个万恶的session呢，只让每个客户端去保存不就好了吗？

可是呢，如果不报错这些session id，又怎么去验证客户端发给我的session id的确是我生成的呢？如果不去验证，我们都不知道他们是不是合法登录的用户，那些不怀好意的家伙们呢就可以伪造session id，为所欲为了。

**验证**是一个关键点！

如果说，zx登录了系统，我给他发一个令牌（token），里面包含了zx的user id，下一次zx再次通过HTTP请求访问我的时候，把这个token通过HTTP header带过来不就行了吗？

不过这和session id没有本质区别呀，任何人都可以伪造，所以我们得想点别的办法，让别人伪造不了。

要不加个密码吧，比如说，我通过HMAC-SHA256算法，加上一个只有我才知道的密钥，对数据做一个签名，把这个签名和数据一起作为token，由于密钥别人都不知道，就无法伪造token了。

![null](https://mmbiz.qpic.cn/mmbiz_png/iaIdQfEric9TxOoDdMlvo5ZiaN3U4adkCfZ1AGg8wzzdq6GrrYSloPpK6gXwR2S1ib2aficg9IWcQVyAfS4vmNY4Zew/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这个token我不保存，当zx把这个token给我发过来的时候，我再用同样的MHAC-SHA256算法和同样的密钥，对数据再计算一次签名，和token中的签名做个比较，如果相同，我就知道zx已经登陆过了，并且可以直接渠道zx的user id，如果不相同，数据部分肯定被人篡改过，我就告诉发送者，对不起，没有认证。

![null](https://mmbiz.qpic.cn/mmbiz_png/iaIdQfEric9TxOoDdMlvo5ZiaN3U4adkCfZrIwXtRWdhuWQMtgVFusZmW7P6vobEJmDUqc6JMQuQo9ibHrZMwjicHlw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Token中的数据是明文保存的（虽然会用Base64做下编码，但那不是加密），还是可以被别人看到的，所以不能保存敏感信息。

当然，如果token被别人偷走了，那我也没有办法，只要是签名匹配，我就认为他是合法用户，这起始和一个人的session id被偷走是一样的。



这样一来。我就不用保存session id了，只是生成token和验证token。

用cpu的计算时间获取了我的session存储空间。



解除了session id 这个负担，服务器集群就可以随便水平扩展，用户的访问量增大，直接加机器就行。



### Cookie

cookie 是一个非常具体的东西，指的就是浏览器中永久存储的一种数据，仅仅就是浏览器实现的一种数据存储功能。

cookie由服务器生成

发送给浏览器

浏览器把cookie以key-value的形式保存到某个目录下的文本文件内

下一次请求相同网站时会把该cookie发送给服务器。



由于cookie 是存储在客户端上

浏览器会加入一些限制确保cookie不会被恶意使用

同时不会占据太多的磁盘空间

因此，每个域的cookie数量都是有限的。



### Session

