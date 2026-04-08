```bash

# 连接SQL
mysql -u root -p
# 创建用户
CREATE USER 'root'@'xxxxx' IDENTIFIED BY 'Ai@123456';
# 授权命令（最高权限）
GRANT ALL PRIVILEGES ON *.* TO 'root'@'xxxxx' WITH GRANT OPTION;
#刷新权限（立即生效）
FLUSH PRIVILEGES;
# 查询用户权限
SELECT host, user FROM mysql.user;

```
