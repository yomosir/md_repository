# nginx相关基础知识

## nginx
1. 什么是nginx
>Nginx是一款轻量级web服务器，也是一款反向代理服务器。
2. Nginx能干什么
  - 直接支持Rails和PHP程序
  - 可以作为HTTP反向代理服务器
  - 作为负载均衡服务器
  - 作为邮件代理服务器
  - 帮助实现前端动静分离
3. Nginx特点
- 高稳定
- 高性能
- 资源占用少
- 功能丰富（可以使用lua写脚本实现一些功能）
- 模块化结构
- 支持热部署

## nginx安装
1. 安装gcc
2. 安装pcre（pcre-devel）
3. 安装zlib (yum install zlivb zlib-devel)
4. openssl(yum install openssl openssl-devel)
5. 下载源码包，解压缩
6. 进入解压缩后目录，执行`./configure`
> 如果指定安装目录，增加参数`--prefix=/usr/nginx`,默认安装在/usr/local/nginx下
7. 继续执行make
8. 执行make install

## 常用命令
- 测试配置文件 ：安装路径下的/nginx/sbin/niginx -t
- 启动命令 ：安装路径下的/nginx/sbin/nginx
- 停止命令 ：安装路径下/nginx/sbin/nginx -s stop(nginx -s quit)
- 重启命令 ：安装路径下/nginx/sbin/nginx -s reload
- 平滑重启 ：`kill -HUP [NginxPID]`
- 增加防火墙访问权限：
   1. 打开iptables文件
   2. 加入 -A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCPET
   3. 保存退出
   4. 重启防火墙

## Nginx虚拟域名配置及测试验证
1. 编辑 sudo vim /usr/local/nginx/conf/nginx.conf, 添加`include vhost/*.conf`
2. 在/usr/local/nginx/conf/目录新建vhost
3. 重启或启动验证
4. 访问验证：http://localhost:80

>nginx本地使用注意事项：

- 可以配置域名转发，但是一定要配置host，并且使用host生效后才可以，设置完成要重启浏览器
