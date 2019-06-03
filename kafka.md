# kafka为什么这么快

- [为什么kafka那么快](https://mp.weixin.qq.com/s?__biz=MzIxMjAzMDA1MQ==&mid=2648945468&idx=1&sn=b622788361b384e152080b60e5ea69a7#rd&utm_source=tuicool&utm_medium=referral)
- [什么是Zero-Copy？](https://blog.csdn.net/u013256816/article/details/52589524)
- [Kafka副本同步机制理解](https://blog.csdn.net/lizhitao/article/details/51718185)
- [Kafka深度解析](http://www.jasongj.com/2015/01/02/Kafka%E6%B7%B1%E5%BA%A6%E8%A7%A3%E6%9E%90/)



# 幂等性
### 唯一id去重


## kafka快速的原因

### 顺序写入
磁盘每次写入都会寻址->写入，其中寻址是最耗时的，所以对磁盘来说随机I/O是最慢的，为了提高磁盘的读写速度，kafka就是使用顺序I/O。
![http://mmbiz.qpic.cn/mmbiz/nfxUjuI2HXjiahgInoFXLfVoghamdPiaBafMYrFo4rFUykuic1Ks0P5DBzxjVfzgYlscCWeicNnE3HrSKxJkCxOcEw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1](http://mmbiz.qpic.cn/mmbiz/nfxUjuI2HXjiahgInoFXLfVoghamdPiaBafMYrFo4rFUykuic1Ks0P5DBzxjVfzgYlscCWeicNnE3HrSKxJkCxOcEw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)