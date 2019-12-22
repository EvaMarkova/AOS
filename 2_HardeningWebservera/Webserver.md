# Hardening webového servera

Webový server nainštalujeme na Debiane 10 s ip adresou 192.168.0.1. Cieľom je rozbehať funkčný Wordpress.
Najprv nainštalujeme apache2. 

```
apt install apache2
```

Keďže sme v prvom zadaní konfigurovali ufw (firewall), skontrolujeme či máme povolené HTTP a HTTPS.

```
ufw app info "WWW Full"
```

Ak nie, povolíme prichádzajúce spojenia HTTP a HTTPS: 

```
ufw allow in "WWW Full"
```

Nainštalujeme databázu MariaDB: 

```
apt install mariadb-server
```

Zmeníme niektoré z menej bezpečných predvolených možností. Zablokujeme vzdialené rootovské prihlásenia a odstránime nepoužitých používateľov databázy.

```
mysql_secure_installation
```

Nainštalujeme php a reštartujeme apache2:

```
apt install php-curl php-gd php-intl php-mbstring php-soap php-xml php-xmlrpc php-zip
systemctl restart apache2
```

Stiahneme a nainštalujeme wordpress:

```
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
```

Potom vytvoríme konfiguračný súbor pre wordpress: 

```
cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php
```

Skopírujeme všetky súbory do priečinku domenask a zmeníme oprávnenia, aby webový server mohol používať dané súbory a priečinky:

```
cp -a /tmp/wordpress/. /var/www/domenask
chown -R www-data:www-data /var/www/domenask
```

Vytvoríme konfiguračný súbor pre našu stránku:

```
nano /etc/apache2/sites-available/domena.sk.conf
```

Vložíme doň:

```
<VirtualHost *:80>        
        ServerAdmin webmaster@localhost
        ServerName domena.sk
        DocumentRoot /var/www/domenask
        <Directory />
           Options FollowSymLinks
           AllowOverride None
        </Directory>
        <Directory /var/www/domenask/>
           Options Indexes FollowSymLinks MultiViews
           AllowOverride None
           Order allow,deny
           allow from all
         </Directory> 
         ErrorLog ${APACHE_LOG_DIR}/error.log 
         LogLevel warn
         CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Povolíme virtuálneho hosta a reštartujeme apache2: 

```
a2ensite domena.sk
systemctl restart apache2
```

Ešte je potrebné otvoriť súbor `/etc/hosts` a pridať doň `192.168.0.1	domena.sk`
