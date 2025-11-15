Xboard aaPanel 环境部署指南目录环境要求快速部署详细配置维护指南故障排除环境要求硬件要求CPU：1 核或更高内存：2GB 或更高存储空间：10GB+ 可用空间软件要求操作系统：Ubuntu 20.04+ / Debian 10+ (⚠️ 不推荐使用 CentOS 7)最新版本的 aaPanel (宝塔面板)PHP 8.2MySQL 5.7+RedisNginx (任意版本)快速部署1. 安装 aaPanel (宝塔面板)URL=[https://www.aapanel.com/script/install_6.0_en.sh](https://www.aapanel.com/script/install_6.0_en.sh) && \
if [ -f /usr/bin/curl ];then curl -ksSO "$URL" ;else wget --no-check-certificate -O install_6.0_en.sh "$URL";fi && \
bash install_6.0_en.sh aapanel
2. 基础环境设置2.1 安装 LNMP 环境在 aaPanel 面板中，安装：Nginx (任意版本)MySQL 5.7PHP 8.22.2 安装 PHP 扩展必需的 PHP 扩展：redisfileinfoswoolereadlineeventmbstring2.3 启用必需的 PHP 函数需要启用的函数：putenvproc_openpcntl_alarmpcntl_signal3. 站点配置3.1 创建网站导航至：aaPanel > 网站 (Website) > 添加站点 (Add site)填写信息：域名 (Domain)：输入您的站点域名数据库 (Database)：选择 MySQLPHP 版本 (PHP Version)：选择 8.23.2 部署 Xboard# 进入站点目录
cd /www/wwwroot/your-domain

# 清理目录
chattr -i .user.ini
rm -rf .htaccess 404.html 502.html index.html .user.ini

# 克隆仓库
git clone [https://github.com/cedar2025/Xboard.git](https://github.com/cedar2025/Xboard.git) ./

# 安装依赖
sh init.sh
3.3 配置站点将运行目录 (Running directory) 设置为 /public添加重写规则 (Rewrite rules)：location /downloads {
}

location / {  
    try_files $uri $uri/ /index.php$is_args$query_string;  
}

location ~ .*\.(js|css)?$
{
    expires      1h;
    error_log off;
    access_log /dev/null; 
}
详细配置1. 配置守护进程安装 Supervisor添加队列守护进程：名称 (Name)：Xboard运行用户 (Run User)：www运行目录 (Running Directory)：站点目录启动命令 (Start Command)：php artisan horizon进程数量 (Process Count)：12. 配置计划任务类型 (Type)：Shell 脚本任务名称 (Task Name)：v2board运行用户 (Run User)：www频率 (Frequency)：1 分钟脚本内容 (Script Content)：php /www/wwwroot/site-directory/artisan schedule:run3. Octane 配置 (可选)3.1 添加 Octane 守护进程名称 (Name)：Octane运行用户 (Run User)：www运行目录 (Running Directory)：站点目录启动命令 (Start Command)：/www/server/php/82/bin/php artisan octane:start --port 7010进程数量 (Process Count)：13.2 Octane 专用重写规则location ~* \.(jpg|jpeg|png|gif|js|css|svg|woff2|woff|ttf|eot|wasm|json|ico)$ {
}

location ~ .* {
    proxy_pass [http://127.0.0.1:7010](http://127.0.0.1:7010);
    proxy_http_version 1.1;
    proxy_set_header Connection "";
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Real-PORT $remote_port;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_set_header Scheme $scheme;
    proxy_set_header Server-Protocol $server_protocol;
    proxy_set_header Server-Name $server_name;
    proxy_set_header Server-Addr $server_addr;
    proxy_set_header Server-Port $server_port;
}
维护指南版本更新# 进入站点目录
cd /www/wwwroot/your-domain

# 执行更新脚本
git fetch --all && git reset --hard origin/master && git pull origin master
sh update.sh

# 如果启用了 Octane，请重启守护进程
# aaPanel > 软件商店 (App Store) > 工具箱 (Tools) > Supervisor > 重启 Octane
日常维护定期检查日志监控系统资源使用情况定期备份数据库和配置文件故障排除常见问题管理员路径更改后需要重启服务才能生效启用 Octane 后任何代码更改都需要重启才能生效PHP 扩展安装失败时，请检查 PHP 版本是否正确数据库连接失败时，请检查数据库配置和权限
