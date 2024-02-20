# Deploying Laravel on Linode, Vultr, DigitalOcean etc with LEMP Stack

Using any cloud hosting provider—Linode, Vultr, DigitalOcean, etc.—this cheatsheet offers detailed instructions on how to deploy and configure a Laravel application on an Ubuntu OS running an LEMP stack.

## Prerequisite
- [x] Any cloud hosting account that you can create Compute VPS Instance
- [ ] Amazon Webservice account with a created EC2 VPS instance (Ubuntu 20 or later is much recommended).
- [ ] Linode account with a created virtual server (Ubuntu 20 or later is much recommended).
- [ ] DigitalOcean account with a created VPS instance (Ubuntu 20 or later is much recommended).
- [x] Knowledge on Basic server administration.
- [x] Git installed on your local machine.
- [x] A Laravel application hosted on GitHub (e.g., "laravel_yourapp").

## Note:
   - Example we have an application called "laravel_yourapp". Change the "laravel_yourapp" for your website/domain name or Laravel app name. 

## Initial Server Setup

1. **Create a VPS instance:**
   - Choose Ubuntu 20.04 as the operating system.
   - Follow the  wizard to create the VPS instance.

2. **Access Your VPS instance:**
   - Open your terminal and use SSH to access your VPS instance:
     ```bash
     ssh root@your_VPS instance_ip
     ```

3. **Update Server Packages:**
   ```bash
   sudo apt update
   sudo apt upgrade -y # optional
   
4. **Install NGINX Web Server**
    ```bash
    sudo apt install nginx
    
    # Start NGINX and Enable it.
    sudo systemctl start nginx
    sudo systemctl enable nginx

    # Check if NGINX status is running
    sudo systemctl status nginx
   
5. **Install MySQL:**
    ```bash
    sudo apt install mysql-server
    sudo mysql_secure_installation # modify some insecure defaults

6. **Change root mysql password:**
    ```bash
    sudo mysql
    ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'your_password';
    mysql -u root -p # Access mysql
    
7. **Install PHP and Required Extensions:**
   - To Download Other Versions of PHP
     ```bash
     sudo add-apt-repository ppa:ondrej/php
     ```
   - Latest Version
    ```bash
    sudo apt install php php-fpm php-mysql php-common php-bcmath php-ctype php-json php-mbstring php-openssl php-pdo php-tokenizer php-xml php-zip php-gd
    ```
    - If you want specific version of PHP like 7.4
    ```bash
    sudo apt install php7.4 php7.4-fpm php7.4-mysql php7.4-common php7.4-bcmath php7.4-ctype php7.4-json php7.4-mbstring php7.4-openssl php7.4-pdo php7.4-tokenizer php7.4-xml php7.4-zip php7.4-gd
    ```
    
9. **Configure Nginx for Laravel:**
    - Create an Nginx server block configuration for your Laravel app (e.g., /etc/nginx/sites-available/laravel_yourapp).
    - Configure the Nginx server block to use PHP-FPM for processing PHP files.
    
10. **Create server block config**
    ```bash
    sudo nano /etc/nginx/sites-available/laravel_yourapp
    ```
    
    ```nginx
    # NGINX CONFIG FOR LARAVEL [https://laravel.com/docs/7.x/deployment]
    server {
       listen 80;
       server_name your_VPS instance_ip;
       root /var/www/laravel_yourapp/public;
    
       add_header X-Frame-Options "SAMEORIGIN";
       add_header X-XSS-Protection "1; mode=block";
       add_header X-Content-Type-Options "nosniff";
    
       index index.php;
    
       charset utf-8;
    
       location / {
           try_files $uri $uri/ /index.php?$query_string;
       }
    
       location = /favicon.ico { access_log off; log_not_found off; }
       location = /robots.txt  { access_log off; log_not_found off; }
    
       error_page 404 /index.php;
    
       location ~ \.php$ {
           fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
           fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
           include fastcgi_params;
       }
    
       location ~ /\.(?!well-known).* {
           deny all;
       }
    }
    ```

    ```nginx
    # From Digital Ocean Snippets [https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-ubuntu-18-04]
    
    server {
        listen 80;
        root /var/www/html;
        index index.php index.html index.htm index.nginx-debian.html;
        server_name your_domain;

        location / {
                try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
        }

        location ~ /\.ht {
                deny all;
        }
      }
    ```
    
10. **Enable the Nginx Configuration:**
    - Create a symbolic link to enable the site:
    ```bash
    sudo ln -s /etc/nginx/sites-available/laravel_yourapp /etc/nginx/sites-enabled/
    ```

    - Unlink the default conf
    ```bash
    sudo unlink /etc/nginx/sites-enabled/default


11. **Test Nginx Configuration:**
    ```bash
    nginx -t

## Deploy Laravel Application

1. **Clone Your Laravel App from GitHub:**
    ```bash
    git clone https://github.com/yourusername/laravel_yourapp.git /var/www/laravel_yourapp

2. **Install Composer**    
    - https://getcomposer.org/download/

3. **Install Composer Dependencies**
    ```bash
    cd /var/www/laravel_yourapp
    composer install --no-interaction --prefer-dist

4. **Generate Laravel Application Key:**
    ```bash
    php artisan key:generate

5. **Set Permissions:**
    - Ensure proper file permissions for Laravel Application
    ```bash
    sudo chown -R www-data:www-data /var/www/laravel_yourapp
    sudo chmod -R 755 /var/www/laravel_yourapp/storage


6. **Access MySQL:**
    ```bash
    mysql -u root -p

7. **Create a database and user:**
    ```bash
    CREATE DATABASE laravel_yourapp_db;
    CREATE USER 'laravel_user'@'localhost' IDENTIFIED BY 'your_password';
    GRANT ALL PRIVILEGES ON laravel_yourapp_db.* TO 'laravel_user'@'localhost';
    FLUSH PRIVILEGES;

8. **EXIT MYSQL**
    ```bash
    exit

9. **Configure .env**
    - Update the Database Configuration
    ```bash
    cd /var/www/laravel_yourapp
    nano .env

    
10. **Migrate and Seed Database (Laravel):**
    ```bash
    php artisan migrate --seed

## Final Steps
1. **Reload NGINX**
    ```bash
    systemctl reload nginx

2. **Restart PHP-FPM**
    ```bash
    systemctl restart php7.4-fpm
