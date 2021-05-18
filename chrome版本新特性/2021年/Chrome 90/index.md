# A safer default for navigation: HTTPS

Chrome 90开始，将会默认使用HTTPS协议打开URL。  
目前这个特性还在灰度没有完全发布。


现象：
1. Chrome 90之前的版本，当我们输入 example.com，chrome会默认访问 http://example.com，服务端如果配置了重定向，则会重定向到https://example.com.  
2. Chrome 90则会默认访问 https://example.com。  

可以参考第一篇[了不起的Chrome浏览器：Chrome 90将默认使用HTTPS，Web更安全了](https://juejin.cn/post/6955700246303571999#heading-3)，文章里面对chrome 90版本之前 和 chrome 90这部分做了详细解说  


我这里要对这个问题进行进一步的提问：  
1. 为什么在chrome 输入 example.com，会默认自动访问 http://example.com; 但是部分网址（eg: baidu.com），输入 baidu.com，则会默认自动访问 https://baidu.com
   
所以，问题就是chrome 浏览器为我们的网址补充网络协议时，以依据怎么样的规则去补充的呢？   

可以参考第二篇[HTTP Strict Transport Security实战详解](https://www.cnblogs.com/sunsky303/p/8862600.html)









# 参考：
- [了不起的Chrome浏览器：Chrome 90将默认使用HTTPS，Web更安全了](https://juejin.cn/post/6955700246303571999#heading-3)  
- [HTTP Strict Transport Security实战详解](https://www.cnblogs.com/sunsky303/p/8862600.html)
- [上古面试题——浏览器地址栏输入后回车会发生什么](https://segmentfault.com/a/1190000021000934?utm_source=tag-newest)
- [关于HSTS安全协议的全面详细解析](https://blog.51cto.com/leoheng/2311422)