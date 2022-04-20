```nginx
# 工作进程：数目。根据硬件挑战，通常为CPU数量或者2倍于CPU
worker_processes  1;

# 错误日志存放路径
error_log  logs/error.log;
# pid（进程标识符）：存放路径。
pid logs/nginx.pid;


events {
    #每个工作进程的最大连接数量。根据硬件调整，和前面工作进程配合起来用，尽量大，但是别把cpu跑到100%就行。每个进程允许的最多连接数，理论上每台nginx服务器的最大连接数为。worker_processes*worker_connections
    worker_connections  1024;
    #keepalive超时时间。
    keepalive_timeout 60;
}

# 设定http服务器，利用它的反向代理功能提供负载均衡支持
http {
    #设定mime类型,类型由mime.type文件定义
    include       mime.types;
    default_type  application/octet-stream;

    access_log off;
    sendfile        on;

    keepalive_timeout  65;
    server_tokens off;
    gzip on;
    gzip_min_length  5k;
    gzip_buffers     4 16k;
    gzip_http_version 1.0;
    gzip_comp_level 9;
    gzip_types    application/javascript   text/plain application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
    gzip_vary on;
    #设置代理服务器列表，默认为轮询
    upstream saleApp {
        server  133.0.189.240:9990;
        server  133.0.189.241:9990;
        server  133.0.189.242:9990;
    }
    server {
        # 设置监听端口
        listen       9995;
        server_name  saleApp;
        ssi    on;

        client_max_body_size 30M;
        client_body_buffer_size 5M;

        location ~* \.(css|flash|media|jpg|png|gif|dll|cab|CAB|ico|woff|woff2|ttf|eot|svg|pdf|bcmap|properties)$ {
            root   /home/saleCenter/release;
            index  login.html login.htm;
            client_body_timeout 20s;
            expires  30d;
        }
        location ~* \.(js|html|htm)$ {
            root   /home/saleCenter/release;
            index  login.html login.htm;
            add_header Cache-Control  max-age=no-cache;
            expires  1s;
        }
        location /saleCenterApp{
            proxy_connect_timeout 60s;
            proxy_send_timeout 90;
            proxy_read_timeout 120;
            proxy_buffer_size 256k;
            proxy_buffers 4 256k;
            proxy_busy_buffers_size 256k;
            proxy_temp_file_write_size 256k;
            proxy_next_upstream error timeout invalid_header http_500 http_503 http_404;
            proxy_max_temp_file_size 128m;
            #proxy_buffering off;
            proxy_redirect off;
            proxy_set_header Host $host;
            proxy_set_header Cache-Control max-age=1;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            add_header  location_gray 'location_rest_v1'; 
            proxy_pass  http://saleApp;
        }
        location /login{
            proxy_connect_timeout 60s;
            proxy_send_timeout 90;
            proxy_read_timeout 120;
            proxy_buffer_size 256k;
            proxy_buffers 4 256k;
            proxy_busy_buffers_size 256k;
            proxy_temp_file_write_size 256k;
            proxy_next_upstream error timeout invalid_header http_500 http_503 http_404;
            proxy_max_temp_file_size 128m;
            #proxy_buffering off;
            proxy_redirect off;
            proxy_set_header Host $host;
            proxy_set_header Cache-Control no-cache;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            add_header  location_gray 'location_rest_v1'; 
            # 将请求转向saleApp定义的服务器列表
            proxy_pass  http://saleApp;	
        }

        error_page 405 =200 $uri;

        error_page   400 404 413 502 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```



#### 代理分发策略

```nginx
#1.默认策略：轮询
upstream saleApp {
    server  133.0.189.240:9990;
    server  133.0.189.241:9990;
    server  133.0.189.242:9990;
}

#2.设置代理服务器并且配置权重，会根据权重将请求进行一个分发
upstream myservers {   
    server 133.0.189.240:9990 weight=1;
    server 133.0.189.241:9990 weight=3;
    server 133.0.189.242:9990 weight=9;
}

#3.每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。
upstream myservers {   
    ip_hash;
    server 133.0.189.240:9990 weight=1;
    server 133.0.189.241:9990 weight=3;
    server 133.0.189.242:9990 weight=9;
}

#4.fair（第三方）按后端服务器的响应时间来分配请求，响应时间短的优先分配。
upstream backend {
    server 133.0.189.240:9990;
    server 133.0.189.241:9990;
    server 133.0.189.242:9990
    fair;
}

#5、url_hash（第三方）
#按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。
#例：在upstream中加入hash语句，server语句中不能写入weight等其他的参数，hash_method是使用的hash算法
upstream backend {
    server 133.0.189.240:9990;
    server 133.0.189.241:9990;
    server 133.0.189.242:9990;
    hash $request_uri;
    hash_method crc32;
}
```

