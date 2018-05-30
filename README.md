﻿# ngx_dynamic_limit_req_module

## Introduction

The *ngx_dynamic_limit_req_module* is used to dynamic lock IP and release regularly.

Configuration example：


    worker_processes  2;
    events {
        worker_connections  1024;
    }
    http {
        include       mime.types;
        default_type  application/octet-stream;
        sendfile        on;
        keepalive_timeout  65;
        
        dynamic_limit_req_zone $binary_remote_addr zone=one:10m rate=100r/s redis=127.0.0.1 block_second=300;
        dynamic_limit_req_zone $binary_remote_addr zone=two:10m rate=50r/s redis=127.0.0.1 block_second=600;
        
        
        server {
            listen       80;
            server_name  localhost;
            location / {
                root   html;
                index  index.html index.htm;
                dynamic_limit_req zone=one burst=5 nodelay;
                dynamic_limit_req_status 403;
            }
            error_page   500 502 503 504  /50x.html;
            location = /50x.html {
                root   html;
            }
        }
        server {
            listen       80;
            server_name  localhost2;
            location / {
                root   html;
                index  index.html index.htm;
                dynamic_limit_req zone=two burst=5 nodelay;
                dynamic_limit_req_status 403;
            }
            error_page   500 502 503 504  /50x.html;
            location = /50x.html {
                root   html;
            }
        }
    }

## Support black-and-white list

###  White list rules
 redis-cli ```set whiteip ip```
 
###  Black list rules 
 redis-cli ```set ip ip ```


## Installation

###  Option #1: Compile Nginx with module bundled
    cd redis-**version**/deps/hiredis
    make 
    make install 
    
    cd nginx-**version**
    ./configure --add-module=/path/to/this/ngx_dynamic_limit_req_module 
    make
    make install


###  Option #2: Compile dynamic module for Nginx

Starting from NGINX 1.9.11, you can also compile this module as a dynamic module, by using the ```--add-dynamic-module=PATH``` option instead of ```--add-module=PATH``` on the ```./configure``` command line above. And then you can explicitly load the module in your ```nginx.conf``` via the [load_module](http://nginx.org/en/docs/ngx_core_module.html#load_module) directive, for example,

```nginx
    load_module /path/to/modules/ngx_dynamic_limit_req_module.so;
```
This module is compatible with following nginx releases:

##FAQ
###```redis connection error: Cannot assign requested address 127.0.0.1```

 At the same time, if there is no need for the external network, we can also let Redis run in the way of Unix Socket to avoid the performance bottleneck of the TCP/IP, and achieve a performance improvement of 25% in high access scenarios

###Modify the configuration file /etc/redis/redis.conf

```unixsocket /var/run/redis/redis.sock```
```unixsocketperm 777```

###or 

Solution on Linux is:
```echo 1 > /proc/sys/net/ipv4/tcp_tw_reuse```
```echo 1 > /proc/sys/net/ipv4/tcp_tw_recycle```

Recommending the first scheme

Author
Gandalf zhibu1991@gmail.com
