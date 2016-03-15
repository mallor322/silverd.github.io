---
layout: post
title: Nginx 配置 SSL 证书支持 Https
---

确保机器上安装了 openssl 和 openssl-devel

    yum install openssl
    yum install openssl-devel
    
确保 Nginx 只是 SSL 模块，编译时带 --with-http_ssl_module 参数，否则会报错

    [emerg] 10464#0: unknown directive "ssl" in /usr/local/nginx/conf/nginx.conf:74”

创建服务端私钥（第三方 SSL 证书签发机构都要求起码 2048 位的 RSA 加密的私钥）

    cd /usr/local/nginx/conf
    openssl genrsa -des3 -out server.key 2048
    
创建证书请求文件（CSR = SSL Certificate Signing Request）

    openssl req -new -key server.key -out server.csr
    
依次输入国家代码 CN，省份城市、邮箱等信息即可。

去除私钥里的密码信息（否则以SSL启动Nginx时会提示必须输入密钥）

    cp server.key server.key.org
    openssl rsa -in server.key.org -out server_nopwd.key

使用刚生成的私钥和CSR进行证书签名
或者到 StartSSL 上传 CSR 后获得经 CA 机构签名后的证书，如：1_study.hicrew.cn_bundle.crt

    openssl x509 -req -days 365 -in server.csr -signkey server_nopwd.key -out server.crt

修改Nginx配置文件，让其包含新标记的证书和私钥：

    server 
    {
        server_name YOUR_DOMAINNAME_HERE;
        listen 443;
        ssl on;
        ssl_certificate /usr/local/nginx/conf/server.crt;
        ssl_certificate_key /usr/local/nginx/conf/server_nopwd.key;
    }

另外还可以加入如下代码实现80端口重定向到443

    server 
    {
        listen 80;
        server_name YOUR_DOMAINNAME_HERE;
        rewrite ^(.*) https://$server_name$1 permanent;
    }
    
重启 nginx 就可以通过以下方式访问了

  https://YOUR_DOMAINNAME_HERE
  
  
## FAQ

问：如何让浏览器信任自己办法的证书，以IE为例：
答：控制面板 -> Internet选项 -> 内容 -> 发行者 -> 受信任的根证书颁发机构 -> 导入 -> 选择 server.crt

问：什么是针对企业的 EV SSL
答：EV SSL，是 Extended Validation 的简称，更注重于对企业网站的安全保护以及严格的认证。最明显的区别就是，通常 EV SSL 显示都是绿色的条

=-=====-=====-=====-====
参考文章：
  
    - <https://www.centos.bz/2011/12/nginx-ssl-https-support/>
    - <http://blog.weiliang.org/linux/632.html>
    - [Nginx 配置 SSL 证书 + 搭建 HTTPS 网站教程](http://www.open-open.com/lib/view/open1433390156947.html)