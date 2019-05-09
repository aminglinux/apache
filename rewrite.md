####apache rewrite
```
Rewirte主要的功能就是实现URL的跳转，它的正则表达式是基于Perl语言。
可基于服务器级的(httpd.conf)和目录级的(.htaccess)两种方式。如果要想用到rewrite模块，必须先安装或加载rewrite模块。
检查模块是否加载
apachectl -M |grep -i rewrite

基于服务器级的(httpd.conf)有两种方法，一种是在httpd.conf的全局下直接利用RewriteEngine on来打开rewrite功能;
另一种是在局部里利用RewriteEngine on来打开rewrite功能,需要注意的是,必须在每个virtualhost里用RewriteEngine on来打开rewrite功能。
否则virtualhost里没有RewriteEngine on它里面的规则也不会生效。

基于目录级的(.htaccess),要注意一点那就是必须打开此目录的FollowSymLinks属性（httpd.conf中的Option FollowSymLinks）
且在.htaccess里要声明RewriteEngine on。
```

#####rewrite规则标记
|标记|说明|
| :--------   | :-----   |
|R[=code](force redirect)| 强制外部重定向，如果code不指定，将用缺省的302 HTTP状态码。|
|F(force URL to be forbidden)|禁用URL,返回403HTTP状态码。|
|G(force URL to be gone)| 强制URL为GONE，返回410HTTP状态码。|
|P(force proxy) |强制使用代理转发。|
|L(last rule) |表明当前规则是最后一条规则，停止分析以后规则的重写。|
|N(next round)| 重新从第一条规则开始运行重写过程。|
|C(chained with next rule)| 与下一条规则关联,如果规则匹配则正常处理，该标志无效，如果不匹配，那么下面所有关联的规则都跳过。|
|T=MIME-type(force MIME type)| 强制MIME类型|
|NS (used only if no internal sub-request) |只用于不是内部子请求|
|NC(no case)| 不区分大小写|
|QSA(query string append) |追加请求字符串|
|NE(no URI escaping of output)| 不在输出转义特殊字符,例如：RewriteRule /foo/(.*) /bar?arg=P1%3d$1 [R,NE] 将能正确的将/foo/zoo转换成/bar?arg=P1=zoo|
| PT(pass through to next handler)| 传递给下一个处理, 例如：RewriteRule ^/abc(.*) /def$1 [PT] # 将会交给/def规则处理|
| S=num(skip next rule(s)) |跳过num条规则|
| E=VAR:VAL(set environment variable)| 设置环境变量|

#####rewrite cond相关用法
|用法|说明|
| :--------   | :-----   |
|-d |判断对象是否是目录并且存在，如RewriteCond %{REQUEST_FILENAME} !-d,加!就是取反|
|-f |判断对象是否是文件并且存在|
|-s |比上面的-f多了一层含义，即文件是否是大于0的 |
|[NC]|在RwriteCond中也可以使用NC，表示不区分大小写|
|[OR]|用于多个条件中间，表示或者|
|-|让apache不再重写，通常和F配合使用，如Rewrite .* - [F]|

#####rewrite举例
```
示例1（域名跳转）
<IfModule mod_rewrite.c> //需要mod_rewrite模块支持
    RewriteEngine on  //打开rewrite功能
    RewriteCond %{HTTP_HOST} !^www.123.com$  //定义rewrite的条件，主机名（域名）不是www.123.com满足条件
    RewriteRule ^/(.*)$ http://www.123.com/$1 [R=301,L] //定义rewrite规则，当满足上面的条件时，这条规则才会执行
</IfModule>

示例2（针对目录跳转域名）
<IfModule mod_rewrite.c> 
    RewriteEngine on  
    RewriteCond %{REQUEST_URI} ^/bbs/  //定义rewrite的条件，访问的目录是/bbs/
    RewriteRule ^/bbs/(.*)$ http://bbs.xxx.com/$1 [R=301,L] //定义rewrite规则，当满足上面的条件时，跳转到域名bbs.xxx.com
</IfModule>
  
示例3（防盗链）

RewriteCond %{HTTP_REFERER}  !^$
RewriteCond %{HTTP_REFERER}  !^http(s)?://(www\.)?xxx.com [NC]
RewriteRule \.(jpg|jpeg|png|gif)$ - [NC,F,L]

示例4（rewrite日志）

如下是2.2以及以下版本配置方法
rewritelog logs/rewrite.log 
rewriteloglevel 9  //9为最大即得到最多的调试信息,最小为1，最小的调试信息，默认为0，没有调试信息

2.4版本这样配置：
    LogLevel rewrite:trace3
    ErrorLog "/tmp/error.log"


示例5（文件不存在）

RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule .* /404.html [R=302,L]

示例6 （限制user_agent)

RewriteCond %{HTTP_USER_AGENT}  .*curl.* [NC,OR]
RewriteCond %{HTTP_USER_AGENT}  .*baidu.com.* [NC]
RewriteRule  .*  -  [F]

```