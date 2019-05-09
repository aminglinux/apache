####access_log
```
CustomLog 指令用来对服务器的请求进行日志记录。格式为： 
格式1：CustomLog 访问日志文件名 记录格式说明串|格式昵称 
格式2：CustomLog "|管道程序名 访问日志文件名" 记录格式说明串|格式昵称 
其中： 
   1.  访问日志文件名：除非文件位置用”/“开头，否则所制定的文件位置是相对于 ServerRoot 目录的相对路径 
   2.  格式昵称：使用 LogFormat 指令将一个记录格式说明串赋以一个名称 
   3.  记录格式说明串：用字符串和格式说明符（以%开头）指定日志记录的内容 
   4.  管道程序名：管道符”|”后面紧跟着一个程序的路径，这个程序把日志从标准输入设备中读入并处理。 
```

#####日志格式
```
LogFormat 指令用于定义访问日志的记录格式。格式为： 
LogFormat "记录格式说明串" 格式昵称 

例如：
LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined 
LogFormat "%h %l %u %t \"%r\" %>s %b" common 
```

#####日志字段说明
| 字段       | 说明    |
| :--------   | :-----   | 
|%v |虚拟主机中的ServerName定义的字段|
|%V |请求的host字段，如果做了泛解析，%v是没有办法记录请求的域名的|
|%h|客户端的IP地址|
|%l |从identd服务器中获取远程登录名称，基本已废弃|
|%u|如果做了认证，该字段记录认证的用户|
|%t|记录日期和时间|
|%r |HTTP请求头的第一行信息，包括：方法、资源、协议|
|%>s|响应请求的状态代码|
|%b|传送的字节数（不包含HTTP头信息）|
|%{Referer}i |记录引用此资源的网页，即从哪里跳过来的|
|%U |请求的URL路径|
|%{User-Agent}i|浏览器信息|

#####日志切割
```
CustomLog "|/usr/local/apache2.4/bin/rotatelogs -l logs/123.com-access_%Y%m%d.log 86400" 

-l：使用本地时间代替GMT时间作为时间基准
86400 指的是日志文件滚动的以秒为单位的间隔时间
```

#####不记录指定类型文件
```
    SetEnvIf Request_URI ".*\.gif$" img
    SetEnvIf Request_URI ".*\.jpg$" img
    SetEnvIf Request_URI ".*\.png$" img
    SetEnvIf Request_URI ".*\.bmp$" img
    SetEnvIf Request_URI ".*\.swf$" img
    SetEnvIf Request_URI ".*\.js$" img
    SetEnvIf Request_URI ".*\.css$" img 
    CustomLog "logs/123.com-access_log" combined env=!img
```

#####记录指定uri
```
SetEnvIf Request_URI "^/aaa/.*" aaa-request
CustomLog "|/usr/local/apache2/bin/rotatelogs -l logs/aaa-access_%Y%m%d.log 86400" combined env=aaa-request
```