### HTTP状态码

#### 类别

|      | 类别                            | 原因短语                   |
| ---- | ------------------------------- | -------------------------- |
| 1XX  | Informational（信息性状态码）   | 接收的请求正在处理         |
| 2XX  | Success（成功状态码）           | 请求正常处理完毕           |
| 3XX  | Redirection（重定向状态码）     | 需要进行附加操作以完成请求 |
| 4XX  | Clien Error（客户端错误状态码） | 服务器无法处理请求         |
| 5XX  | Server Error （服务器错误代码） | 服务器处理请求出错         |



#### 常见的状态码

##### 1XX

- 100	服务器处理中，要求客户端继续发送请求

##### 2XX

- 200	响应成功

##### 3XX

- 301	**永久重定向**	应该重新保存书签
- 302    **临时性重定向** -- 尽量使用永久重定向，防止网址劫持
- 303    **临时性重定向**并要求使用**GET方法**访问
- 304    客户端发送带附带条件的请求时，服务端允许访问资源，但是发生**请求不满足条件**
- 307    临时性重定向 不允许POST方法变成GET方法

##### 4XX

- 400	语法错误
- 403    拒绝访问，权限不足，比如访问WEB-INFO文件下的资源
- 404    页面丢失
- 405    方法错误，使用GET访问POST

##### 5XX

- 500    服务器内部错误
- 503    服务器超载或正在维修





##### 网址劫持

```
从网址A 做一个302 重定向到网址B 时，主机 服务器的隐含意思是网址A 随时有可能改主意，重新显示本身的内容或转向其他的地方。大部分的搜索引擎在大部分情况下，当收到302 重定向时，一般只要去抓取目标网址就可以了，也就是说网址B。如果搜索引擎在遇到302 转向时，百分之百的都抓取目标网址B 的话，就不用担心网址URL 劫持了。问题就在于，有的时候搜索引擎，尤其是Google，并不能总是抓取目标网址。比如说，有的时候A 网址很短，但是它做了一个302 重定向到B 网址，而B 网址是一个很长的乱七八糟的URL 网址，甚至还有可能包含一些问号之类的参数。很自然的，A 网址更加用户友好，而B 网址既难看，又不用户友好。这时Google 很有可能会仍然显示网址A。由于搜索引擎排名算法只是程序而不是人，在遇到302 重定向的时候，并不能像人一样的去准确判定哪一个网址更适当，这就造成了网址URL 劫持的可能性。也就是说，一个不道德的人在他自己的网址A 做一个302 重定向到你的网址B，出于某种原因， Google 搜索结果所显示的仍然是网址A，但是所用的网页内容却是你的网址B 上的内容，这种情况就叫做网址URL 劫持。你辛辛苦苦所写的内容就这样被别人偷走了。302 重定向所造成的网址URL 劫持现象，已经存在一段时间了。不过到目前为止，似乎也没有什么更好的解决方法。在正在进行的谷歌数据中心转换中，302 重定向问题也是要被解决的目标之一。从一些搜索结果来看，网址劫持现象有所改善，但是并没有完全解决。
```

