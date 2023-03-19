#Instalacion en ubuntu server
```
sudo nano /etc/cloud/cloud.cfg
```
Cambiar alli a
```
users:
  - default
  - root

disable_root: false
ssh_pwauth:   true
```

despues editar
```
nano /etc/ssh/sshd_config
```

y colocar
```
Port 6813 
PermitRootLogin yes
PasswordAuthentication yes
```

Para el password del root
```
sudo su
passwd root
# 84River@B
```
En los permisos de firewall
```
sudo ufw allow ssh
sudo ufw allow 6813/tcp
```

Reiniciamos servicios
```
sudo service ssh restart
sudo systemctl restart ssh
```

Abrir el puerto 6813 en la admin del servidor
```
#ssh -p 6813 root@0.0.0.0
cd .ssh
nano authorized_keys
```

Eliminar todo antes de ssh-rsa 
```
# Si se cuelga apt, buscar el pid de root
#ps aux | grep -i apt
#kill PID
#Salir  y volver a entrar
```

# Instalacion
```
apt update
```
```
apt -y install apache2
ufw allow Apache
systemctl enable --now apache2
```

```
apt -y install php libapache2-mod-php php-{gd,common,mail,mail-mime,mysql,pear,db,mbstring,xml,curl}
```

```
apt -y install mariadb-server php-pear
```

```
sudo pear install DB
sudo pear install MDB2
pear channel-update pear.php.net
```

```
mysql_secure_installation
# Enter
# Disallow root login remotely? [Y/n] n
# password usada aqui es 84Uniq@
```

```
apt -y install freeradius freeradius-mysql freeradius-utils
systemctl enable --now freeradius.service 
```

```
sudo ufw allow to any port 1812 proto udp
sudo ufw allow to any port 1813 proto udp
```

```
mysql -u root -p
CREATE DATABASE radius;
GRANT ALL ON radius.* TO radius@localhost IDENTIFIED BY "84Uniq@";
FLUSH PRIVILEGES;
quit;
```

```
mysql -u root -p radius < /etc/freeradius/3.0/mods-config/sql/main/mysql/schema.sql
sudo ln -s /etc/freeradius/3.0/mods-available/sql /etc/freeradius/3.0/mods-enabled/
```

```
apt install git
git clone https://github.com/wirisp/daloup.git
```


Remplazamos el archivo sql , cambiale la contraseña usada password= 84Uniq@ por la tuya
```
\mv /root/daloup/ubuntu/sql /etc/freeradius/3.0/mods-available/sql
nano /etc/freeradius/*/mods-enabled/sql
```

```
sudo chgrp -h freerad /etc/freeradius/3.0/mods-available/sql
sudo chown -R freerad:freerad /etc/freeradius/3.0/mods-enabled/sql
sudo systemctl restart freeradius.service
```

```
git clone https://github.com/wirisp/daloradius.git
cd daloradius
mysql -u root -p radius < contrib/db/fr2-mysql-daloradius-and-freeradius.sql
mysql -u root -p radius < contrib/db/mysql-daloradius.sql
```

```
cd ..
mv daloradius /var/www/html/
\mv /root/daloup/ubuntu/daloradius.conf.php /var/www/html/daloradius/library/daloradius.conf.php
#Cambiar contrase;a
nano /var/www/html/daloradius/library/daloradius.conf.php
```

```
sudo chown -R www-data:www-data /var/www/html/daloradius/
sudo chmod 664 /var/www/html/daloradius/library/daloradius.conf.php
```

```
sudo systemctl restart freeradius.service apache2
```

```
timedatectl set-timezone America/Mexico_City
```

```
apt install php-dba
```

```
touch /var/log/freeradius/radius.log
touch /var/log/messages
touch /tmp/daloradius.log
```

```
chmod 755 /var/log/freeradius
chmod 644 /var/log/freeradius/radius.log
chmod 644 /var/log/messages
touch /tmp/daloradius.log
```

```
#cambio directorio 
\mv /root/daloup/ubuntu/ubunturadiusd.conf /etc/freeradius/3.0/radiusd.conf
\mv /root/daloup/ubuntu/default /etc/freeradius/3.0/sites-enabled/default
\mv /root/daloup/ubuntu/sqlcounter /etc/freeradius/3.0/mods-available/sqlcounter
\mv /root/daloup/ubuntu/access_period.conf /etc/freeradius/3.0/mods-config/sql/counter/mysql/access_period.conf
\mv /root/daloup/ubuntu/quotalimit.conf /etc/freeradius/3.0/mods-config/sql/counter/mysql/quotalimit.conf
\mv /root/daloup/ubuntu/radutmp /etc/freeradius/3.0/mods-enabled/radutmp
\mv /root/daloup/ubuntu/queries.conf /etc/freeradius/3.0/mods-config/sql/main/mysql/queries.conf
```

```
\mv /root/daloup/ubuntu/index.php /var/www/html/index.php
```

```
cd daloup
mysql -p -u root radius < /root/daloup/ubuntu/base.sql
```

sudo nano -l /var/www/html/daloradius/library/exten-radius_server_info.php

from:

<td class='summaryKey'> MySQL </td> <td class='summaryValue'><span class='sleft'><?php echo check_service("mysql"); ?></span> </td>

to:

<td class='summaryKey'> MariaDB </td> <td class='summaryValue'><span class='sleft'><?php echo check_service("mariadb"); ?></span> </td>
