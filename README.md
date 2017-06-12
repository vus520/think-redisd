ThinkPHP 5.0 Redisd驱动，简单的读写分离实现
===============

#### step 1, 安装 php-redis 驱动

https://github.com/phpredis/phpredis


Centos上的安装命令
```
git clone https://github.com/phpredis/phpredis
cd phpredis
phpize
./configure [--enable-redis-igbinary]
make && make install

echo extension=redis.so>/etc/php.d/redis.ini

php -m | grep redis
```

#### step 2, 修改thinkphp配置文件，加载 redisd 驱动

```
vim application/config.php
```

```
配置参数:
'cache' => [
    'type'       => '\think\Redisd'
    'host'       => 'A:6379,B:6379', //redis服务器ip，多台用逗号隔开；读写分离开启时，默认写A，当A主挂时，再尝试写B
    'slave'      => 'B:6379,C:6379', //redis服务器ip，多台用逗号隔开；读写分离开启时，所有IP随机读，其中一台挂时，尝试读其它节点，可以配置权重
    'port'       => 6379,    //默认的端口号
    'password'   => '',      //AUTH认证密码，当redis服务直接暴露在外网时推荐
    'timeout'    => 10,      //连接超时时间
    'expire'     => false,   //默认过期时间，默认为永不过期
    'prefix'     => '',      //缓存前缀，不宜过长
    'persistent' => false,   //是否长连接 false=短连接，推荐长连接
],
```

#### step 3, 使用redisd驱动

```
$redis = \think\Cache::connect(Config::get('cache'));
$redis->master(true)->setnx('key');
$redis->master(false)->get('key');
```