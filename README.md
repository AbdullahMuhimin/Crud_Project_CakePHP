🍰 CakePHP Deployment on Oracle Cloud VM – Beginner-Friendly Guide
This guide helps beginners deploy a CakePHP CRUD project with MySQL and Apache on an Oracle Cloud VM. Follow each step carefully to avoid errors.
🧾 Prerequisites

- A working CakePHP project (e.g., Crud_Project/).
- Oracle Cloud VM with public IP (e.g., Your_VM_Public_IP).
- SSH access tool like MobaXterm.
- Basic understanding of file paths and terminal commands.

🪜 Step-by-Step Deployment
✅ Step 1: Prepare Local CakePHP Project

Make sure your CakePHP project runs locally:
URL: http://localhost/Crud_Project/

In config/app_local.php:
  'username' => 'root', 
  'password' => '', 
  'database' => 'crud_project' 

✅ Step 2: Log in to the Oracle VM
Use MobaXterm terminal:
ssh opc@Your_VM_Public_IP
 Once logged in, switch to root user:
sudo su
✅ Step 3: Install Apache, PHP 8.2+, and MySQL
# Update system
dnf update -y
# Install Apache
dnf install httpd -y
systemctl enable --now httpd
# Install Remi repo to get PHP 8.2
dnf install -y https://rpms.remirepo.net/enterprise/remi-release-8.rpm 
dnf module reset php -y 
dnf module enable php:remi-8.2 -y 
# Install PHP 8.2 and required extensions
dnf install -y php php-mysqlnd php-mbstring php-intl php-xml php-bcmath php-pdo php-gd php-cli php-soap php-json php-curl
# Install MariaDB (MySQL)
dnf install mariadb-server -y
systemctl enable --now mariadb

✅ Step 4: Upload CakePHP Project
From Windows CMD:
scp -r C:/xampp/htdocs/Crud_Project opc@Your_VM_Public_IP:/var/www/html/
Set permissions:
chown -R apache:apache /var/www/html/Crud_Project
chmod -R 755 /var/www/html/Crud_Project
chmod -R 777 /var/www/html/Crud_Project/tmp
chmod -R 777 /var/www/html/Crud_Project/logs

Set permissions in VM : 
Back in the VM terminal:
cd /var/www/html
chown -R apache:apache Crud_Project
chmod -R 755 Crud_Project
chmod -R 777 Crud_Project/tmp
chmod -R 777 Crud_Project/logs


✅ Step 5: Configure Apache for CakePHP
Create the Apache config file:
nano /etc/httpd/conf.d/crud_project.conf
Paste the following:

<VirtualHost *:80>
    ServerName Your_VM_Public_IP
    DocumentRoot /var/www/html/Crud_Project/webroot

    <Directory /var/www/html/Crud_Project/webroot>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog /var/log/httpd/crud_error.log
    CustomLog /var/log/httpd/crud_access.log combined
</VirtualHost>

Restart Apache:
systemctl restart httpd
✅ Step 6: Disable Default Apache Page
mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf.bak
systemctl restart httpd
✅ Step 7: Allow HTTP in OCI

In OCI Console > Networking > VCN > Subnet > Security Lists:
- Add Ingress Rule:
  - Source CIDR: 0.0.0.0/0
  - Protocol: TCP
  - Port: 80

✅ Step 8: Configure MySQL database and user
Login to MySQL:
mysql -u root
Inside MySQL shell:

CREATE DATABASE crud_project;
CREATE USER 'crud_user'@'localhost' IDENTIFIED BY 'crud_password';
GRANT ALL PRIVILEGES ON crud_project.* TO 'crud_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;

✅ Step 9: Update CakePHP DB Config

Edit the app_local.php file:
nano /var/www/html/Crud_Project/config/app_local.php

Edit config/app_local.php:

'Datasources' => [
    'default' => [
        'host' => 'localhost',
        'username' => 'crud_user',
        'password' => 'crud_password',
        'database' => 'crud_project',
        ...
    ],
]
Press CTRL +X, Then Y, Then Enter to save. 

✅ Step 10: Create or Import Tables
Option A: Manually create 'posts' table:
Log in to MySQL on your VM:
mysql -u root -p
Then switch to your database:
USE crud_project;
Now, create the posts table (example schema):

CREATE TABLE posts (
  id INT(11) NOT NULL AUTO_INCREMENT,
  title VARCHAR(255) NOT NULL,
  body TEXT,
  created DATETIME,
  modified DATETIME,
  PRIMARY KEY (id)
);
Exit MySQL:
EXIT;


Option B: Export and import full DB:
Export from local (CMD):
"C:\xampp\mysql\bin\mysqldump.exe" -u root -p crud_project > crud_project.sql
Upload to VM:
scp C:/Users/a2i/crud_project.sql opc@Your_VM_Public_IP:/home/opc/
Import into MySQL on VM:
mysql -u root -p crud_project < /home/opc/crud_project.sql
✅ Step 11: Access Application
Visit in browser: http://Your_VM_Public_IP/
✅ Step 12: View Data via MySQL

mysql -u root -p
USE crud_project;
SELECT * FROM posts;

🎉 Done!
Your CakePHP app is now running on Oracle Cloud VM with database support.
