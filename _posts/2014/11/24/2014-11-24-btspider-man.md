---
layout : life
title : BTspider维护手册
category : 程序
date : 2014-11-24 15:52
---
# 提示
1. 我们帮您部署的时候, 按照您的带宽, 服务器配置, 以及机房位置, 设置了最合适的配置. 以下配置说明主要目的是让您更加了解BTspider. 以便以后根据自身情况, 作些适当的调整. 如果您实在搞不懂, 您可随时在调整爬虫的时候, 向我们咨询, 不收费.
2. BTspider的爬虫部件在`/root/btspider`目录里.
3. BTspider的网站部件在`/var/www/btspider-web`目录里.


# 爬虫部件
## btspider可用的命令帮助

```sh
./btspider

# 将会输出以下如何使用的信息
USEAGE:
    btspider dht start|stop|restart 0.0.0.0 12345 # 启动|停止|重启 DHT爬虫
    btspider bt start|stop|restart name           # 启动|停止|重启 BT爬虫
    btspider total show                           # 显示一共有多少资源
    btspider code show                            # 显示机器码
```
## 运行BTspider的DHT爬虫

```sh
./btspider dht start 0.0.0.0 1234

# 可运行多个进程, 端口要不一样.
./btspider dht start 0.0.0.0 1235
./btspider dht start 0.0.0.0 1236
```

## 停止BTspider的DHT爬虫

```sh
./btspider dht stop 0.0.0.0 1234
./btspider dht stop 0.0.0.0 1235
./btspider dht stop 0.0.0.0 1236
```

## 重启BTspider的DHT爬虫

```sh
./btspider dht restart 0.0.0.0 1234
./btspider dht restart 0.0.0.0 1235
./btspider dht restart 0.0.0.0 1236
```

## 运行BTspider的BT爬虫
可以运行多个, 运行得越多, 耗费资源更高, 爬取速度也会更快, 爬取速度受限于带宽, 服务器配置.

```sh
./btspider bt start 1
./btspider bt start 2
./btspider bt start 3
./btspider bt start 4
./btspider bt start 5
```

这里需要注意的是, 如果您运行了5个BT爬虫, 那么需要在crontab里写上这5条的restart计划任务,
这主要目的是让BT爬虫"重生", 使其以最高效的速度运行, 不然后面会越来越慢.

以root身份输入以下命令

```sh
crontab -e
```
把如下内容复制进去, 然后保存

```sh
*/10 * * * * /root/btspider/btspider bt restart 1
*/10 * * * * /root/btspider/btspider bt restart 2
*/10 * * * * /root/btspider/btspider bt restart 3
*/10 * * * * /root/btspider/btspider bt restart 4
*/10 * * * * /root/btspider/btspider bt restart 5
```


如果您后面执行了如下命令来终止一个BT爬虫进程, 那么需要从crontab里删除代号1的restart.
不然10分钟后, 会再次启动.

```sh
./btspider bt stop 1
```
## 停止BTspider的BT爬虫
```sh
./btspider bt stop 1
./btspider bt stop 2
./btspider bt stop 3
./btspider bt stop 4
./btspider bt stop 5
```

## 重启BTspider的BT爬虫
```sh
./btspider bt restart 1
./btspider bt restart 2
./btspider bt restart 3
./btspider bt restart 4
./btspider bt restart 5
```

## 爬虫部件配置文件(/etc/btspider/btspider.conf)

