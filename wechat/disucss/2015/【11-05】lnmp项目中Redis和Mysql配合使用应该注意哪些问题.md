> 转载自：<http://mp.weixin.qq.com/s?__biz=MzA3MDA2MjE2OQ==&amp;amp;mid=400403375&amp;amp;idx=1&amp;amp;sn=dae58e70526da962744a4bee2e7618de&amp%20;amp;scene=1&amp;amp;srcid=11058TEUUDj72vcFbeWc6SsI#rd>

#### 今日话题

lnmp项目中Redis和Mysql配合使用应该注意哪些问题？ - 刺客

1. 我这边因为项目小，主要用redis充当mysql的缓存使用，把活跃数据预读到redis中，这样绝大部分的请求处理都不用去查mysql，然后再触发数据同步，不知道这样是否科学，现在有个数据同步的疑问，怎样去触发redis和mysql同步而又不影响正常的客户端请求？毫无相关经验，还望指教 - 刺客
2. 不会影响客户端请求的吧，redis不存在就穿透数据库了，不知道你说的影响客户端请求指的是什么 - viktor  
回: MySQL udf 解析binlog到redis，不过缓存用redis大材小用了吧 - 金灶沐  
回: 这是要把mysql全部复制到redis？ - viktor  
回: …不用 触发就放呗 已经在redis里了，在淘汰被 哈哈哈 - 金灶沐  
回: 他说的是活跃数据，没弄明白他问的是什么，内存满了淘汰可能新数据进来踢掉热数据 lru的缺点 - viktor  
回: 你考虑的太多了，你当哪的量都那么大呢，t呗，那说明的加机器了，扩大内存，压缩…都行… 不动手光想，上来120G内存，玩呗，郭老师…😁 你大数字人太牛逼，上来就过亿流量看过程… 看方案 - 金灶沐  
回: 防止缓存穿透db我这边做了 - viktor  
回: 那哥们说的穿透是不是说缓存数据没了，请求全打到db里面，如果redis数据是用触发式更新是会碰上这个情况 - 轩脉刃  
回: 是指redis 挂了没有cache层 直接打db - 北极宫爵  
3. redis有就用 redis没有就mysql？请问。热数据  比如刚发 点击量很高 是如何加到redis的？  mysql触发？ - eli 🎧  
回: 一般都是先写cache 然后在异步写db，你业务来定的呀 - 北极宫爵  
回: 如果先缓存 如何判断这条数据是热数据？比如博客站。有写明星 一篇博客发了马上暴增连击量。这时候这篇博客存redis合适。而其他访问量低的 写redis有些浪费吧。静态就好了吧 - eli 🎧
4. 我觉得有两种模式做。 1. php 控制逻辑是取nosql还是sql更新nosql  缺点分离不够彻底 2. php不管nosql和sql之间的关系 使用单独的机制做nosql sql间的数据同步 缺点 如果同步机制出问题，前台展示的数据就错了，目前我是用的1的机制 nosql 有数据就取nosql，nosql没数据就sql取并存入nosql问题，没去测试过当nosql没有的时候，这个时候并发下， sql也会并发 - @理鱼
5. 我是想问。如何自动判断这条数据很热 烫手。自动写redis里 - eli 🎧
6. php控制的话  redis没有 取mysql。看访问量与发布时间   每分钟 或是干脆几秒超过多少点击。就写到redis ？ 之后下次 还是redis开始 没有再mysql？所有内容全写redis。 总觉得奢侈 - eli 🎧  
回: 那你设置过期时间 - 轩脉刃  
回: 由于我这里缓存的数据很多没用redis而是用的ssdb，写硬盘不奢侈 @eli    
我用的 TTl 缓存多久根据业务情况，所有缓存key 可以分组好 管理后台 或者 用户后台可以去干掉这一组触发 重新缓存机制  - @理鱼  
回: ssdb。用ssd组阵列？   不是说 ssd读写越多坏的越快？   便宜是便宜。能有内存踏实嘛 -eli 🎧  
回: 是否是热数据还是得靠你自己判断，根据权重来写存储逻辑。 - cary  
回: 如果内存放的下 当然更好。我缓存了不少 大段文本 所以用内存不合适 - @理鱼  
回: 内容发布者权重？  说是内容权重 对么？ - eli 🎧  
回: 硬件坏这个是没办法的。你的做高可用，我用云 ucloud 阿里 - @理鱼  
回: 这个是需要你根据系统业务来做辨别的 - cary
7. 阿里开源了Canal，一个MySQL同步工具，伪装成MySQL slave，解析binlog  - Daniel
8. 目前是用 crontab 同步(囧) ， 我觉得还是用 消息队列好点 - Feel.
9. redis 缓存一般是主动过期和被动过期。主动过期包括数据变更，就主动删除了。被动就是自己过期了。一般大流量电商或者高访问网站对于通用数据。如果数据有更新都是主动删除再预先生成缓存。 防止数据库被击穿 - 如末
10. 构建健壮且弹性的数据层，Redis既做NoSQL又做Cache，监控缓存命中率，管理好热数据和冷数据策略，灾变和数据预热，等等，Redis也要做好自己的数据持久化～考虑整体数据层垂直和水平立体伸缩性～ - xingxing
11. 可以多利用redis的数据结构，不是简单把redis当做keyvalue - 轩脉刃
12. 大规模网站 redis和mysql都必须做好住主从分离 避免单主库被网卡打满这种悲剧事件 redis设置连接timeout时间短一点 而不是无用连接撑到最大值导致耗尽内存 。客户端一定用完连接要close ，最好有连接池主从分离中间件。 缓存命中率越高越好 ，穿透到db一般都是悲剧。早点知道这些我家tv就不会首日宕机两小时了😂😂😂 mysql 索引必须命中 简单查询不用复杂连表 -  财主刀刀-沈冠璞
13. 最近听说朋友公司把数据全部存redis，mysql都弃用了。。 成lnrp组合了都，可能场景不同，他们好像都是些不太重要的udp数据 - 布罗塔
14. 之前也尝试过， 但是在做一些统计分析时候， 就苦恼了 - 周渊
15. 土豪公司啊，弃用mysql Redis的内存利用率也是蛋疼，复杂的数据结构又很吃内存，断电对redis和mysql的影响是一样的，对数据而言几乎没影响。Redis就是贵，对事务支持有限，一些较复杂的统计分析不如mysql方便，需要自己在redis做索引 - 廖强
16. 之前公司也是大量使用redis,结果数据阻塞了，只能重启，丢失数据 - 布罗塔
17. 同步 保持数据都刷到数据库 架构越复杂，考虑的问题越多，程序也越来越复杂， - 小白
18. 我提前把活跃数据例如商品，近一个月的登录用户以及相关信息，拿到redis中，然后所有的操作都直接操作redis，不查询mysql，然后事后再统一把两个数据进行同步，把redis的数据同步到mysql中 - 刺客  
问: 以redis为主，写操作通过队列再慢慢写到mysql么 - Moses  
嗯 因为数据库数据如果很多的话我不可能把所有的都拿到redis中，只是把mysql当做后端储藏室，一般情况不请求mysql，不知道我这个思路是否科学 - 刺客  
回: 你这样的话 用mc 看是不是更直接些 - Jimmy  
回: mc万一宕机就坏了 写操作并不是实时同步到mysql - 刺客  
还好~ 之前做广告投放就是以redis为主  mysql 只用来存储最后的结果 和 初始化的数据  - Moses  
19. 如果是memcached的话~ 我觉得redis 更适合做数据加减 写入。 毕竟支持的数据类型比较丰富，而且mc集群的话 只能靠php的扩展来做，挂掉一台 和 批量增加集群的话  对缓存的影响比较大，而且每台机器在一致性哈希的集群圆环上占的百分比也不固定，1024个节点上  每台memcache 占的节点数分布不是那么均匀 - Moses  
回: redis对后期扩展比较方便吧 - 刺客  
回: 恩  毕竟新版的都原生支持集群的 - Moses
20. 开一个异步的进程去处理 - 小白
21. php写redis的时候  同时写到队列呗   然后后台跑定时任务 回写到mysql  - Moses

