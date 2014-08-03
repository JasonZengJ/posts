title: Nginx 安装ngx_log_if第三方模块
date: 2014-08-04 01:36:36
tags: Nginx
---

最近忽然想打开Nginx的日志记录了，但是每一次访问至少要生成2.9k的记录，啥js,css,image等文件的每一个请求都单独记录了一条数据，对于我那硬盘少的可怜的仅20GB的服务器，访问个上百万次，几G就没了。于是乎上网搜了一个叫做ngx_log_if的Nginx第三方模块，可以用来根据匹配规则来决定哪些数据请求不需要记录日志。


###一、下载ngx_log_if
github地址：[ngx_log_if](https://github.com/cfsego/ngx_log_if/)

###二、安装ngx_log_if模块

1、进入nginx源码目录，我的是在/home下

2、看到有一个configure的可执行文件，嘿嘿，就是它了！按照官网所描述的添加第三方模块的方法：

Modules are typically added by compiling them along with the Nginx source.

From the Nginx source directory, type:

```
./configure --add-module=/path/to/module1/source \
            --add-module=/path/to/module2/source
```

You can use as many --add-module arguments as needed.

Be aware that some modules may require additional libraries to be installed on your system.

由以上说明可知，所以要加入第三方模块，要跟着ngxin源码一起编译，进入nginx源码目录( [Ngxin源码下载](http://nginx.org/en/download.html) )，敲入一下命令:

```
 ./configure --add-module=/home/ngx_log_if  //ngx_log_if 是目录名
```

结果悲剧的提示说：

```
checking for PCRE library ... not found
checking for PCRE library in /usr/local/ ... not found
checking for PCRE library in /usr/include/pcre/ ... not found
checking for PCRE library in /usr/pkg/ ... not found
checking for PCRE library in /opt/local/ ... not found

./configure: error: the HTTP rewrite module requires the PCRE library.
You can either disable the module by using --without-http_rewrite_module
option, or install the PCRE library into the system, or build the PCRE library
statically from the source with nginx by using --with-pcre=<path> option.
```

因为configure功能使用到了nginx的rewrite模块，而nginx的rewrite模块需要PCRE库，所以又跑去装PCRE..

```
cd /usr/local/src
wget http://downloads.sourceforge.net/pcre/pcre-8.10.tar.bz2
tar -jxf pcre-8.10.tar.bz2 
cd pcre-8.10
./configure
make 
make install
```

很幸运，编译安装成功了！鲜花，鼓掌！然后再次尝试之前配置三方模块的命令，结果也顺利配置好了.

```
Configuration summary
  + using system PCRE library
  + OpenSSL library is not used
  + md5: using system crypto library
  + sha1: using system crypto library
  + using system zlib library

  nginx path prefix: "/usr/local/nginx"
  nginx binary file: "/usr/local/nginx/sbin/nginx"
  nginx configuration prefix: "/usr/local/nginx/conf"
  nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
  nginx pid file: "/usr/local/nginx/logs/nginx.pid"
  nginx error log file: "/usr/local/nginx/logs/error.log"
  nginx http access log file: "/usr/local/nginx/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"

```

ok,大功搞成，剩下的就是编译安装nginx了，小case嘛...

```
cd /home/nginx-1.5.8
make
make install
```
...
......
.........
等的黄花菜都凉了，终于编译安装好了,Perfect！接下来就是功能使用啦！功能如何使用，在ngx_log_if的README.md文档里面说明的比较详细：

This directive allows you to define multiple conditions in the form of using the directive in multiple times in the same block. For example:

```
server {
    access_log_bypass_if ($status = 400);
    access_log_bypass_if ($host ~* 'nolog.com');
    access_log_bypass_if ($uri = 'status.nginx') and;
    access_log_bypass_if ($status = 200);
}
```

In the case, nginx will not write access log when the status code is 400, or when the host is 'nolog.com', or when the uri is 'status.nginx' and the status code is 200.

However, if you define them both in the blocks in the father child relationship, the child block will not inherit and merge the configuration in parent block, of course. FOr example:

```
server {
    access_log_bypass_if ($status = 400);

    location / {
        access_log_bypass_if ($host ~* 'nolog.com');
    }
}
```

In this case above, in location "/", only "($host ~* 'nolog.com')" is available, but "($status = 400)" is ignored and makes no difference.

果断开始给nginx日志瘦身哒！







