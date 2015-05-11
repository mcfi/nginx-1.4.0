# Build nginx-1.4.0 and host wordpress.

1. You need to compile MCFI/PICFI toolchain! Download it here at https://github.com/mcfi/MCFI and follow the instructions.

2. Execute nginx.sh in this directory to build and install nginx.

3. Install mysql, php5-mysql and php5-fpm using the following command

   ```sudo apt-get install mysql-server php5-mysql php5-fpm php5-gd libssh2-php```

4. In config file (```/etc/php5/fpm/pool.d/www.conf```) for php5-fpm,
   change "user" to the user name of nginx's executable.
   
   Uncomment ```listen.allowed_clients = 127.0.0.1``` to allow local connections.
   
   Make php5-fpm connectable from non-root users

   ```su && chmod o+rw php5-fpm.sock```

5. Create a MySQL database
   
   Type ```mysql -u root -p``` in bash and execute the following commands.
   
   ```CREATE DATABASE wordpress;```
   
   ```CREATE USER wordpressuser@localhost IDENTIFIED BY 'password';```
   
   ```GRANT ALL PRIVILEGES ON wordpress.* TO wordpressuser@localhost;```
   
   ```FLUSH PRIVILEGES;```

6. Download wordpress and decompress it to $WORDPRESS.
   
   ```cd $WORDPRESS``
   
   ``cp wp-config-sample.php wp-config.php``

   Open wp-config.php and modify the following configuration as below:
   
   ``define('DB_NAME', 'wordpress');```
   
   ```define('DB_USER', 'wordpressuser');```
   
   ```define('DB_PASSWORD', 'password');```

   Copy wordpress files to nginx installation dir's html subdir
   
   ```rsync -avP $WORDPRESS/ $NGINX/html/```
   
   ```mkdir $NGINX/html/wp-content/uploads```

7. Use the following configuration file for nginx. Start nginx and access
   the front page through ```http://localhost/wp-admin/install.php``` using a browser.

```
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
```
