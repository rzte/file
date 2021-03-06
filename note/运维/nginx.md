## 为什么选择Nginx

Nginx 是一个高性能的 Web 和反向代理服务器, 它具有有很多非常优越的特性:

- **作为 Web 服务器**

相比 Apache，Nginx 使用更少的资源，支持更多的并发连接，体现更高的效率，这点使 Nginx 尤其受到虚拟主机提供商的欢迎。能够支持高达 50,000 个并发连接数的响应，感谢 Nginx 为我们选择了 epoll and kqueue 作为开发模型.

- **作为负载均衡服务器**

Nginx 既可以在内部直接支持 Rails 和 PHP，也可以支持作为 HTTP代理服务器 对外进行服务。Nginx 用 C 编写, 不论是系统资源开销还是 CPU 使用效率都比 Perlbal 要好的多。

- **作为邮件代理服务器**

Nginx 同时也是一个非常优秀的邮件代理服务器（最早开发这个产品的目的之一也是作为邮件代理服务器），Last.fm 描述了成功并且美妙的使用经验。

- **Nginx 安装非常的简单，配置文件 非常简洁（还能够支持perl语法），Bugs非常少的服务器**

Nginx 启动特别容易，并且几乎可以做到7*24不间断运行，即使运行数个月也不需要重新启动。你还能够在 不间断服务的情况下进行软件版本的升级。

## 基础配置

### server_tokens

是否在错误界面和请求头中展示版本信息

- syntax: server_tokens on|off

- default: server_tokens on

- context: http, server, location

### types

对于`image/jpeg`、`application/pdf`、`video/mpeg`等这些特殊的类型，为了防止类型解析可以做如下配置：

```
location /download/ {	# 对于 download 路径下的资源要直接下载，而不进行解析
    types        { }	# 清空原有类型映射
    default_type application/octet-stream;	# 设置默认类型为普通响应流
}
```

### 变量

核心模块支持内置变量，其名称与Apache中变量名称相对应

- **$http_user_agent**

- **$http_cookie**

- **$arg_PARAMETER**
	url中的参数（如果有）

- **$args**
	请求行中的参数

- **$binary_remote_addr**
	二进制形式的客户端地址

- **$body_bytes_sent**
	响应时送出的body字节数数量。即使连接中断，这个数据也是精确的。

- **$content_length**
	等于请求头中的Content-Length

- **$cookie_COOKIE**

- **$document_root**
	This variable is equal to the value of directive root for the current request;

- **$document_uri**
	The same as $uri.

- **$host**
	请求主机头字段，否则为服务器名称。

- **$http_HEADER**
	转换为小写并将“破折号”转换为“下划线”时HTTP头

- **$is_args**
	如果有$args参数，这个变量等于”?”，否则等于”"，空值。

- **$limit_rate**
	这个变量可以限制连接速率。

- **$query_string**
	同 $args.

- **$remote_addr**
	The address of the client.

- **$remote_port**
	The port of the client;

- **$remote_user**
	Auth Basic认证时的用户名

- **$request_filename**
	此变量等于当前请求的文件路径，由指令root或别名和URI请求组成;

- **$request_body**
	请求体,此变量的重要性出现在具有指令proxy_pass或fastcgi_pass的位置

- **$request_body_file**
	请求体的临时文件

- **$request_completion**
	 如果请求结束，设置为OK. 当请求未结束或如果该请求不是请求链串的最后一个时，为空(Empty)

- **$request_method**
	请求方法：GET、POST...

- **$request_uri**
	请求uri

- **$scheme**
	schema，例：
	```conf
	rewrite  ^(.+)$  $scheme://example.com$1  redirect;
	```

- **$server_addr**
	服务器地址。通常，为获得此变量的值，需要进行一次系统调用。为了避免系统调用，有必要在指令listen中指示地址并使用参数bind

- **$server_name**
	服务器名

- **$server_port**
	服务端口

- **$server_protocol**
	协议：`HTTP/1.0` or `HTTP/1.1`

- **$uri**
	当前的uri，可能与$request_uri不同。$request_uri是浏览器发过来的，$uri是重定向后的uri

### Upstream（轮询、负载均衡）

```conf
upstream backend  {
  server backend1.example.com weight=5;
  server backend2.example.com:8080;
  server unix:/tmp/backend3;
}

server {
  location / {
    proxy_pass  http://backend;
  }
}s
```