```sh
# /etc/btspider/btspider.conf

[identity]
# 相当于身份证号码, 一套爬虫绑定一台服务器.
# 该值在我们给您安装部署的时候, 给您绑定的.
id = 25af9a37ba20f69c8df6


# MySQL 连接配置
[db]
host = 127.0.0.1        # 连接地址
port = 3306             # 连接端口
user = root             # 连接账号
passwd = root           # 连接密码
dbname = btspider       # 库名


# Redis 连接配置
[redis]
host = 127.0.0.1        # 连接地址
port = 6379             # 连接端口
db = 0                  # 库名, 只能是0-15


# BTspider的DHT爬虫, 可以独立运行在一台服务器
[dht]
# 在Redis中存放资源集合队列的KEY.
# 您可以在redis-cli里输入SCARD infohashes来知道有多少条资源
# 队列等待处理, 如果总是大于0, 那么您可以再启动一个BT爬虫进
# 程来加速资源处理.
redis-key = infohashes

# DHT爬虫自身的队列, 如果您带宽不高, 可以降低点,
# 值越大, 耗费的带宽越高. 当然, 耗费带宽越高, 爬取的资
# 源速度也更快, 不过需要配合BT爬虫, 只要DHT爬虫和BT配合好了,
# 两个低配的VPS都能比得上那些不会搭配的高配独立服务器.
max-node-qsize = 100


# 资源在Redis中的最大等待被BT爬虫处理的队列数,
# 如果BT爬虫跑慢了, 该值得改小一点, 不然等待时间长了,
# 资源将会从全世界各个BT客户端下载不到.
redis-infohash-max-qsize = 5000


# BTspider的BT爬虫, 可以独立运行在一台服务器
[bt]
# 爬取资源超时时间, 单位秒, 超时越长, 将会有更大几率成功爬取,
# 代价就是降低BT爬虫速度. 也会影响队列里资源生命时效.
timeout = 3

# 每个BT爬虫进程的线程数, 这里要特别注意, 如果您运行了5个BT爬虫进程, threads是50的话,
# 那么一共50 * 5 + 2 = 252线程, 而每个线程独占一个MySQL连接数, 因此要保证您MySQL最大
# 连接数足以支撑, 您可修改my.cnf的max_connections值, 还需要注意, 网站也会使用MySQL连接数,
# 所以一定要为网站部件+爬虫部件一起考虑. 否则网站或者会出现连接数不够用, 将会出现某些"罢工"现象.
threads = 50

# 爬虫程序运行出现的异常日志记录位置
[log]
app = /tmp/btspider-app.log
```

# 网站配置

```php
# /var/www/btspider-web/conf/AppConf.php
<?php
namespace btspider\conf;

class AppConf {

    // MySQL 连接配置
    public static $db = array(
        'dsn' => 'mysql:dbname=btspider; host=127.0.0.1; port=3306; charset=utf8',
        'user' => 'root',
        'passwd' => 'root',
    );

    // Redis 连接配置
    public static $redis = array(
        'host' => '127.0.0.1',
        'port' => 6379,
        'db' => 0,
    );

    // coresek 连接配置
    public static $coreseek = array(
        'host' => '127.0.0.1',
        'port' => 9312,
    );

    // 每页显示多少条资源
    public static $page_per_limit = 20;

    // 静态缓存文件存放目录, 执行脚本的用户要有对该目录写入权限, 所在分区要有足够的容量
    public static $cache_dir = 'cache';

    // 服务器静态缓存文件生存时间, 默认1个小时, 单位秒, 如果没过期,
    // 浏览器也会缓存, 不再在服务器下载页面内容
    public static $max_age = 3600;

    // 以下所有配置禁止修改, 除非你知道你在做什么.
    public static $orderby = array(
        'size' => 'r_length',
        'first' => 'r_first',
        'last' => 'r_last',
        'hot' => 'r_hot',
    );
    public static $sortby = array('asc', 'desc');
    public static $index = array(
        'main_0', 'main_1', 'main_2', 'main_3', 'main_4', 'main_5', 'main_6', 'main_7',
        'main_8', 'main_9', 'main_a', 'main_b', 'main_c', 'main_d', 'main_e', 'main_f',
    );
    public static $prefix = array('0', '1', '2', '3', '4', '5', '6',
        '7', '8', '9', 'a', 'b', 'c', 'd', 'e', 'f',
    );
}
```