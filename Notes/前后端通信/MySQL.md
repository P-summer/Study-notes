[安装教程](https://blog.csdn.net/qq_40907977/article/details/109645908)  
__port number__：默认3306  
富士康PC 3366;window service name：MySQL80;C:\Program Files\MySQL\MySQL Server 8.0\bin  
__查看启动__：services.msc  
__命令连接__：mysql -u root -p -h localhost -P 3366 blog
+ -u 参数后面跟着你的 MySQL 用户名，
+ -p 参数表示需要输入密码来连接到数据库，
+ -h 参数指定主机名为 localhost，
+ -P 参数指定端口号为 3366，your_database_name 是你要连接的数据库的名称。

