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
        listen 80;
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
1. Auto-start NGINX on system start: `sudo chkconfig nginx on`

## Apache
1. Install: `sudo yum install httpd24 -y`
1. Configure `/etc/httpd/conf/httpd.conf` as follows:
    ```httpd.conf
    NameVirtualHost 127.0.0.1:8080  # Only accessible from localhost
    Listen 8080
    ```
1. Configure virtual hosts at `/etc/httpd/conf.d/vhosts.conf`:
    ```
    <VirtualHost 127.0.0.1:8080>
        ServerAdmin email@sub1.domain.com
        DocumentRoot /var/www/sub1/public/
        ServerName sub1.domain.com
        ErrorLog logs/sub1.domain.com-error_log
        CustomLog logs/sub1.domain.com-access_log common
    </VirtualHost>
    ```
1. Create new directory for virtual host: `sudo mkdir /var/www/sub1`
1. Give ownership to current logged in user: `sudo chown $USER:$USER -R /var/www/sub1/`
1. Give write permission: `sudo chmod -R 755 /var/www`
1. Auto-start Apache on system start: `sudo chkconfig httpd on`

## SSL (Let's Encrypt)
1. Install Certbot:
    ```
    sudo yum install python27-devel git -y
    sudo git clone https://github.com/letsencrypt/letsencrypt /opt/letsencrypt
    sudo /opt/letsencrypt/letsencrypt-auto --debug
    ```
1. Request cert:
    ```
    sudo /opt/letsencrypt/letsencrypt-auto --authenticator standalone --installer nginx --pre-hook "nginx -s stop" --post-hook "nginx"
    ```
    - Go through the wizard carefully.
    - To renew: 
        ```
        sudo /opt/letsencrypt/letsencrypt-auto --authenticator standalone --installer nginx --pre-hook "nginx -s stop" --post-hook "nginx"
        ```
1. If all went well, your certs will be at `/etc/letsencrypt/live/sub1.domain.com/` and your `/etc/nginx/conf.d/default.conf` has been updated by cerbot automatically.

## MariaDB
1. Add yum repository. Create this file: `/etc/yum.repos.d/maria.repo`:
    ```
    [mariadb]
    name = MariaDB
    baseurl = http://yum.mariadb.org/10.1/centos6-amd64
    gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
    gpgcheck=1
    ```
    - Refer https://mariadb.com/kb/en/library/yum/
1. Install: 
    ```
    sudo yum makecache
    sudo yum install MariaDB-server MariaDB-client -y
    ```
1. Secure your MariaDB installation: `sudo mysql_secure_installation`
1. Auto-start MariaDB on system start: `sudo chkconfig mysql on`

## PHP 7.1
1. Install: `sudo yum install php71 -y`
1. Install PHP Modules
    - Run `yum search php71-` to search for available modules and just yum install it.
    
## Start All Services
1. `sudo service nginx start`
1. `sudo service httpd start`
1. `sudo service mysql start`

## References:

- https://gist.github.com/nrollr/56e933e6040820aae84f82621be16670
- https://www.digitalocean.com/community/tutorials/how-to-configure-nginx-as-a-reverse-proxy-for-apache
- https://stackoverflow.com/questions/14434120/nginx-set-multiple-server-name-with-ssl-support
- https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-centos-7
- https://coderwall.com/p/e7gzbq/https-with-certbot-for-nginx-on-amazon-linux
