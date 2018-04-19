## 项目简介
    具体功能： 
    
    1.连接东方财富网新股页面(http://quote.eastmoney.com/centerv2/hsgg/xg)，爬取所有新股信息； 
    
    2.剔除股票编码为30开头的创业板块新股；
    
    3.数据清洗并入mysql库
    
    4.添加run.py文件自定义定时执行或立即执行获取信息 

    5.使用celery消息队列工具，自定义程序自动运行和间隔动态运行，实现数据的动态获取 

    6.使用redis保存celery中borker消息的队列缓存,实现异步任务调度和定时开启爬取并快速动态更新数据功能 
    
    7.采用Redis做url动态缓存，实现分布式爬取系统
## 环境要求:
    环境及模块:python3+scrapy框架+Fiddler抓包工具+celery（要求版本4.1.0）+ redis +mysql
## 项目主要方法:
    东方财富网打开新股列表(http://quote.eastmoney.com/centerv2/hsgg/xg) 
    后,可以发现下一页加载的方式为异步加载的方式, 
    不能直接根据url获取所有页面的股票信息,所以需要Fiddler进行抓包, 
    抓包取得每页股票信息列表的json数据,根据请求头的url分析其所有页面json对应url 
    的规律,使用scrapy进行爬取，使用celery实现程序自动执行，使用redis做任务消息动态缓存和url动态分配缓存
## 运行说明： 
    1.run.py文件里启动程序，去掉While True 循环直接运行可以实时运行 
    
    2.在run.py文件While True循环里设置target_time，可实现程序在该指定时间点爬取信息 

    3.在tasks.py里使用celery消息队列工具+redis数据库管理，实现定时/循环/无重复地运行程序 
    
    Celery启动爬虫步骤： 
    
    1) 先启动Master版,至少半分钟后再启动slave版,Master需启动两个定时器(见上述) 
    
    2) 爬取时先启动celery的worker监听任务消息列表: 终端输入celery -A tasks -l INFO worker 
    
    3) 接着启动定时器往队列写入任务: 开启另一个终端celery -A tasks -l INFO beat  即可运行程序，在worker终端查看日志 
    
    注意不能在一个url仍在爬取时突然切断,易引发celery任务写入和mysql存库冲突,若有发生冲突,重启系统并清空一次redis即可重新爬取
## 入库说明：
    将项目获取的数据按字段存入mysql数据库中，数据库版本：Mysql 5.7，使用模块：pymsql 
    
    我在项目数据库里的初始命名 
    
    数据库名为：eastmoney
    
    表名为：gupiao 
    
    各字段名依次为：stock_num/stock_name/stock_price/stock_change_range/stock_change_price 
    
## 项目性能：
   在单机开一台slave连同Master一起爬取可以实现最快每5秒刷新一次数据表，效率主要受写入数据库时的速度影响，也与slave的数量成正比
