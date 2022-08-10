####  HTTP概念

- HTTP是面向于资源的应用层协议

- 两台计算机使用HTTP进行通信时，一定有一方担任客户端，一方担任服务端

- HTTP不保存状态，所以提供了cookie保存状态

##### HTTP1.1提供的方法

- GET、POST、PUT、HEAD、DELETE、OPTIONS、TRACE、CONNECT

- GET：获取资源。GET用来请求访问URI标识的资源
- POST：传输实体主体。将实体提交到指定的资源，例如表单上传，用于更新资源或创建资源
- PUT：传输实体主体。将文件保存到URI指定位置，用于更新资源；但由于HTTP/1.1的PUT方法本身不带验证机制，所以一般的web网站不使用该方法而是POST
- HEAD：获得报文首部
- DELETE：删除URI指定标识的资源。与PUT方法一样，由于本身无验证机制，一般不被支持
- OPTIONS：查询服务器针对请求URI标识的资源支持的方法
- TRACE：追踪路径
- CONNECION：用隧道协议连接代理。主要使用SSL和TSL将内容加密后经网络隧道传输

##### HTTP1.1方法的比较

- GET与POST

![图片来自于RUNOOB.COM](https://img-blog.csdnimg.cn/20200915204942992.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDQxOTk4NA==,size_16,color_FFFFFF,t_70#pic_center)

- POST与PUT
  - 幂等性（idemptence）：多次操作与一次操作的结果相同
  - 根据幂等性区分：POST是非幂等的，PUT是幂等的
  - 根据操作资源对象区分：POST作用于一个集合资源，put作用于一个具体的资源；简单说，对数据库进行操作时，POST更类似于add，直接对表进行操作，不需要定位到某一行，而PUT更类似于update，需要定位到具体资源

#### HTTP的组成

一个HTTP报文由三部分组成：报文首部+空行（CR+LF）+报文主体

##### 报文首部

请求报文首部和响应报文首部略有不同：

- 请求报文：**请求行**（请求方法+请求URI+HTTP版本）+首部字段（**请求首部字段**+通用首部字段+实体首部字段+其它）
- 响应报文：**状态行**（HTTP版本+状态码+原因短语）+首部字段（**响应首部字段**+通用首部字段+实体首部字段+其它）

空行：

- HTTP使用的是ASCII码规范，一个空行包括两个部分，回车（0x0d)与换行（0x0a)

报文主体：

- 报文主体包括实体首部字段+实体主体，所以一般情况下报文主体等于实体主体，但当实体主体进行编码后，要在实体首部字段中添加相应的字段，此时报文主体不等于实体主体



#### HTTP的首部字段

##### 通用首部字段

| 字段名            | 说明                                 | 字段值    |
| ----------------- | ------------------------------------ | --------- |
| Cache-Control     | 缓存管理                             | no-cache/ |
| Connection        | 逐跳首部、连接管理                   |           |
| Date              | 报文被创建的日期和时间               |           |
| Pragma            | HTTP/1.0向后兼容使用，唯一值no-cache |           |
| Trailer           | 报文末端的首部一览                   |           |
| Transfer-Encoding | 报文主体的传输编码方式               |           |
| Upgrade           | 升级为其它协议                       |           |
| Via               | 代理服务器的相关信息                 |           |
| Warning           | 缓存的错误信息                       |           |

##### 请求首部字段

| 字段名            | 说明                               | 字段值                                                       |
| ----------------- | ---------------------------------- | ------------------------------------------------------------ |
| Accept            | 用户代理可处理的媒体类型           | type/subtype;q=0.5（不指定权重时，默认为1.0）                |
| Accept-Charset    | 优先的字符集                       | ISO-8859-1;q=0.5（保留后三位小数）                           |
| Accept-Encoding   | 优先的内容编码                     | gzip、compress、deflate、identity                            |
| Accept-Language   | 优先的自然语言                     | cn...                                                        |
| Authorization     | Web认证信息                        |                                                              |
| Expect            | 期待服务器的特定行为               | 100-continue（状态码100）                                    |
| From              | 用户的邮件地址                     | 邮件                                                         |
| **Host**          | 请求资源所在服务器                 | 请求资源所在的互联网主机名和端口号                           |
| If-Match          | 比较实体标记                       | 匹配资源的ETag（服务器资源的ETag是否为这个，是就执行请求）   |
| If-Modified-Since | 比较资源的更新时间                 | date（若服务器在这个时间后更新过，则处理请求，否则返回304）  |
| If-None-Match     | 比较实体标记                       | 匹配资源的ETag（服务器是否有该ETag对应的资源，是就拒绝请求） |
| If-Range          | 资源未更新时发送实体Byte的范围请求 | ETag                                                         |
|                   |                                    |                                                              |