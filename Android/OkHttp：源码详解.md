# OkHttp：源码详解

> 源码基于okhttp3 java版本：3.14.x

在《OkHttp：基本使用详解》和 《OkHttp：进阶详解》中描述了OkHttp的基本使用方法。自此我们了解了OkHttp的基础使用和了解了使用中设计到的相关概念。

接下来我们通过《OkHttp：源码详解》一系列文章从源码的角度来阐述OkHttp的工作流程和设计思想。

本系列文章分为六部分

OkHttp：源码详解之核心流程（一）

OkHttp：源码详解之重试重定向拦截器（二）

OkHttp：源码详解之桥拦截器（三）

OkHttp：源码详解之缓存拦截器（四）

OkHttp：源码详解之连接拦截器（五）

OkHttp：源码详解之请求服务拦截器（六）

这六篇文章涵盖了OkHttp源码的核心流程，尽可能详细的去描述，力求读者在阅读并理解后即可对OkHttp的设计思想和工作流程有一个清晰的认知。

强烈建议读者配合OkHttp的源码一起阅读，更易于理解（源码较多，有些部分无法全部陈列在文章中）。

当然这只是一部分主干源码，想要了解其他更加细致的实现还请自行阅读源码。

关于您的疑问，欢迎在评论区留言，我们一起分析探讨。

---

参考资料：

- [你想要的系列：网络请求框架OkHttp3全解系列](https://juejin.im/post/6873476209737629709/#heading-7)
- [Android |《看完不忘系列》之okhttp](https://juejin.im/post/6856966817844625415)
- [OkHttp3源码解析(整体流程)](https://mp.weixin.qq.com/s/gkIs54E-nT4ZqO21x02x7w)
- [Android 网络框架之OkHttp源码解析](https://juejin.im/post/6873476209737629709/#heading-7)

---

**我的[学习笔记](https://github.com/zhangzhian/LearningNotes)，欢迎star和fork**

**欢迎关注我的公众号，持续分析优质技术文章**![欢迎关注我的公众号](https://img-blog.csdnimg.cn/20190906092641631.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5,size_16,color_FFFFFF,t_70)