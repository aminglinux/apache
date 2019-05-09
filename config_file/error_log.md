####error_log
```
查看默认error_log所在路径
apachectl -V |grep  DEFAULT_ERRORLOG
```

#####LogLevel
LogLevel 用于调整记于错误日志中的信息的详细程度。
其格式为： LogLevel 错误日志记录等级 
 
|紧急程度  |   等级   |  说明 |
| :--------   | :-----   | :-----   |
|1  |   emerg  |   出现紧急情况使得该系统不可用，如系统宕机等 |
|2   |  alert  |  需要立即引起注意的情况 |
|3   |  crit  |   危险情况的警告 |
|4  |  error|     除了emerg、alert、crit的其他错误 |
|5  |  warn  |   警告信息 |
|6  |   notice  |   需要引起注意的情况，但不如error、warn重要 |
|7  |   info    | 值得报告的一般消息 |
|8  |   debug  |   由运行于debug模式的程序所产生的消息 |