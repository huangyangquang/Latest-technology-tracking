# 浅析chrome新特性之默认使用HTTPS，追溯源头至HSTS

<!-- TOC -->

- [浅析chrome新特性之默认使用HTTPS，追溯源头至HSTS](#浅析chrome新特性之默认使用https追溯源头至hsts)
- [参考：](#参考)
- [社交信息 / Social Links:](#社交信息--social-links)
  - [(Welcome to pay attention, 欢迎关注)](#welcome-to-pay-attention-欢迎关注)

<!-- /TOC -->
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


# 社交信息 / Social Links:
 ## (Welcome to pay attention, 欢迎关注)
<p>Github：
    <a target="_blank" href="https://github.com/huangyangquang">@huangyangquang</a> 
    | 
    <a target="_blank" href="https://github.com/huangyangquang/Latest-technology-tracking">最新技术追踪</a> 
    |
    <a target="_blank" href="https://github.com/huangyangquang/Algorithm">javascript版算法</a> 
    |
    <a target="_blank" href="https://github.com/huangyangquang/DEMO">早期前端知识总结 + 案例</a> (欢迎**star**)  
</p>
<p>Social：
    <a target="_blank" href="https://weibo.com/u/6385661354">新浪微博</a> 
    | 
    <a target="_blank" href="https://www.zhihu.com/people/cclv3">知乎</a>
    | 
    <a target="_blank" href="https://juejin.cn/user/2735240661699181">掘金</a>
    | 
    <a target="_blank" href="https://segmentfault.com/u/c_z7wgq/articles">是否</a>
</p>
<p>E-mail： 572518423@qq.com</p>
<p>Old Blog：
    <a target="_blank" href="https://blog.csdn.net/huangyangquan3?type=blog">CSDN</a>
</p>
<p>
    微信公众号：前端学长Joshua
</p>
<img src="../../../static/img/wechatQrCode.jpg" width="50%">

