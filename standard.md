####apache运维规范
```
注意：本规范不包含具体的细节配置，只列出规范点以及思路。
```

#####安装、升级
```
1 安装

1）源码安装
2）只能到官方下载源码包
3）下载apr-1.6、apr-util-1.6以及httpd-2.4，分别解压三个源码包
把apr-1.6.3 放到httpd源码包的/srclib/下，改名apr
把apr-util-1.6.1 放到httpd源码包的/srclib/下，改名apr-util
4）编译参数./configure --prefix=/dir/  --enable-so --enable-mpms-shared=all  --with-mpm=event  --enable-mods-shared=most --with-included-apr
   说明：这里的/dir/为apache安装路径，根据需求定目录

2 升级
当有漏洞发生需要通过官方提供补丁以及升级文档进行升级。
若是常规版本升级，需要通过以下步骤进行升级
1）先安装新版本
2）根据老版本的配置文件配置新版本
3）监听不同端口，测试可用性
4）切换（停止老版本，开启新版本）
```

#####服务管理
```
1）以非root用户启动apache
2）若监听端口为1024以内的端口，需要借助sudo启动
3）启动 apachectl  start
4）关闭 apachectl stop
5）重启 apachectl restart
6）重载 apachectl graceful
```

#####用户、权限管理
```
1）配置文件中指定固定的User和Group（如，统一配置为nobody）
2）启动服务的用户和配置文件中的用户保持一致
3）整个apache目录（如，/usr/local/apache2)属主和属组设置要和配置文件中指定的User和Group一致
4）整个apache目录，所有目录权限为755，所有文件权限为644
```

#####虚拟主机配置
```
1）打开虚拟主机配置
Include  conf/vhos/*.conf

2）所有虚拟主机配置文件全部放到conf/vhost/目录下面，并且以域名命名配置文件
如，conf/vhost/www.test.com.conf

3）禁止默认虚拟主机访问
查看默认虚拟主机 apachectl  -S
```

#####日志配置
```
1）所有日志（访问日志、错误日志）全部放到/data/logs/目录下
2）为每一个虚拟主机配置访问日志和错误日志
3）配置日志切割、旧日志删除或归档策略
4）配置静态文件不记录日志
```

#####mpm配置
```
1）2.0启用prefork模式
2）2.2启用worker模式
3）2.4启用event模式
4）prefork配置示例(服务器配置4c8g，假设单个httpd进程占用内存不超过20M)
<IfModule mpm_prefork_module>
    ServerLimit 300  
    StartServers 50  
    MinSpareServers 50  
    MaxSpareServers 100  
	MaxClients 300  
    MaxRequestsPerChild 10000
</IfModule>

5）worker配置示例（服务器配置4c8g，假设单个httpd进程占用内存不超过20M）
<IfModule mpm_worker_module>
    ServerLimit              300
    StartServers             5
    MinSpareThreads         125
    MaxSpareThreads         250
    ThreadsPerChild         25
    MaxRequestWorkers      7500
    MaxConnectionsPerChild   5000
</IfModule>

6）event配置示例（服务器配置4c8g，假设单个httpd进程占用内存不超过20M）
<IfModule mpm_event_module>
    ServerLimit		   300
    StartServers            5
    MinSpareThreads        125
    MaxSpareThreads        250
    ThreadsPerChild         25
    MaxRequestWorkers      7500
    MaxConnectionsPerChild  5000
</IfModule>

```
#####安全相关
```
参考 https://coding.net/u/aminglinux/p/apache/git/blob/master/security.md
```