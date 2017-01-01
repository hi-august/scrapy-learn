# scrapy

####一些常见问题, 经验:
0. 了解scrapy已经做过的功能, 优化等. . . 防止重复造轮子, 如, 去重, 编码检测, useragent, 代理ip, mongodb, mysql存储, 下载件文件名称修改, 布隆过滤器 , dns缓存, http长连接,gzip等等. 
1. JS相关. 
这个是被问的最多的. 看具体情况解决. 可模拟相关js执行、绕过, 或直接调浏览器去访问. 自己用一个JS引擎+模拟一个浏览器环境难度太大了（参见V8的DEMO）. 
调浏览器有很多方法. 难以细说, 关键字如下, selenium, phantomjs, casperjs, ghost, webkit, scrapyjs, splash. 一些细节如关掉CSS渲染, 图片加载等. 只有scrapyjs是完全异步的, 相对是速度最快的, scrapyjs将webkit的事件循环和twisted的事件循环合在一起了. 其他的方案要么阻塞, 要么用多进程. 简单的js需求（对效率要求不高）随意选, 最优方案是scrapyjs+定制webkit（去掉不需要的功能）. 调浏览器开页面是比较耗资源的（主要是cpu）
2. 内容解析. 
XPATH就可以了, 感兴趣可以看下pyquery, css选择器. 
如果想获得网页对应的txt, 可以调浏览器, 有个类似plain_txt的接口可以获取网页保存成txt. 
模糊匹配参考scrapy github里面几个库, 机器学习不一定好用（效果问题, 人工问题-需要训练）. 还有写些正则去模糊匹配. 
新闻类似的正文提取有readability,boilerplate. 
3. 分布式. 
首先考虑按任务（目标）切分, 然后让不同目标的爬虫在不同机器上跑
完全的对等分布式（多爬虫爬一个目标）, 把任务队列替换掉爬虫改改即可. github里面有几个现有的实现参考. 
分布式需求可能是伪命题. 想清楚为何要分布式. 硬件够不够, 像什么拿一个不支持持久化的url队列的爬虫说量大需要分布式的, 我只能默念, 你为何这么吊. 
4. 部署, 调度
部署推荐scrapyd. 这也是官方推荐的方法. 
大量爬虫的调度, 这个目前（13-10）没有现成的合适方法, 期望是实现爬虫的某些配置放数据库, 提供web后台 , 然后按配置周期、定时运行爬虫, 终止, 暂停爬虫等等. 可以实现, 但要自己写不少东西. 
5. ip限制问题
买的起大量ip的可买（买大量同网段爬可能导致整网段被封）. 
找大量免费的开放http代理, 筛选可用的, 免费开放代理不可靠, 写个调度机制, 自动根据成功次数, 延迟等选择合适代理, 这个功能难以在scrapy内实现, 参考scrapinghub的crawlera, 我完成了一个本地版. 
6. url去重. 
如果有千万级的URL需要去重, 需要仔细看下scrapy的去重机制和bloom filter（布隆过滤器）. bloomfilter有个公式可以算需要多少内存. 另bloomfilter  + scrapy在github有现有实现可以参考. 
7. 存储. 
mongodb, mongodb不满足某些功能时考虑hbase,参考http://blog.scrapinghub.com/2013/05/13/mongo-bad-for-scraped-data/
8. 硬件扛不住别玩爬虫. . . 曾在I3 4G 1T上跑爬虫. 卡在磁盘io（量大, 磁盘io差, 内存低）, 出现内存占用飙升. 很难调试（调试爬虫看实际跑耗时较长）, 初步以为是爬虫有问题内存占用高导致数据库卡. 调试结果确认为, 配置低量太大, 导致数据库慢, 数据库慢之后爬虫任务队列占满内存并开始写磁盘, 又循环导致数据库慢. 
9. 爬虫监控
scrapyd自带简单的监控, 不够的话用scrapy的webservice自己写, 暂无（13.10）现成的
10. =. =想到再补充. 
11. scrapy-redis分布式 http://www.xgezhang.com/python_scrapy_redis_crawler.html
12. 爬虫防止被ban可以,设置延迟下载,设置user-agent,禁用cookies,代理ip
13. Scrapy原理, scrpy Engine(引擎)用来控制整个系统的数据处理流程,
Scheduler(调度器)从scrapy engine接受request排序并列入队列
Downloader抓去网页, 并把response返回spider
Spider定制爬虫爬取的规则
Item Pipeline(项目管道)用来处理从网页抽取的项目

