####Apache访问控制
```
在2.2以及以前版本中，使用allow，deny来实现访问控制，
在2.4版本中，使用require granted和require denied来实现。

访问控制主要使用在<location> <locationmatch> <directory> <directorymatch> <files> <filesmatch> 中。
```

#####allow和deny用法
```
示例1：
order allow,deny
allow from ip1
allow from ip2
deny from all

示例2：
order deny,allow
deny from all
allow from ip1

示例3：
order allow,deny
allow from all
允许所有请求

示例4：
order allow,deny
deny from all
拒绝所有请求

示例5：
order allow,deny 
拒绝所有

示例6：
order deny,allow
允许所有

示例7：
order deny,allow
deny from all
拒绝所有
```

##### require示例
```
require all granted  允许所有

require all denied 拒绝所有

require ip ip1 ip2  

<requireall>
require not ip ip1 ip2
</requireall>

require ip ip段

require host hostname（域名）
```

#####Directory示例
```
<Directory /1/2/3>
    Order Allow,Deny
    Allow from all
    Deny from 
</Directory>
```

#####Files示例（黑名单）
```
需要模块 authz_host_module

    <Files 1.txt>
        <requireall>
            require all granted
            require not ip 172.7.15.113
        </requireall>
    </Files>
  
  <Files ~ "admin(.*)">
        <requireall>
            require all granted
            require not ip 172.7.15.113
        </requireall>
  </Files>
 
```

#####FilesMatch示例 （黑名单）
```
    <FilesMatch "a.*html$">
        <requireall>
            require all granted
            require not ip ip1 ip2
        </requireall>
    </FilesMatch>

```

#####Location示例 (白名单）
```
    <Location /abc/>
            require all denied
            require ip 172.7.15.113
    </Location>


    <Location ~ abc>
            require all denied
            require ip 172.7.15.113
    </Location>

```

#####LocationMatch示例（白名单）
```
    <LocationMatch  "abc">
            require all denied
            require ip 172.7.15.113
    </LocationMatch>
```

#####用户认证
```
模块： authn_file_module  authn_core_module authz_user_module auth_basic_module authz_core_module

/usr/local/apache2.4/bin/htpasswd -cm /data/.htpasswd user1

    <Directory /data/wwwroot/www.123.com> //指定认证的目录
        AllowOverride AuthConfig //这个相当于打开认证的开关
        AuthName "123.com user auth" //自定义认证的名字，作用不大
        AuthType Basic //认证的类型，一般为Basic，其他类型阿铭没用过
        AuthUserFile /data/.htpasswd  //指定密码文件所在位置
        require valid-user //指定需要认证的用户为全部可用用户
    </Directory>

或者

    <FilesMatch admin.php>
        AllowOverride AuthConfig
        AuthName "123.com user auth"
        AuthType Basic
        AuthUserFile /data/.htpasswd
        require valid-user
    </FilesMatch>

```

#####限速
```
模块： env_module  ratelimit_module

<Location "/downloads/*.mp4">
    SetOutputFilter RATE_LIMIT
    SetEnv rate-limit 400 
    SetEnv rate-initial-burst 512
</Location>
 
单位是kb/s，针对每一个请求限速
```
