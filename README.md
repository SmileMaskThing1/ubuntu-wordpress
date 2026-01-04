Ubuntu setup:

. during install, language should be English.
. Keep keyboard layout the same.
> Leave Ubuntu server checked.
> Keep IPv4 static. Just press Done.
> No need for proxy, just skip this one.
> Once repositories are listed, press Done.
> (Root) Custom Storage Layout -> Free Space -> Add GPT Partition -> 25G; Mount: / -> Create.
> (Home) Free Space -> Add GPT Partition -> 7G; Mount: /home -> Create.
> (Physical) Add Partition -> 6.9G; Format: Swap -> Create.
> Once all three partitions are made, tab and press Done; then Continue.
> Name: Giorgi -> ServerName: giorgiserver1 -> username: student -> password: 1234
> Do not activate Ubuntu Pro, just continue.
> check Install OpenSSH Server before pressing Done.
> Skip services, just press Done; Then wait for a good while for it to install before it prompts you to continue.
> When Reboot Now is listed, just press it; Once it prompts you to "remove installation medium", just press ENTER, as this is just a virtual machine.

>> login: student -> password: 1234 (as we have set up)

>> --- Commands to eventually set up desktop experience ---
>> sudo -s (password 1234) (takes us to /home)
>> apt update
>> apt upgrade -y
>> ifconfig -> apt install net-tools -y
>> ifconfig (checking inet)
>> apt install mc -y
>> mc (why not)

>> --- Set up ubuntu desktop experience --- 
>> apt install ubuntu-desktop -y
>> once installed, it will reboot. password is 1234.
>> First two phases done. Now move on to setting up wordpress.

------------------------------------------------------------
------------------------------------------------------------
------------------------------------------------------------
Wordpress setup:


>> --- Update system ---
sudo apt update && sudo apt upgrade -y

>> --- Install Apache, PHP, MySQL ---
sudo apt install apache2 mysql-server php libapache2-mod-php php-mysql wget -y

>> --- Download and extract WordPress ---
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -xvzf latest.tar.gz

>> Move WordPress into Apache web root
sudo mv wordpress /var/www/html/

>> Set permissions
sudo chown -R www-data:www-data /var/www/html/wordpress
sudo chmod -R 755 /var/www/html/wordpress

>> --- Configure MySQL database ---
sudo mysql -u root -p <<EOF
CREATE DATABASE wordpress;
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
EOF

>> --- Create SSL directory and generate self-signed certificate ---
sudo apt install openssl -y
sudo mkdir -p /etc/apache2/ssl
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/apache2/ssl/apache.key \
  -out /etc/apache2/ssl/apache.crt

>> --- Edit Apache SSL config (default-ssl.conf) ---
# Insert inside <VirtualHost _default_:443>:
# SSLEngine on
# SSLCertificateFile /etc/apache2/ssl/apache.crt
# SSLCertificateKeyFile /etc/apache2/ssl/apache.key
sudo nano /etc/apache2/sites-available/default-ssl.conf

>> --- Enable SSL in Apache ---
sudo a2enmod ssl
sudo a2ensite default-ssl
sudo systemctl reload apache2

>> --- Test access ---
>> HTTP:
>>   http://<your_VM_IP>/wordpress
>> HTTPS:
>>   https://<your_VM_IP>/wordpress
