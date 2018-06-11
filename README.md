[toc]
# 文章
- [Nginx从听说到学会-简书](https://www.jianshu.com/p/630e2e1ca57f)
- [[code.nginx] Nginx配置说明-简书](https://www.jianshu.com/p/d608d67a6cb9)

# 安装
- 下载nginx
  - [选择 stable 版本下载](http://nginx.org/en/download.html)
- 解压
  - ```tar zxf nginx-1.14.0.tar.gz```
- 配置
  - ```./configure --prefix=/usr/local/nginx```
  - 如果提示缺少 pcre 库,则从 http://www.pcre.org/
  - 假设解压在/usr/local/src/pcre
- 再次配置
  - ``` ./configure --prefix=/usr/local/nginx  --with-pcre=/usr/local/src/pcre```
- ```make && make install```
- 启动 nginx
  - ```./sbin/nginx```
- 链接80端口
  - `telnet localhost 80`
- 如Apache占用80端口 杀掉进程
  - `pkill -9 httpd`
- 停止防火墙
  - `service iptables stop`

# 命令参数
- `nginx -t`         作用: 测试配置是否正确
- `nginx -s reload`  作用: 加载最新配置
- `nginx -s stop`    作用: 立即停止
- `nginx -s quit`    作用: 优雅停止
- `nginx -s reopen`  作用: 重新打开日志

# Nginx配置段详解
`nginx.conf`
```nginx
// 有1个工作的子进程,可以自行修改,但太大无益,因为要争夺CPU, 
worker_processes 1;// 一般设置为 CPU数*核数 
Events {
  // 一般是配置nginx连接的特性
  // 如1个worker能同时允许多少连接
  worker_connections 1024; // 这是指 一个子进程最大允许连1024个连接
}

http {
  //这是配置http服务器的主要段
  Server1 {
    // 这是虚拟主机段独处理
  }
  Location{
    //定位,把特殊的路径或文件再次定位,如image目录单
  } // 如.php单独处理
  Server2 {

  }
}
```

# Nginx配置虚拟主机
**例子1: 基于域名的虚拟主机**
```nginx
server {
  listen 80; #监听端口
  server_name a.com; #监听域名,如有多个,空格隔开
  location / {
    root /var/www/a.com; #根目录定位
  }
}
index index.html; #默认索引页
```
**例子2: 基于端口的虚拟主机配置**
```nginx
server {
    listen 8080;
    server_name a.com;
    location / {
        root /var/www/html8080;
    }
}
```

# 支持pathinfo
```nginx
# 典型配置
location ~ \.php$ {
    root           html;
    fastcgi_pass   127.0.0.1:9000;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME  $DOCUMENT_ROOT$fastcgi_script_name;
    include        fastcgi_params;
}

# 修改第1,6行,支持pathinfo

 location ~ \.php(.*)$ { # 正则匹配.php后的pathinfo部分
    root html;
    fastcgi_pass   127.0.0.1:9000;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME  $DOCUMENT_ROOT$fastcgi_script_name;
    fastcgi_param  PATH_INFO $1; # 把pathinfo部分赋给PATH_INFO变量
    include        fastcgi_params;
}
```

# url重写
```nginx
location / {
    root html;
    index index.php index.html;
    if ( !-e $request_filename) {
        rewrite ^/(.*)$ /index.php/$1;
    }
}
```
# try_files
```nginx
location / {
    root html;
    index index.php index.html;
    try_files $uri /index.php?$uri;
}
```
