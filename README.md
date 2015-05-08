How to use MCFI/PICFI to harden nginx-1.4.0 and support word press.

0. You need to compile MCFI/PICFI toolchain!

1. See nginx.sh for how nginx.sh is build.

2. Install mysql, php5-mysql and php5-fpm by
   
   sudo apt-get install mysql-server php5-mysql php5-fpm php5-gd libssh2-php

3. In config file (/etc/php5/fpm/pool.d/www.conf) for php5-fpm,
   change "user" to the user name of nginx's executable;
   uncomment listen.allowed_clients = 127.0.0.1 to allow local connections.
   su && chmod o+rw php5-fpm.sock # make this socket connectable

4. Create a MySQL database
   mysql -u root -p
   CREATE DATABASE wordpress;
   CREATE USER wordpressuser@localhost IDENTIFIED BY 'password';
   GRANT ALL PRIVILEGES ON wordpress.* TO wordpressuser@localhost;
   FLUSH PRIVILEGES;
   exit

5. Download wordpress and decompress it to $WORDPRESS.
   cd $WORDPRESS
   cp wp-config-sample.php wp-config.php

   Set mysql credentials below:
   define('DB_NAME', 'wordpress');
   define('DB_USER', 'wordpressuser');
   define('DB_PASSWORD', 'password');

   # copy wordpress files to nginx installation dir's html subdir
   rsync -avP $WORDPRESS/ $NGINX/html/
   
   mkdir $NGINX/html/wp-content/uploads

6. Use the following configuration file for nginx. Start nginx and access
   the front page by localhost/wp-admin/install.php

=====================================


#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        root   /home/ben/MCFI/server/html;

        index  index.php index.html index.htm;
        
        location / {
            try_files $uri $uri/ /index.php?q=$uri&$args;
        }

        error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        location ~ \.php$ {
            try_files $uri =404;
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_pass   unix:/var/run/php5-fpm.sock;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include        fastcgi_params;
        }

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }
}
