####Options指令
```
Options指令可以在Apache服务器核心配置(server config)、虚拟主机配置(virtual host)、特定目录配置(directory)以及.htaccess文件中使用。
Options指令的主要作用是控制特定目录将启用哪些服务器特性。

Options指令常见的配置示例代码如下：
<Directory />
#指定根目录"/"启用Indexes、FollowSymLinks两种特性。
    Options Indexes FollowSymLinks
    AllowOverride all
    Order allow,deny
    Allow from all
</Directory>
```

#####语法
```
Options指令的完整语法为：Options [+|-]option [[+|-]option] ...。
简而言之，Options指令后可以附加指定多种服务器特性，特性选项之间以空格分隔。

All
表示除MultiViews之外的所有特性。这也是Options指令的默认设置。

None
表示不启用任何的服务器特性。

FollowSymLinks
服务器允许在此目录中使用符号连接。如果该配置选项位于<Location>配置段中，将会被忽略。

Indexes
如果输入的网址对应服务器上的一个文件目录，而此目录中又没有DirectoryIndex指令
(例如：DirectoryIndex index.html index.php)，那么服务器会返回由mod_autoindex模块生成的一个格式化后的目录列表。

MultiViews
允许使用mod_negotiation模块提供内容协商的"多重视图"。简而言之，如果客户端请求的路径可能对应多种类型的文件，
那么服务器将根据客户端请求的具体情况自动选择一个最匹配客户端要求的文件。
例如，在服务器站点的file文件夹下中存在名为hello.jpg和hello.html的两个文件，此时用户输入Http://localhost/file/hello，
如果在file文件夹下并没有hello子目录，那么服务器将会尝试在file文件夹下查找形如hello.*的文件，
然后根据用户请求的具体情况返回最匹配要求的hello.jpg或者hello.html。

SymLinksIfOwnerMatch
服务器仅在符号连接与目标文件或目录的所有者具有相同的用户ID时才使用它。
简而言之，只有当符号连接和符号连接指向的目标文件或目录的所有者是同一用户时，才会使用符号连接。
如果该配置选项位于<Location>配置段中，将会被忽略。

ExecCGI
允许使用mod_cgi模块执行CGI脚本。
  
Includes
允许使用mod_include模块提供的服务器端包含功能。
  
IncludesNOEXEC
允许服务器端包含，但禁用"#exec cmd"和"#exec cgi"。但仍可以从ScriptAlias目录使用"#include virtual"虚拟CGI脚本。
此外，Options指令语法允许在配置选项前加上符号"+"或者"-"。

实际上，Apache允许在一个目录配置中设置多个Options指令。不过，一般来说，如果一个目录被多次设置了Options，
则指定特性数量最多的一个Options指令会被完全接受(其它的被忽略)，而各个Options指令之间并不会合并。
但是如果我们在可选配置项前加上了符号"+"或"-"，那么表示该可选项将会被合并。
所有前面加有"+"号的可选项将强制覆盖当前的可选项设置，而所有前面有"-"号的可选项将强制从当前可选项设置中去除。
```

#####示例
```
#示例1
<Directory /web/file>
Options Indexes FollowSymLinks
</Directory>

<Directory /web/file/image>
Options Includes
</Directory>
#目录/web/file/image只会被设置Includes特性

#示例2
<Directory /web/file>
Options Indexes FollowSymLinks
</Directory>

<Directory /web/file/image>
Options +Includes -Indexes
</Directory>
#目录/web/file/image将会被设置Includes、FollowSymLinks两种特性

```