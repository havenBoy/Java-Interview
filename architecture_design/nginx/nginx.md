# ningx 环境搭建

> 介绍如何搭建nginx环境，以及配置负载均衡环境

## 环境准备

- gcc环境   yum install gcc-c++ 
- Perl库 正则表达式库 yum install -y pcre pcre-devel
- zlib库 压缩与解压缩库 yum install -y zlib zlib-devel
- openssl 安全套接字密码库  yum install -y openssl openssl-devel
- nginx-1.8.0.tar.gz  安装包
  
  ## 安装过程
- 把安装包解压在 /usr/local/nginx
- 进入解压后的nginx目录， ./configure --prefix=/usr/local/nginx 指定安装的路径
- make && make install 编译
- cd /usr/local/nginx/sbin/nginx 即可启动nginx
- 浏览器输入Linux服务器ip地址即可（注意开放80端口）
- /sbin/iptables -I INPUT -p tcp --dport 80 -j ACCEPT 
- /etc/rc.d/init.d/iptables save
  
  ## 负载均衡服务搭建
