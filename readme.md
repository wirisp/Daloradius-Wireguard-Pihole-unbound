# Daloradius+wireguard+pihole+unbound en  debian 11
Se realiza la instalacion de Daloradius ,en conjunto con Pihole y unbound :shipit:

- [x] Daloradius
- [x] Wireguard
- [x] Pihole
- [x] Unbound

## Clave ssh, cambio de puerto e ingreso por clave
- Actualizamos el sistema
```
apt -y update && apt -y upgrade
```
- Editamos el archivo
```
nano /etc/ssh/sshd_config
```
- Dentro del archivo anterior colocamos el puerto y checamos que se encuentre sin almoadillas (#) lo demas
```
Port 6813 
PermitRootLogin yes
PasswordAuthentication yes
```
- Reinicamos el servicio con
```
systemctl restart sshd
```
_A partir de ahora el puerto ya no sera 22 si no el 6813_
- Creamos una carpeta para la clave ssh y la editamos
```
mkdir .ssh && cd .ssh
nano authorized_keys
```
_Dentro de ella colocamos nuestra ssh, si no usaras ssh simplemente cierralo y pasa al siguiente paso_
- Le colocamos permisos al archivo y a la carpeta
chmod 600 ~/.ssh/authorized_keys && chmod 700 ~/.ssh/
# Instalacion de freeradius
- Instalamos algunos paquetes necesarios
```
apt -y install software-properties-common gnupg2 dirmngr
```
- Agregamos claves y repositorios de Mariadb
```
apt-key adv --fetch-keys 'https://mariadb.org/mariadb_release_signing_key.asc'
```
```
add-apt-repository 'deb [arch=amd64,arm64,ppc64el] https://mirror.rackspace.com/mariadb/repo/10.5/debian bullseye main'
```
- Actualizamos el sistema o repositorio
```
apt -y update
```
- Instalamos los paquetes necesarios de mariadb
```
apt install mariadb-server mysqltuner -y
```
- Iniciamos el servicio de mariadb
```
systemctl start mysql.service
```
- Preparamos la base de datos
```
mysql_secure_installation
# enter
# Disallow root login remotely? [Y/n] n
# password usada aqui es 84Uniq@
```
_Si nos pregunta desactivar conexiones remotas colocar n_
- Creamos una base de datos
```
mysql -u root -p
CREATE DATABASE radius;
GRANT ALL ON radius.* TO radius@localhost IDENTIFIED BY "84Uniq@";
FLUSH PRIVILEGES;
quit;
```
- Instalacion de apache server
```
apt -y install apache2
```
```
apt -y install php libapache2-mod-php php-{gd,common,mail,mail-mime,mysql,pear,mbstring,xml,curl}
```
- Instalamos freeradius
```
apt -y install freeradius freeradius-mysql freeradius-utils
```
```
systemctl enable --now freeradius.service
```
- Enviamos el schema de la base de datos
```
mysql -u root -p radius < /etc/freeradius/3.0/mods-config/sql/main/mysql/schema.sql
```
- Instalamos git y clonamos un repositorio
```
apt install -y git
```
```
git clone https://github.com/wirisp/Daloradius-Wireguard-Pihole-unbound.git dapiun
```
- Movemos ciertos archivos, podemos ir checando su conetenido por si debemos cambiar algo en ellos
```
\mv /root/dapiun/sql /etc/freeradius/3.0/mods-available/sql
nano /etc/freeradius/3.0/mods-available/sql
```

```
\mv /root/dapiun/eap /etc/freeradius/3.0/mods-available/eap
```

```
ln -s /etc/freeradius/3.0/mods-available/sql /etc/freeradius/3.0/mods-enabled/
nano /etc/freeradius/3.0/mods-enabled/sql
```

```
chgrp -h freerad /etc/freeradius/3.0/mods-available/sql
chown -R freerad:freerad /etc/freeradius/3.0/mods-enabled/sql
```
- Reiniciamos freeradius
```
systemctl restart freeradius
```
## Instalacion de Daloradius
- Instalamos zip y unzip
```
apt -y install wget zip unzip
```
```
git clone https://github.com/wirisp/daloradius-almalinux8.git daloradius
cd daloradius
```
```
mysql -u root -p radius < contrib/db/fr2-mysql-daloradius-and-freeradius.sql
mysql -u root -p radius < contrib/db/mysql-daloradius.sql
```
- Regresammos a la carpeta con
```
cd ..
```
- Movemos la carpeta daloradius a html del servidor
```
\mv daloradius /var/www/html/
```
- Movemos algunos archivos del github clonado
```
\mv /root/dapiun/daloradius.conf.php /var/www/html/daloradius/library/daloradius.conf.php
nano /var/www/html/daloradius/library/daloradius.conf.php
```
```
chown -R www-data:www-data /var/www/html/daloradius/
chmod 664 /var/www/html/daloradius/library/daloradius.conf.php
```
```
systemctl restart freeradius.service apache2
```
```
timedatectl set-timezone America/Mexico_City
```
- Instalamos pear
```
pear install DB
pear install MDB2
pear channel-update pear.php.net
```
- Copiamos los siguientes archivos
```
mv /root/dapiun/radiusd.conf /etc/freeradius/3.0/radiusd.conf
nano /etc/freeradius/3.0/radiusd.conf
```

```
\mv /root/dapiun/index.php /var/www/html/index.php
```

- Seguimos copiando archivos
```
\mv /root/dapiun/default /etc/freeradius/3.0/sites-enabled/default
nano /etc/freeradius/3.0/sites-enabled/default
```

```
\mv /root/dapiun/sqlcounter /etc/freeradius/3.0/mods-available/sqlcounter
\mv /root/dapiun/access_period.conf /etc/freeradius/3.0/mods-config/sql/counter/mysql/access_period.conf
\mv /root/dapiun/quotalimit.conf /etc/freeradius/3.0/mods-config/sql/counter/mysql/quotalimit.conf
\mv /root/dapiun/queries.conf /etc/freeradius/3.0/mods-config/sql/main/mysql/queries.conf
\mv /root/dapiun/radutmp /etc/freeradius/3.0/mods-enabled/radutmp
```
- Descargamos alguna base de datos de ejemplo con perfiles
```
wget https://raw.githubusercontent.com/wirisp/daloup/main/dbname.sql -O dbname.sql
```
- La restauramos con:
```
mysql -p -u root radius < /root/dbname.sql
```
- Ponle el password de tu db radius `84Uniq@`
- reiniciamos los servicios
```
systemctl status apache2
systemctl status freeradius
```
- Cambiar en el siguiente archivo unas lineas donde dice mysql por Mariadb
```
nano -l /var/www/html/daloradius/library/exten-radius_server_info.php
```
_Cambiar a:_
```
<td class='summaryKey'> MariaDB </td> <td class='summaryValue'><span class='sleft'><?php echo check_service("mariadb");  ?></span> </td>
```
- Darlepermisos a carpetas log
```
chmod 777  /var/log/syslog
chmod 777 /var/log/freeradius
chmod 755 /var/log/radius/
chmod 644 /var/log/radius/radius.log
chmod 644 /var/log/messages
chmod 644 /var/log/dmesg
touch /tmp/daloradius.log
```
- Reiniciar sistema e ingresar
```
reboot
```
- Ingresar a daloradius por la direccion `http://IP/daloradius` con usuario `administrator` y clave `radius` o `84Uniq@`
_Si hay error de puertos Es necesario que se abran los puertos en el vps de administracion 1812,1813,3306,6813,80,8080,443_

## Instalacion de WIREGUARD
```
git clone https://github.com/wirisp/pihole-wireguard.git
cd pihole-wireguard/
ls
mv piwire.sh /root
cd
chmod +x *.sh
bash piwire.sh 
```

```
nano /etc/sysctl.conf
```

```
net.ipv6.conf.all.disable_ipv6 = 0
kernel.panic = 10
```
```
sysctl -p
```
```
systemctl stop wg-quick@wg0.service
wg-quick down wg0
systemctl restart wg-quick@wg0.service
systemctl status wg-quick@wg0.service
```
- Si causa error el ping entonces cambiar dns con nano /etc/resolv.conf
```
ping google.com
```
```
nano /etc/resolv.conf
```
- agregar
```
nameserver 8.8.8.8
nameserver 8.8.4.4
```
- Reiniciamos con
```
service resolvconf restart
```
## instacion de  PIHOLE
```
curl -sSL https://install.pi-hole.net | bash
```
_Seleccionar interfaz wg0_

```
systemctl enable pihole-FTL
systemctl restart wg-quick@wg0.service
pihole -a -p
```
```
#curl -sSL https://install.pi-hole.net | bash
#curl -sSL https://install.pi-hole.net | PIHOLE_SKIP_OS_CHECK=true bash
```
## instalacion UNBOUND
```
cd pihole-wireguard/
chmod +x *.sh
./unbound.sh 
```

```
> /etc/pihole/setupVars.conf
```
```
echo "PIHOLE_INTERFACE=wg0
QUERY_LOGGING=true
INSTALL_WEB_SERVER=true
INSTALL_WEB_INTERFACE=true
LIGHTTPD_ENABLED=true
CACHE_SIZE=10000
DNS_FQDN_REQUIRED=true
DNS_BOGUS_PRIV=true
DNSMASQ_LISTENING=single
WEBPASSWORD=a31c87c18e9ff2eca7edb3aa0f7ee8ec24e92157a6f55d873115fd4084c37b0c
BLOCKING_ENABLED=true
DNSSEC=false
REV_SERVER=false
PIHOLE_DNS_1=127.0.0.1#5335
PIHOLE_DNS_2=127.0.0.1#5335" >> /etc/pihole/setupVars.conf 
```
```
systemctl enable pihole-FTL
```
```
reboot
```
```
systemctl restart wg-quick@wg0.service
systemctl status wg-quick@wg0.service
ip link show
```



- Restaldo de la db
```
mysqldump -p -u root radius > dbname.sql
```
- Restaurar db
```
mysql -p -u root radius < dbname.sql
```

- Desinstalacion de pihole
```
pihole uninstall
```
## Crear web php para impresion de vouchers online
 Para poder imprimir los vauchers de forma online sin herramientas, solamente clonaremos un repositorio y modificaremos los datos de acceso de nuestro servidor a la base de datos.
- Clonamos el repositorio 
```
git clone https://github.com/wirisp/printdalo.git print
mv print /var/www/html
```
- Seguimos el tutorial para modificar el acceso a la base de datos, si haz seguido esta instalacion solo modifica en
```
nano /var/www/html/print/index.php
```
- en la linea 35 cambia los datos
```
 $con = mysqli_connect("localhost","radius","84Uniq@","radius");
 ```
 - ***localhost** servidor
 - ***radius*** usuario de la base de datos
 - ***84Uniq@*** Password de la base de datos
 - ***radius*** nombre de la base de datos
 
 Tambien en este archivo puedes filtrar para impresion como desees, yo lo he echo con `batch_name` lo cual filtra por lote, pero igual podrias usar; `id` ,`username` etc, esto en la linea 40.
```
$query = "SELECT * FROM radius.fichas WHERE CONCAT(batch_name) LIKE '%$filtervalues%'";
```

### Impresion de vouchers desde online
Podemos imprimir los vouchers creados pero solamente para aquellos que fueron creados como lote, para ello necesitamos el nombre de lote

- Crear lote de vouchers en `http://IP/daloradius/mng-batch.php`
- rear lote de vouchers en `http://IP/daloradius/mng-batch.php`
- Copiar el nombre del lote, por si lo olvidaste, se encuentra listado en `http://IP/daloradius/mng-batch-list.php`
- Ir al apartado de impresion de vouchers `https://IP/print`
- Introducir el lote y darle en filtrar.

### Errores y soluciones posibles
- Reparar base de datos

```
mysql -u -p root radius

REPAIR TABLE TABLE;
```
## Buscar dentro de servidor
```
grep -rl "8.8.8.8" /etc
```
