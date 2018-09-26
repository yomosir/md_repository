# VSFTP
    在linux系统上安装ftp服务
## 安装过程
#### 通过yum进行安装

执行```yum -y install yum```，默认的配置文件在  
   
    /etc/vsftp/vsftp.conf

#### 创建ftp创建虚拟用户
1. 在用户目录或者根目录下创建ftp文件夹
2. 添加匿名用户，无登录权限 ```useradd ftpuser -d /ftpfile -s /sbin/nologin```
3. 修改ftpfile文件夹的权限为ftpuser
4. 重设ftpuser的密码：```passwd ftpuser```

#### 配置
1. 到```/etc/vsftp```目录下
2. 创建新文件 chroot_list
3. 把刚才新增的虚拟用户添加到此配置文件中，后续要引用
4. ```sudo vim /etc/selinux/config```，修改SELINUX=disabled   
```如果验证的时候碰到550拒绝访问的时候可执行 sudo setsebool -P ftp_home_dir 1``` 
5. 配置vsftp.conf
6. 防火墙配置
 防火墙配置文件为```/etc/sysconfig/iptables```
```bash
-A INPUT -p TCP --dport 61001:62000 -j ACCEPT
-A OUTPUT -p TCP --sport 61001:62000 -j ACCEPT

在配置上20和21端口

```
最后重启防火墙

#### 验证
1. 先启动ftp服务
2. 查找ip
3. 用浏览器访问ftp服务