####apache优化
```
apache运行在操作系统上，所以操作系统本身的优化也是非常关键的，除了操作系统的优化外，apache也可以通过以下几个方面进行优化。
```

#####安装
```
1）源码安装

2）除核心模块外，其他模块动态化安装

3）不使用的模块不加载
Loaded Modules:
 core_module (static)
 so_module (static)
 http_module (static)
 mpm_event_module (shared)
 authz_core_module (shared)
 mime_module (shared)
 log_config_module (shared)
 unixd_module (shared)
```

#####MPM调整
```
首先要确定apache的运行方式是哪个MPM
apachectl -M 
2.4版本建议使用event

# StartServers: initial number of server processes to start （启动服务时，要启几个进程）
# MinSpareThreads: minimum number of worker threads which are kept spare （最小空闲线程数，如果空闲的线程小于设定值，apache会自动建立线程）
# MaxSpareThreads: maximum number of worker threads which are kept spare（最大空闲线程数，如果空闲的线程大于设定值，apache会自动kill掉多余的线程）
# ThreadsPerChild: constant number of worker threads in each server process（每个进程包含的线程数）
# MaxRequestWorkers: maximum number of worker threads （最多有多少个工作线程）
# MaxConnectionsPerChild: maximum number of connections a server process serves before terminating（每个进程最多处理多少个连接）

配置示例：
<IfModule mpm_event_module>
    ServerLimit            200  #最多开启多少进程，该数值 * ThreadsPerchild的积应该大于等于MaxConnectionsPerChild
    StartServers           5
    MinSpareThreads        250
    MaxSpareThreads        500
    ThreadsPerChild        50
    MaxRequestWorkers      5000
    MaxConnectionsPerChild   10000
</IfModule>

配置原则：
最核心的配置：MaxRequestWorkers
根据服务器的cpu、内存数量来决定开多少个MaxRequestWorkers。
虽然线程越多，可以处理的并发越大，但线程太多，CPU核数不够支撑，反而会降低性能。
需要根据自己的业务特性来适当调整MaxRequestWorkers数量，这个指标并不是一撮而就的，需要反复调整，最终达到一个合理数值。

```

#####启用压缩
```
模块：  deflate_module  filter_module
LoadModule deflate_module modules/mod_deflate.so
LoadModule filter_module modules/mod_filter.so (AddOutputFilterByType)

<Ifmodule mod_deflate.c>
    DeflateCompressionLevel 9      #压缩等级，越大压缩越狠，消耗CPU也越高
    SetOutputFilter DEFLATE           #启用压缩
    AddOutputFilterByType DEFLATE text/html text/plain text/xml     #仅压缩限制特定的MIME类型文件：
    AddOutputFilterByType DEFLATE application/javascript
    AddOutputFilterByType DEFLATE text/css
    AddOutputFilterByType DEFLATE image/gif image/png  image/jpe image/swf image/jpeg image/bmp
    #DeflateFilterNote ratio     #在日志中放置压缩率标记，下面是记录日志的，这个功能一般不用
    #LogFormat '"%r" %{outstream}n/%{instream}n (%{ratio}n%%)' deflate
    #CustmLog logs/deflate_log.log deflate
</Ifmodule>
```

#####配置静态文件过期时间
```
模块： expires_module
LoadModule expires_module modules/mod_expires.so
<IfModule mod_expires.c>
    ExpiresActive on
    ExpiresDefault "access plus 12 month"
    ExpiresByType text/html "access plus 12 months"
    ExpiresByType text/css "access plus 12 months"
    ExpiresByType image/gif "access plus 12 months"
    ExpiresByType image/jpeg "access plus12  12 months"
    ExpiresByType image/jpg "access plus 12 months"
    ExpiresByType image/png "access plus 12 months"
    EXpiresByType application/x-shockwave-flash "access plus 12 months"
    EXpiresByType application/x-javascript "access plus 12 months"
    ExpiresByType video/x-flv "access plus 12 months"
</IfModule>

```

#####开启防盗链
```
1） 通过rewrite实现：
需要rewrite_module
LoadModule rewrite_module modules/mod_rewrite.so

<IfModule mod_rewrite.c>
    RewriteCond %{HTTP_REFERER}  !^$
    RewriteCond %{HTTP_REFERER}  !^http(s)?://(www\.)?xxx.com [NC]
    RewriteRule \.(jpg|jpeg|png|gif)$ - [NC,F,L]
</IfModule>

2）通过setenvif实现
需要access_compat_module和setenvif_module
LoadModule access_compat_module modules/mod_access_compat.so （Order）
LoadModule setenvif_module modules/mod_setenvif.so （setenvif)

        SetEnvIfNoCase Referer "http://www.123.com" local_ref
        SetEnvIfNoCase Referer "http://123.com" local_ref
        SetEnvIfNoCase Referer "^$" local_ref
        <filesmatch "\.(txt|doc|mp3|zip|rar|jpg|gif)">
            Order Allow,Deny
            Allow from env=local_ref
        </filesmatch>

```


#####长连接
```
如果用户请求有大量的js、css、图片等元素，可以设置如下参数：
KeepAlive On
KeepAliveTimeout  30
```



