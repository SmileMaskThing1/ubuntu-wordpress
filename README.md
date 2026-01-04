Ubuntu setup:




------------------------------------------------------------
------------------------------------------------------------
------------------------------------------------------------
Wordpress setup:


# --- Update system ---
sudo apt update && sudo apt upgrade -y

# --- Install Apache, PHP, MySQL ---
sudo apt install apache2 mysql-server php libapache2-mod-php php-mysql wget -y

# --- Download and extract WordPress ---
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -xvzf latest.tar.gz

# Move WordPress into Apache web root
sudo mv wordpress /var/www/html/

# Set permissions
sudo chown -R www-data:www-data /var/www/html/wordpress
sudo chmod -R 755 /var/www/html/wordpress

# --- Configure MySQL database ---
sudo mysql -u root -p <<EOF
CREATE DATABASE wordpress;
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
EOF

# --- Create SSL directory and generate self-signed certificate ---
sudo apt install openssl -y
sudo mkdir -p /etc/apache2/ssl
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/apache2/ssl/apache.key \
  -out /etc/apache2/ssl/apache.crt

# --- Edit Apache SSL config (default-ssl.conf) ---
# Insert inside <VirtualHost _default_:443>:
# SSLEngine on
# SSLCertificateFile /etc/apache2/ssl/apache.crt
# SSLCertificateKeyFile /etc/apache2/ssl/apache.key
sudo nano /etc/apache2/sites-available/default-ssl.conf

# --- Enable SSL in Apache ---
sudo a2enmod ssl
sudo a2ensite default-ssl
sudo systemctl reload apache2

# --- Test access ---
# HTTP:
#   http://<your_VM_IP>/wordpress
# HTTPS:
#   https://<your_VM_IP>/wordpress
