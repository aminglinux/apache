####apache安全
```
我们主要从以下几个方面来达到安全的目的。
```
#####配置运行账号
```
User nobody
Group nobody
```

#####禁止列出目录
```
如果配置中有  Options Indexes FollowSymLinks 
需要去掉Indexes这项
如：
    <Directory /data/wwwroot/www.123.com>
        AllowOverride none
        Options -Indexes
        require all granted
    </Directory>

```

#####自定义错误页面
```
 ErrorDocument 403 /403.html
 ErrorDocument 404 /404.html
 ErrorDocument 500 /500.html
 ErrorDocument 502 /502.html
```
#####隐藏版本号
```
ServerTokens Prod
ServerSignature Off 
```

#####禁止默认虚拟主机
```
把第一个虚拟主机直接禁止访问
<Location />
    Order allow,deny
    deny from all
</Location>
```

#####关闭不使用的模块
```
apachectl -M 先查看加载的模块，然后关闭不用的模块
```

#####可写目录控制权限
```
对于一些需要写的目录，例如临时文件、缓存、日志等目录，可以禁止web访问。
<Directory /xxx/xxx/temp/>
    Order allow,deny
    deny from all
</Directory>

如果是php网站，对于可写的目录，例如上传图片的目录，还要禁止php解析
<Directory /xxx/xxx/upload/>
    php_admin_flag engine off
</Directory>
```

#####限制某些类型文件的访问
```
<Files ~ "\.htpasswd|\.htaccess|\.bak>
    Order allow,deny
    deny from all
</Files>
```

#####关闭.htaccess的支持
```
AllowOverride All 
改为
AllowOverride None
```

#####关闭自带CGI功能
```
<Directory "/usr/local/apache2/cgi-bin">
    AllowOverride None
    Options None
    Require all granted
</Directory>

改为 

<Directory "/usr/local/apache2/cgi-bin">
    AllowOverride None
    Options None
    Require all denied
</Directory>

```