**ip_hash**

```conf
upstream backend {
  ip_hash;		# 使请求根据客户端的IP地址在上游之间分配，hash的关键在于客户端的C类地址
  server   backend1.example.com;
  server   backend2.example.com;
  server   backend3.example.com  down;
  server   backend4.example.com;
}
```

### HttpAccess模块

ngx_http_access_module 模块使有可能对特定IP客户端进行控制. 规则检查按照第一次匹配的顺序

```conf
location / {
: deny    192.168.1.1;
: allow   192.168.1.0/24;
: allow   10.1.1.0/16;
: deny    all;
}
```

### Http Referer模块

验证referer

```conf
location /photos/ {
  valid_referers none blocked www.mydomain.com mydomain.com;	# 设置有效的referer，这里只允许不带referer(none) 或referer不合法（blocked）或 referer为mydomain.com等域名

  if ($invalid_referer) {
    return   403;
  }
}
```

**valid_referers**

这个指令基于请求头 referer 为 **$invalid_referer** 赋值 0 或 1，如果referer匹配到了，则**$invalid_referer**为0。也就是说，在valid_referers上设置有效的referer

- syntax: valid_referers [none|blocked|server_names] ...
	- none 没有referer头（比如直接在浏览器打开一个链接）
	- blocked 代表不合法的referer，如 Referer: www.baidu.com  （正常应该是 Referer: http://www.baidu.com）
	- server_names 列出一个或多个域名
- default: none
- context: server, location

**防盗链示例**

```conf
location ~* \.(gif|jpg|png|swf|flv)$ {
	root html;		# 如果在server{}中有设置可以不需要设定
	valid_referers none blocked *.nginxcn.com;
	if ($invalid_referer) {
		rewrite ^/ www.nginx.cn;
		#return 404;
	}
}
```

### HttpProxy模块

该模块可以将请求导向另一台服务器，使用 HTTP/1.0 ，没有保持请求的能力

注意一点,当使用HTTP PROXY 模块时(或者甚至是使用FastCGI时),用户的整个请求会在nginx中缓冲直至传送给后端被代理的服务器.因此,上传进度的测算就会运作得不正确,如果它们通过测算后端服务器收到的数据来工作的话

```conf
location / {
: proxy_pass        http://localhost:8000;
: proxy_redirect   http://localhost:8000/ /;	# 若8000端口的服务器发送给客户端的是重定向请求（比如重定向到 http://localhost:8000/)，可通过 proxy_redirect 将请求改为 /
: proxy_set_header  X-Real-IP  $remote_addr;
}
```

