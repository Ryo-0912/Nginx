まず、Nginxの設定ファイルを開く。

```jsx
sudo nano /etc/nginx/nginx.conf
```

設定ファイルの中で、**`server`** ブロックまたは **`location`** ブロック内で **`root`** ディレクティブを探す。このディレクティブに指定されているパスが、ウェブサーバが探しに行くディレクトリ。

実行結果

```jsx
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    client_max_body_size 50M;
    include	  /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for" '
                      'uid="$upstream_http_x_user"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;

    fastcgi_connect_timeout 3000;
    fastcgi_send_timeout 3000;
    fastcgi_read_timeout 3000;

    proxy_connect_timeout 3000;
    proxy_send_timeout 3000;
    proxy_read_timeout 3000;
}
```

このファイル内では、serverブロックやlocationブロックがなく、その代わりに`include /etc/nginx/conf.d/*.conf;`しているため、こちらの中身を確認。

```jsx
cd /etc/nginx/conf.d/
```

このフォルダ下のファイルを確認。

```jsx
[ec2-user@ip-10-0-24-261 conf.d]$ ls
default.conf  default.conf_24.06  default.conf_bk  php-fpm.conf
```

ファイルの中身確認。

```jsx
nano default.conf
```

実行結果

```jsx
server {

   listen 80;
   server_name _;

   **root** /usr/share/nginx/html/setup/example/public;

   add_header X-Frame-Options "SAMEORIGIN";
   add_header X-XSS-Protection "1; mode=block";
   add_header X-Content-Type-Options "nosniff";

   index index.php;
   charset utf-8;

   location / {
       try_files $uri $uri/ /index.php?$query_string;
   }

   location = /favicon.ico { access_log off; log_not_found off; }
   location = /robots.txt { access_log off; log_not_found off; }

   error_page 404 /index.php;

   location ~ \.php$ {
       fastcgi_pass unix:/var/run/php-fpm/www.sock;
       fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
       fastcgi_hide_header x-user;
       include fastcgi_params;
   }

   location ~ /\.(?!well-known).* {
       deny all;
   }
}
```

serverブロックを発見し、その中にrootを確認。

ここに、デプロイしたファイル群が存在する。
