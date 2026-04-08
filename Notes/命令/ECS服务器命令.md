```bash
# 使用 PM2 启动（直接指定入口文件）
pm2 start index.js --name "ai-backend"
# 带参数启动（推荐，限制内存、指定环境）
pm2 start app.js --name ai-backend --node-args="--max-old-space-size=2048" --env production
# 保存当前进程列表
pm2 save
# 生成开机自启动脚本（按提示执行给出的命令）
pm2 startup
# 查看所有 PM2 管理的进程
pm2 list
# 实时监控服务（CPU、内存、日志）
pm2 monit
# 重启指定服务
pm2 restart ai-backend
# 重载服务（零停机，适合热更新）
pm2 reload ai-backend
# 停止指定服务
pm2 stop ai-backend
# 停止所有服务
pm2 stop all
# 删除指定服务（从 PM2 列表中移除）
pm2 delete ai-backend
# 删除所有服务
pm2 delete all
# 防火墙规则 ports:允许的端口
sudo firewall-cmd --list-all
```
