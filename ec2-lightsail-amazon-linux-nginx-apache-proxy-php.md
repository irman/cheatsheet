# [WIP]

# Setting up Nginx-Apache Reverse Proxy, PHP, & MySQL on EC2/Lightsail with Amazon Linux

Final setup should consists of:

1. NGINX (reverse proxy & static contents)
    - Supports SSL ([Let's Encrypt](https://letsencrypt.org)).
    - Supports multiple domains, 1 IP.
1. Apache (Dynamic content: PHP)
1. PHP 7
1. MariaDB

You can skip certain parts if you don't need it.

## NGINX
1. Install: `sudo yum install nginx -y`
1. Configure this file: `/etc/nginx/conf.d/default.conf` to something like this:
    ```nginx.conf
    server {
        listen 443;
        server_name sub1.domain.com;
        root /var/www/;
        index index.php index.html index.htm;
        
        location / {
            try_files $uri $uri/ /index.php;
        }

        location ~ \.php$ {
            proxy_set_header X-Real-IP  $remote_addr;
            proxy_set_header X-Forwarded-For $remote_addr;
            proxy_set_header Host $host;
            proxy_pass http://127.0.0.1:8080;
         }

        location ~ /\.ht {
            deny all;
        }

        #ssl on;
        #ssl_certificate /var/www/ssl/samecertif.crt;
        #ssl_certificate_key /var/www/ssl/samecertif.key;
    }
    ```

## Apache
1. Install: `sudo yum install httpd -y`
2. Configure `/etc/httpd/conf/httpd.conf` as follows:
    ```httpd.conf
    NameVirtualHost 127.0.0.1:8080  # Only accessible from localhost
    Listen 8080
    
    <VirtualHost 127.0.0.1:8080>
        ServerAdmin email@sub1.domain.com
        DocumentRoot /var/www/sub1/public/
        ServerName sub1.domain.com
        ErrorLog logs/sub1.domain.com-error_log
        CustomLog logs/sub1.domain.com-access_log common
    </VirtualHost>
    ```

## References:

- https://gist.github.com/nrollr/56e933e6040820aae84f82621be16670
- https://www.digitalocean.com/community/tutorials/how-to-configure-nginx-as-a-reverse-proxy-for-apache
- https://stackoverflow.com/questions/14434120/nginx-set-multiple-server-name-with-ssl-support
- https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-centos-7