#### 分享链接

1. 谈PHP中信息加密技术 http://mp.weixin.qq.com/s?__biz=MjM5NTg2NTU0Ng==&mid=401148556&idx=3&sn=7e445c42c2051b5648cda62ad157766d - 黑夜路人
2. Docker 工程师必读论文：Google Borg http://mp.weixin.qq.com/s?__biz=MzA4MzQ1NjQ5Nw==&mid=400243103&idx=1&sn=c76e7acfbf73649190277a7cbe3b0191 - 黑夜路人
3. http://goodui.org/  这个网站不错 - smarteng
4. 实时数据库报警与事件研究 http://www.docin.com/p-290897491.html - 周渊
5. 我们现在的系统用到了sso，用的是rails，插件是 https://github.com/rubycas/rubycas-server - 机械唯物主义  
原理就是跑一个sso的服务器，后面连到你主站的用户数据库，用户登录你的A网站，就会导向到sso服务器验证，获得一个ticket，然后回A网站，A网站再往sso网站请求用户信息返回，在A网站上面创建或者更新这个用户的信息。当用户登出的时候，A网站通知sso，sso通知所有登录的网站都登出。
6. 你知道 Hello World 的历史么？ http://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=400496257&idx=1&sn=d544edf8d9f57350f9f9492ec5536efb  - 猿蜗
7. 高并发Web服务的演变——节约系统内存和CPU http://mp.weixin.qq.com/s?__biz=MjM5NTg2NTU0Ng==&mid=401212664&idx=2&sn=24baee94669195275e463129eb66fc6f - 杜世伟
8. PHP7的扩展支持情况 https://github.com/gophp7/gophp7-ext/wiki/extensions-catalog - 姚文强
9. 【问底】徐汉彬：Web系统大规模并发——电商秒杀与抢购 http://www.csdn.net/article/2014-11-28/2822858 - 小白