一些定向器、参数：[HttpProxy模块](http://www.nginx.cn/doc/standard/httpproxy.html)


### HttpRewrite模块

rewrite的主要功能是实现URL地址的重定向,指令根据配置文件中的顺序来执行。该功能需要PCRE支持

rewrite是实现URL重写的关键指令，根据regex（正则表达式）部分内容，重定向到replacement，结尾是flag标记

- 语法: rewrite regex replacement flag
- 默认: none
- 作用域: server, location, if

flag标记说明：

- last  #本条规则匹配完成后，继续向下匹配新的location URI规则
- break  #本条规则匹配完成即终止，不再匹配后面的任何规则
- redirect  #返回302临时重定向，浏览器地址会显示跳转后的URL地址
- permanent  #返回301永久重定向，浏览器地址栏会显示跳转后的URL地址

注意重写表达式只对相对路径有效。如果想配对主机名，你应该使用if语句，如下：

```conf
if ($host ~* www\.(.*)) {	# 此处用if匹配主机名
: set $host_without_www $1;
: rewrite ^(.*)$ http://$host_without_www$1 permanent;   # $1 匹配 '/foo', 而不是 'www.mydomain.com/foo'
}
```

### return

这个指令根据规则的执行情况，返回一个状态值给客户端。可使用值包括：204，400，402-406，408，410，411，413，416以及500-504。也可以发送非标准的444代码-未发送任何头信息下结束连接。

location /download/ {
: rewrite  ^(/download/.*)/media/(.*)\..*$  $1/mp3/$2.mp3  break;
: rewrite  ^(/download/.*)/audio/(.*)\..*$  $1/mp3/$2.ra   break;
: return   403;
}

### Map

- syntax: map $var1 $var2 { ... }

- context: http

map 指令是由 ngx_http_map_module 模块提供的，默认情况下安装 nginx 都会安装该模块。

map 的主要作用是创建自定义变量，通过使用 nginx 的内置变量，去匹配某些特定规则，如果匹配成功则设置某个值给自定义变量。 而这个自定义变量又可以作于他用。

```conf
map  $http_host  $name  {
  hostnames;			# 它允许更容易地匹配诸如主机名之类的值，具有起始点的名称可以匹配确切的主机名和以该值结尾的主机名
  default          0;	# 默认
  example.com      1;	# $http_host 匹配到 example.com 时， $name的值为1
  *.example.com    1;
  test.com         2;
  *.test.com       2;	# $http_host 匹配到 *.test.com 时， $name的值为2
  .site.com        3;
}
```

```conf
map $uri $new {		# 根据 $uri 决定 $new的值
  default        http://www.domain.com/home/;	# $new默认值

  /aa            http://aa.domain.com/;			# $uri为 /aa 时的new的值
  /bb            http://bb.domain.com/;
  /john          http://my.domain.com/users/john/;
}

server {
  server_name   www.domain.com;
  rewrite  ^    $new   redirect;	# 这里 $new 会依据上面的 map 决定
}
```

### HttpUserId

userid模块用来标识用户，为连接设置cookie

Context：http,server,location

```conf
userid          on;
userid_name     uid;

userid_domain   example.com;
userid_path     /;
userid_expires  365d;
userid_p3p      'policyref="/w3c/p3p.xml", CP="CUR ADM OUR NOR STA NID"';			# 为和cookie一起传递的P3P头指定一个值。
```

### Addition模块

这个模块可以在相应体前后增加些内容

```conf
location / {
  add_before_body   /before_action;			# 将 /before_action 的内容放在原响应体前面
  add_after_body    /after_action;			# 将 /after_action 的内容追加在响应体后面
}
```

- add_before_body
	- syntax: add_before_body uri
	- default: no
	- context: http, server, location

- add_after_body
	- syntax: add_after_body uri
	- default: no
	- context: http, server, location

- addition_types(允许增加让该location处理的其他MIME类型（默认为"text/html"）)
	- syntax: addition_types mime-type [mime-type ...]
	- default: text/html
	- context: http, server, location

### Memcached

nginx的memcached_module模块可以直接从memcached服务器中读取内容后输出，后续的请求不再经过应用程序处理

```conf
server {
 location / {
	 set  $memcached_key  $uri;
	 memcached_pass   name:11211;
	 default_type     text/html;
	 error_page       404 = /fallback;
 }

 location = /fallback {
 	proxy_pass       backend;
 }
}
```

**指令**

- [#memcached_pass memcached_pass]

- [#memcached_connect_timeout memcached_connect_timeout]

- [#memcached_send_timeout memcached_send_timeout]

- [#memcached_read_timeout memcached_read_timeout]

- [#memcached_buffer_size memcached_buffer_size]

- [#memcached_next_upstream memcached_next_upstream]

**$memcached_key**

memcached的key可以通过memcached_key变量来设置，如以$uri。如果命中，那么直接输出内容，没有命中就意味着nginx需要从应用程序请求页面。同时，我们还希望该应用程序将键值对写入到memcached，以便下一个请求可以直接从memcached获取。

**memcached_pass**

指定memcached服务器地址。使用变量$memcached_key为key查询值，如果没有相应的值则返回error_page 404。

- 语法：memcached_pass address:port or socket；
- 默认值：none
- 配置段：location, if in location

**memcached_connect_timeout**

与memcached服务器建立连接的超时时间。通常不超过75s。

- 语法：memcached_connect_timeout time;
- 默认值：60s;
- 配置段：http, server, location

**memcached_read_timeout**

定义从memcached服务器读取响应超时时间。

- 语法：memcached_read_timeout time;
- 默认值：60s;
- 配置段：http, server, location

**memcached_send_timeout**

设置发送请求到memcached服务器的超时时间。

- 语法：memcached_send_timeout time;
- 默认值：60s
- 配置段：http, server, location

**memcached_buffer_size**

读取从memcached服务器接收到响应的缓冲大小。尽快的将响应同步传给客户端。

- 语法: memcached_buffer_size size;
- 默认值: 4k|8k;
- 配置段: http, server, location

**memcached_next_upstream**

指定在哪些状态下请求将转发到另外的负载均衡服务器上，仅当memcached_pass有两个或两个以上时使用。

- 语法: memcached_next_upstream error | timeout | invalid_response | not_found | off ...;
- 默认值： error timeout;
- 配置段: http, server, location

### If 判断

- 语法: if (condition) { ... }
- 默认: none
- 作用域: server, location

若 condition 为真，则执行 代码块 {} 中的内容

- 若为变量
	false的值为：空字符串 "" 或者任何以 0 开头的字符串

- 字符串比较
	使用 `=` 和 `!=` 操作符

- 正则比较
	- `~、!~`
	大小写敏感

	- `~*、!~*`
	大小写不敏感

- `-f`、`!-f`
	检查文件是否存在

- `-d`、`!-d`
	检查目录是否存在

- `-e`、`!-e`
	检查是否存在一个文件、一个目录或是一个符号链接

- `-x`、`!-x`
	检查是否存在一个文件是否可执行

正则表达式一部分可以在括号中，稍后在块中可以访问其值

```conf
if ($http_user_agent ~ MSIE) {
: rewrite  ^(.*)$  /msie/$1  break;
}
if ($http_cookie ~* "id=([^;] +)(?:;|$)" ) {
: set  $id  $1;
}
if ($request_method = POST ) {
: return 405;
}
if (!-f $request_filename) {
: break;
: proxy_pass  http://127.0.0.1;
}
if ($slow) {
: limit_rate  10k;
}
if ($invalid_referer) {
: return   403;
}
```

## 附加配置

### SSL支持

默认情况下，模块不是构建的，必须指定使用--with-http_ssl_module参数构建它来./configure。构建此模块需要OpenSSL库和包含文件，通常是单独包中的必要文件。

```conf
worker_processes 1;
http {

  server {
    listen               443;
    ssl                  on;
    ssl_certificate      /usr/local/nginx/conf/cert.pem;
    ssl_certificate_key  /usr/local/nginx/conf/cert.key;
    keepalive_timeout    70;
  }

}
```

#### 生成证书

生成虚拟证书，可使用以下几步：

```bash
$ cd /usr/local/nginx/conf
$ openssl genrsa -des3 -out server.key 1024
$ openssl req -new -key server.key -out server.csr
$ cp server.key server.key.org
$ openssl rsa -in server.key.org -out server.key
$ openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```

配置 nginx.conf

```conf
server {
    server_name YOUR_DOMAINNAME_HERE;
    listen 443;
    ssl on;
    ssl_certificate /usr/local/nginx/conf/server.crt;
    ssl_certificate_key /usr/local/nginx/conf/server.key;

}
```

#### 指令

**ssl**

为server开启SSL

- syntax: ssl [on|off]
- default: ssl off
- context: main, server

**ssl_certificate**

PEM证书（公钥）。从版本0.6.7开始，文件路径相对于nginx配置文件nginx.conf的目录，而不是nginx前缀目录。

- syntax: ssl_certificate file
- default: ssl_certificate cert.pem
- context: main, server

**ssl_certificate_key**

PEM密钥文件（私钥）。从版本0.6.7开始，文件名路径相对于nginx配置文件nginx.conf的目录，而不是nginx前缀目录。

- syntax: ssl_certificate_key file
- default: ssl_certificate_key cert.pem
- context: main, server

**ssl_client_certificate**

CA证书，用于检查客户端证书。

- syntax: ssl_client_certificate file
- default: none
- context: main, server

**ssl_dhparam**

表示具有PEM格式的Diffie-Hellman参数的文件，用于协商TLS会话密钥。

- syntax: ssl_dhparam file
- default: none
- context: main, server

**ssl_ciphers**

设置加密套件

- syntax: ssl_ciphers file
- default: ssl_ciphers ALL:!ADH:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP
- context: main, server

**ssl_crl**

指定证书吊销列表（CRL）文件，该文件用于检查客户端证书。

- syntax: ssl_crl file
- default: none
- context: http, server

**ssl_prefer_server_ciphers**

设置协商加密算法时，优先使用我们服务端的加密套件，而不是客户端浏览器的加密套件

- syntax: ssl_prefer_server_ciphers [on|off]
- default: ssl_prefer_server_ciphers off
- context: main, server

**ssl_protocols**

启动特定的加密协议,TLSv1.1与TLSv1.2要确保OpenSSL >= 1.0.1 ，SSLv3 现在还有很多地方在用但有不少被攻击的漏洞。

- syntax: ssl_protocols [SSLv2] [SSLv3] [TLSv1]
- default: ssl_protocols SSLv2 SSLv3 TLSv1
- context: main, server

**ssl_verify_client**

验证客户端证书。参数'ask'检查客户端证书是否已提供。

- syntax: ssl_verify_client on|off|ask
- default: ssl_verify_client off
- context: main, server

**ssl_verify_depth**

设置客户端证书链中的深度检查。

- syntax: ssl_verify_depth number
- default: ssl_verify_depth 1
- context: main, server

**ssl_session_cache**

设置ssl/tls会话缓存的类型和大小。如果设置了这个参数一般是shared，buildin可能会参数内存碎片，默认是none，和off差不多，停用缓存。如 `shared：SSL:10m` 表示我所有的nginx工作进程共享ssl会话缓存，官网介绍说1M可以存放约4000个sessions

- syntax: ssl_session_cache off|none|builtin:size and/or shared：name:size
- default: ssl_session_cache off
- context: main, server

**ssl_session_timeout**

客户端可以重用会话缓存中ssl参数的过期时间，内网系统默认5分钟太短了，可以设成30m即30分钟甚至4h

- syntax: ssl_session_timeout time
- default: ssl_session_timeout 5m
- context: main, server

这个模块支持几个非标准错误码，可借助 error_page 调试

- 495 - error checking client certificate
- 496 - client did not grant the required certificate
- 497 - normal request was sent to HTTPS

**ssl_engine**

设置要使用的OpenSSL引擎，比如 Padlock，它需要新版本的OpenSSL

syntax: ssl_engine

### HttpSubstitution

这个模块可以搜索替换响应中的信息，在编译nginx时必需加上`--with-http_sub_module option`

```conf
location / {
  sub_filter      </head>
  '</head><script language="javascript" src="$script"></script>';
  sub_filter_once on;
}
```

**sub_filter**

sub_filter 允许替换源文件里的多个文本（多次替换）匹配是非常快速的。替换必须包含变量，一个location只能一个替换规则.

- syntax: sub_filter text substitution
- default: none
- context: http, server, location

**sub_filter_once**

sub_filter_once off 允许查找替换所有匹配行，默认只替换第一个.

- syntax: sub_filter_once on|off
- default: sub_filter_once on
- context: http, server, location

**sub_filter_types**

sub_filter_types用于指定替换sub_filter的类型，默认为text/html.

- syntax: sub_filter_types mime-type [mime-type ...]
- default: sub_filter_types text/html
- context: http, server, location

## 邮件配置

Nginx能够处理和代理以下邮件协议：

- IMAP
	IMAP全称是Internet Mail Access Protocol，即交互式邮件存取协议，它是跟POP3类似邮件访问标准协议之一。不同的是，开启了IMAP后，您在电子邮件客户端收取的邮件仍然保留在服务器上，同时在客户端上的操作都会反馈到服务器上，如：删除邮件，标记已读等，服务器上的邮件也会做相应的动作。所以无论从浏览器登录邮箱或者客户端软件登录邮箱，看到的邮件以及状态都是一致的

- POP3
	POP3是Post Office Protocol 3的简称，即邮局协议的第3个版本,它规定怎样将个人计算机连接到Internet的邮件服务器和下载电子邮件的电子协议。它是因特网电子邮件的第一个离线协议标准,POP3允许用户从服务器上把邮件存储到本地主机（即自己的计算机）上,同时删除保存在邮件服务器上的邮件

- SMTP
	SMTP 的全称是“Simple Mail Transfer Protocol”，即简单邮件传输协议。它是一组用于从源地址到目的地址传输邮件的规范，通过它来控制邮件的中转方式。SMTP 协议属于 TCP/IP 协议簇，它帮助每台计算机在发送或中转信件时找到下一个目的地



## 参考
> [nginx中文文档](http://www.nginx.cn/doc/)
>
> [nginx+memcache实现页面缓存应用](https://www.cnblogs.com/lpfuture/p/5800042.html)
>
> [Nginx URL重写（rewrite）配置及信息详解](https://www.cnblogs.com/czlun/articles/7010604.html)
>
> [nginx配置ssl加密](https://segmentfault.com/a/1190000002866627)




















