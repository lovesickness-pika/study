### DNS域名解析

1. 检查浏览器缓存中是否缓存过该域名对应的IP地址
2. 如果在浏览器缓存中没有找到IP，那么将继续查找本机系统是否缓存过IP
3. 向本地域名解析服务系统发起域名解析的请求
4. 向根域名解析服务器发起域名解析请求
5. 根域名服务器返回gTLD域名解析服务器地址
6. 向gTLD服务器发起解析请求
7. gTLD服务器接收请求并返回Name Server服务器
8. Name Server服务器返回IP地址给本地服务器
9. 本地域名服务器缓存解析结果
10. 返回解析结果给用户



##### 图解

![image-20210824095139603](C:\Users\皮卡丘\AppData\Roaming\Typora\typora-user-images\image-20210824095139603.png)