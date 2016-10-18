# RRHH
## Base de datos (MySQL)
vamos a utilizar la base de datos de Recursos Humanos que ofrece MySQL [https://dev.mysql.com/doc/employee/en/](https://dev.mysql.com/doc/employee/en/) para luego generar una interfaz web que manipule los datos.

debemos descargar la base de datos de ejemplo desde [https://launchpad.net/test-db/employees-db-1/1.0.6/+download/employees_db-full-1.0.6.tar.bz2]( https://launchpad.net/test-db/employees-db-1/1.0.6/+download/employees_db-full-1.0.6.tar.bz2)

descomprimir el archivo de la base de datos.

```bash
tar -xvjf employees_db-full-1.0.6.tar.bz2
cd employees_db
```

restaurar la base de datos

```bash
mysql -t -u root -p < employees.sql
```
**-t** es para mostrar los resultados en forma de tabla

**-u root** es para indicar que se va iniciar sesión con el usuario root.

**-p** va a pedir la contraseña del usuario, en nuestra maquina virtual la contraseña del usuario root de mysql es **datos**

comprobar la estructura y datos de la base de datos.
```bash
time mysql -u root -p -t < test_employees_md5.sql
```

conectar mysql Workbench para trabajar con la base de datos

## Servidor WEB apache 2
en la maquina virtual está instalado el Servidor Web apache 2, hay que verificar que apache entienda el lenguaje php.

verificamos los permisos de la carpeta DocumentRoot de apache que se encuentra en /var/www/html

listamos el contenido de /var/www/html

```bash
ls -alh /var/www/html
```
podemos apreciar en la primera columna con la notación rwx (**R**ead, **W**rite ,e**X**ecute) los permisos para el usuario dueño, grupo dueño y otros usuarios.

en la tercera columna el **usuario dueño**, si aparece **root** es usuario "administrador o super usuario" de linux.

en la cuarta columna el **grupo dueño**.

debemos cambiar el usuario dueño de la carpeta DocumentRoot junto a todos los archivos  

```bash
sudo chown web -vfR /var/www/html
```
**sudo** ejecutar el comando como super usuario.

**chown** cambiar el dueño del archivo/carpeta.

**web** es el usuario que usamos normalmente para usar la maquina virtual.

**-vfR** son las opciones del comando que significan **v** verbose **f** force **R** Recursive

ahora para comprobar que funcione podemos abrir nuestro editor de texto en el directorio DocumentRoot

```bash
atom /var/www/html
```
ahora desde el editor de texto creamos el archivo **testphp.php** en el directorio **/var/www/html**
que contenga el siguiente código php.

```php
<?php
phpinfo();
```

ahora accedemos desde el navegador web a la url **[http://localhost/testphp.php](http://localhost/testphp.php)**
podremos apreciar la configuración de PHP, junto a los módulos que tiene funcionando.

si muestra una página en blanco es por que apache no está configurado correctamente. y es necesaro instalar en el Servidor WEB el modulo de php con el siguiente comando

```bash
sudo apt-get install libapache2-mod-php
```

una vez instalado el módulo es necesario reiniciar el servicio de apache para que inicie con la nueva configuración.

```bash
sudo service apache2 restart
```
