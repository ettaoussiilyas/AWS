
If you want to install your Laravel project inside `/home/ubuntu`, you need to configure **Apache** to serve your Laravel application from there. Below is a step-by-step guide to achieve this:

---

## **Step 2: Update System Packages**
```sh
sudo apt update && sudo apt upgrade -y
```
```sh
sudo apt install fish
```
---

## **Step 3: Install Apache, PHP 8.3, and Required Extensions**
```sh
sudo apt install apache2 -y
```
```sh
sudo add-apt-repository ppa:ondrej/php -y
```
```sh
sudo apt update
```
```sh
sudo apt install php8.3 php8.3-cli php8.3-common php8.3-mbstring php8.3-xml php8.3-bcmath php8.3-curl php8.3-zip php8.3-mysql unzip -y
```

Verify PHP installation:
```sh
php -v
```

---

## **Step 4: Install Composer**
```sh
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
composer -V
```

---

## **Step 5: Install Node.js and NPM (Latest)**
```sh
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install -y nodejs
node -v
npm -v
```

---

## **Step 6: Install MySQL Server**
```sh
sudo apt install mysql-server -y
sudo systemctl start mysql
sudo systemctl enable mysql
```

Secure MySQL:
```sh
sudo mysql_secure_installation
```

Create a Laravel database:
```sh
sudo mysql -u root
```
```sql
CREATE DATABASE laravel_db;
```

---

## **Step 7: Install Laravel in `/home/ubuntu`**
Navigate to the `home/ubuntu` directory:
```sh
cd /home/ubuntu
```


Clone your Laravel project (or create a new one):
```sh
mkdir projects && cd projects
composer create-project laravel/laravel:11 myapp && cd myapp
```

Install Laravel dependencies:
```sh
composer install --no-dev --optimize-autoloader
npm install && npm run build
```

Set the correct permissions:
```sh
sudo chown -R www-data:www-data /home/ubuntu/projects
```

```sh
sudo chmod -R 775 /home/ubuntu/projects/myapp/storage /home/ubuntu/projects/myapp/bootstrap/cache
```

```sh
sudo chown o+x /home/ubuntu/projects
sudo chown o+x /home/ubuntu
```

---

## **Step 8: Configure Apache**
### **1. Disable Default Apache Virtual Host**
```sh
sudo a2dissite 000-default.conf
```

### **2. Create a New Virtual Host**
```sh
sudo nano /etc/apache2/sites-available/laravel.conf
```

Add the following configuration:
```apache
<VirtualHost *:80>
    ServerName ip dylk
    DocumentRoot /home/ubuntu/projects/myapp/public

    <Directory /home/ubuntu/projects/myapp/public>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>   
```

### **3. Enable the New Virtual Host**
```sh
sudo a2ensite laravel.conf
sudo a2enmod rewrite
sudo systemctl restart apache2
```

---

## **Step 9: Configure Laravel**
Copy `.env` file:
```sh
cp .env.example .env
```

Update `.env` with the database credentials:
```sh
nano .env
```
Modify:
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_db
DB_USERNAME=laravel_user
DB_PASSWORD=your_password
```

Generate application key:
```sh
php artisan key:generate
```

Run migrations:
```sh
php artisan migrate --seed
```

Clear and cache configurations:
```sh
php artisan config:clear
php artisan cache:clear
php artisan config:cache
```

Restart Apache:
```sh
sudo systemctl restart apache2
```

---

## **Step 10: Set Up Supervisor for Queue (Optional)**
```sh
sudo apt install supervisor -y
sudo nano /etc/supervisor/conf.d/laravel-worker.conf
```
Add:
```
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /home/ubuntu/laravel-app/artisan queue:work --tries=3
autostart=true
autorestart=true
user=www-data
numprocs=1
redirect_stderr=true
stdout_logfile=/home/ubuntu/laravel-app/storage/logs/worker.log
```

Restart Supervisor:
```sh
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start laravel-worker:*
```
