####apache代理
```
apache可以实现正向代理和反向代理。
实现代理的模块： proxy_module   proxy_http_module  proxy_balancer_module

httpd.conf中增加：
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so
```

#####正向代理配置
```
        ProxyRequests On   //打开正向代理
        ProxyVia On
        <Proxy *>
           Order Deny,Allow
           Deny from all
           Allow from 127.0.0.1
           Allow from 192.168.0.0/24
        </Proxy>

```

#####反向代理
```
        ProxyRequests Off  #关闭正向代理
        ProxyPreserveHost On
        
        ProxyPass / http://127.0.0.1:8080/   
        ProxyPassReverse / http://127.0.0.1:8080/  

        <Proxy *>
            Order Deny,Allow
            Allow from all
        </Proxy>

说明： 
1）ProxyPassReverse的作用
例：请求 www.123.com/abc/1.html --> 代理  www.123.com/abc/ --> ProxyPass www.456.com/def/
在RS上有设置  /def/1.html 跳转到  /def/2.html，即www.456.com/def/1.html --> www.456.com/def/2.html
如果没有ProxyPassReverse，则最终访问的链接是www.123.com/def/2.html
设置了ProxyPassReverse，最终访问的链接是www.123.com/abc/2.html

2）ProxyVia

语法：ProxyVia [on|off|full|block]，默认为off

如果设置为Off，将不会采取特殊的处理。如果一个请求或应答包含 "Via:" 头，将不进行任何修改而直接通过。
如果设置为On，每个请求和应答都会对应当前主机得到一个"Via:" 头。
如果设置为Full，每个产生的 "Via:" 头中都会额外加入Apache服务器的版本，以"Via:"注释域出现。
如果设置为Block，每个代理请求中的所有"Via:"头行都将被删除。且不会产生新的 "Via:" 头。

3）ProxyPreserveHost
如果设置为on，则会把header中的Host字段传递给被代理服务器。
```

#####负载均衡
```
模块： proxy_balancer_module
LoadModule proxy_balancer_module modules/mod_proxy_balancer.so

    <Proxy balancer://mycluster>
        BalancerMember http://192.168.0.121:80/ loadfactor=1
        BalancerMember http://192.168.0.122:80/ loadfactor=2
        BalancerMember http://192.168.0.123:80/
    </Proxy>
     
    ProxyRequests Off
    ProxyPreserveHost On
    ProxyPass / balancer://mycluster/
    ProxyPassReverse / balancer://mycluster/
 
负载均衡的策略(需开启对应模块)：
lbmethod=byrequests 按照请求次数均衡(默认)  lbmethod_byrequests_module
lbmethod=bytraffic 按照流量均衡   lbmethod_bytraffic_module
lbmethod=bybusyness 按照繁忙程度均衡(总是分配给活跃请求数最少的服务器)   lbmethod_bybusyness_module

配置：
<Proxy balancer://mycluster lbmethod=bytraffic>
```

#####会话保持
```
需要模块： headers_module

Header add Set-Cookie "ROUTEID=.%{BALANCER_WORKER_ROUTE}e; path=/" env=BALANCER_ROUTE_CHANGED
<Proxy "balancer://mycluster">
    BalancerMember "http://192.168.1.50:80" route=1
    BalancerMember "http://192.168.1.51:80" route=2
    ProxySet stickysession=ROUTEID
</Proxy>
ProxyPass        "/test" "balancer://mycluster"
ProxyPassReverse "/test" "balancer://mycluster"

```

#####健康检查
```
模块： proxy_hcheck_module
该功能依赖 mod_watchdog

ProxyPass指令自带了ping属性，可用于简单判断后端节点是否健康，只要Ping能通信就认为是健康的。
但显然，对于Http服务来说，健康的指标并不能简单地通过它来判断。例如，检测某个页面是否正常、是否允许某方法等。
因此，httpd提供了一个专门的健康状况检查模块mod_proxy_hcheck用于个性化订制检查指标。

检查指标也即检查方法有以下几种，由hcmethod指定：

TCP：检查是否能与后端节点建立TCP套接字。
OPTIONS：发送一个HTTP OPTIONS请求给后端节点。
HEAD：发送一个HTTP HEAD请求给后端节点。
GET：发送一个HTTP GET请求给后端节点。
该健康状况检查模块认为，只要HTTP方法的检查指标返回2xx或3xx状态码都认为是健康的。

指定了检查方法后，还需订制检查的细节，例如检查的时间间隔。包括以下几项：

hcinterval：默认为30秒。发送检查的时间间隔，单位为秒。
hcuri：健康检查时，追加在URL后的URI。通常用于GET检查方法。
hcpasses：默认为1。表示只有检查了N次后都是通过的，才认为该节点是健康的可再次启用。
hcfails：默认为1。表示只有检查了N次后都是失败的，才认为该节点已经不健康，于是禁止使用该节点。
例如，以下是几个健康检查的配置示例：

<Proxy balancer://foo>
  BalancerMember http://www.example.com/  hcmethod=GET hcuri=/status.php
  BalancerMember http://www1.example.com/ hcmethod=TCP hcinterval=5 hcpasses=2 hcfails=3
  BalancerMember http://www2.example.com/
</Proxy>

ProxyPass "/" "balancer://foo"
ProxyPassReverse "/" "balancer://foo"
```