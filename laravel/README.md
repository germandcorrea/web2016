# Laravel
[WEB oficial](https://laravel.com)

[Documentación](https://laravel.com/docs/5.3)

[Documentación Traducida](http://laraveles.com/docs/5.1/)

## Plugins necesarios de ATOM
la maquina virtual ya tiene instalado atom con los siguientes plugins funcionando.
- atom-bootstrap3
- atom-autocomplete-php
- aligner-php
- autocomplete-php
- blade-snippets
- language-blade
- laravel
- linter
- [linter-php](https://atom.io/packages/linter-php) para que funcione correctamente esté plugin es necesario que en la variable de entorno PATH se encuentre el directorio del ejecutable de PHP, para comprobar esto hay que abrir una ventana de comandos y escribir el comando **php -version** debe responder cual es la versión instalada de php caso contrario va a indicar que no encuentra el ejecutable de PHP.


## Composer
Es el administrador de dependencias de PHP.

[WEB Oficial de composer](https://getcomposer.org/)

### Actualizar composer en la maquina virtual
```bash
sudo composer selfupdate
```

## Crear un nuevo proyecto Laravel
en cualquier directorio podemos generar un nuevo proyecto con el siguiente comando.

```bash
composer create-project laravel/laravel app1
```
Luego de descargar todas las dependencias necesarias se va a generar una nueva carpeta en el directorio actual denominada **app1** que contiene todos los archivos del proyecto **Es Muy importante ejecutar todos los comandos desde dentro del directorio de la aplicación** por ese motivo luego de creado el proyecto es necesario moverse a ese directorio.

```bash
cd app1
```

### Iniciar el servidor de desarrollo con Artisan
```bash
php artisan serve
```
### Instalar laravel-debugbar
[laravel-debugbar](https://github.com/barryvdh/laravel-debugbar)

```bash
composer require barryvdh/laravel-debugbar
```

Una vez que se descargaron todas las dependencias es necesario editar el archivo **config/app.php** y agregar el siguiente elemento al array de **providers**

```php
Barryvdh\Debugbar\ServiceProvider::class,
```

También es necesario agregar el sigeuinte elemento al array de **aliases**

```php
'Debugbar' => Barryvdh\Debugbar\Facade::class,
```

### Generar la autentificación de usuarios

```bash
php artisan make:auth
```

### Configurar la conexión a la base de Datos
los parámetros de conexión deben modificarse en el archivo **.env**, laravel buscará la configuración de la aplicación primero desde variables de entorno del sistema operativo, luego desde el archivo **.env**, por último desde los valores por defecto.

la estructura del archivo es **clave=valor** debemos buscar y modificar los valores de las claves que inician con **DB_**.

```php
DB_DATABASE=miBaseDeDatos
DB_USERNAME=miUsuario
DB_PASSWORD=miContraseña
```

### Migrar la base de datos
```bash
php artisan migrate
```
### Moficar **/home** para mostrar el listado con los correos electronicos de los usuairos con una paginación de 10 usuarios por página.

#### Generar Datos aleatorios con los Seeders más Faker
[Documentación Oficial](https://laravel.com/docs/5.3/seeding)

Para poder ver en funcionamiento la paginación que genera laravel necesitamos rellenar la tabla de usuarios con datos, la idea es usar las herramientas incluidas en larvel para poder generar datos de forma automática con los **Seeders**, más la libreria **Faker**[https://github.com/fzaninotto/Faker](https://github.com/fzaninotto/Faker) que nos brinda un forma simple de generar datos aleatorios escribiendo código PHP.

##### crear el Seeder
Desde **artisan** tenemos disponible el comando **make:seeder** para generar el esqueleto de la clase.

```bash
php artisan make:seeder UsersTableSeeder
```

El comando anterior generará un archivo en el directorio **database/seeds/** con el nombre **UsersTableSeeder.php** el cual contiene una clase denominada **UsersTableSeeder**, en donde podemos observar el método vacio run, es en este método donde debemos escribir nuestro código que genere los datos en la base de datos.

Si abrimos el archivo **database/factories/ModelFactory.php** veremos la definición de los datos aleatorios para el modelo **App/User**, básicamente retorna un array asociativo donde las claves del array son los nombres de los atributos del modelo, asociados a los valores que se generarán aletoriamente usando el servicio de faker (**$faker**), de manera que en nuestras clases de Seeder podremos utilizar el método **factory** para generar la cantidad de objetos que necesitemos para el Modelo.
De manera tal que el nuestro método run quedara de la siguiente manera.

```php
public function run()
{
    factory(App\User::class,100)->create();
}
```

No debemos olvidar habilitar nuestro Seeder en el archivo **database/seeds/DatabaseSeeder.php**, agregando una línea al método **run** de la clase **DatabaseSeeder** indicando el nombre de la clase de nuestro Seeder.

```php
public function run()
{
    $this->call(UsersTableSeeder::class);
}
```
##### Ejecutar los Seeder
Finalmente podremos ejecutar desde **artisan** el comando **db:seed** para ejecutar los Seeders y rellenar las bases de datos.

```bash
php artisan db:seed
```

una menera alternativa es hacerlo de forma automatica despues de aplicar una migración.

```bash
php artisan migrate --seed
```

o en la migración eliminar las tablas de la base de datos y regenrar la estructura para luego rellenar con datos.

```bash
php artisan migrate:refresh --seed
```

### Modificar controladores y vistas
para identificar que método de que controlador y que vista debemos modificar para poder resolver el enunciado lo primero que tenemos que revisar son las rutas en el archivo **routes/web.php** y buscar la ruta **/home**.

```php
Route::get('/home', 'HomeController@index');
```

De esa Ruta podemos inferir que cuando un cliente solicite la ruta **/home** con verbo http **GET** se ejecutará el método **index** de la clase **HomeController**.


## Agregar la dependencia **laracasts/generators**

https://github.com/laracasts/Laravel-5-Generators-Extended

Con está librería vamos a agregar comandos a **artisan** para generar las migraciones de forma más simple.

```bash
composer require laracasts/generators --dev
```

Registrar la librería para que funcione en el entorno de desarrollo, buscamos el archivo **app/Providers/AppServiceProvider.php** y en el método **register** agregamos

```php
if ($this->app->environment() == 'local') {
	$this->app->register(\Laracasts\Generators\GeneratorsServiceProvider::class);
}
```

Los nuevos comandos de artisan son:
- make:migration:schema
- make:migration:pivot
- make:seed



## Habilitar la dependencia l5scaffold

https://github.com/laralib/l5scaffold

Scaffolding de Bootstrap 3 a Laravel 5
instalar el paquete l5scaffold

```bash

composer require 'laralib/l5scaffold' --dev
```

Registrar el Service Provider en el archivo **config/app.php**, buscar el array
**'providers'** y agregar el elemento

```php

Laralib\L5scaffold\GeneratorsServiceProvider::class,
```

ahora se agregó un nuevo comando en artisan **make:scaffold** que necesita minimamente dos parámetros, el primero es el nombre de la Clase de Modelo y como segundo el parámetro el esquema del modelo con la opción **--schema** que tiene una cadena donde se define la estructura y restricciones de los atributos, con un formato simple, cada atributo está definido indicando **nombre_atributo:tipo_de_dato:restricciones_separadas_por_dos_puntos** y separando cada atributo por una "coma".

por ejemplo podemos generar un modelo denominado **Tweet** con los atributos **title** de tipo **string** con un valor por defecto **Tweet #1**, más el atributo **body** de tipo **text**

```bash
php artisan make:scaffold Tweet --schema="title:string:default('Tweet #1'), body:text"
```


## Desplegar la aplicación en Heroku

https://devcenter.heroku.com/articles/getting-started-with-laravel

lo primero que necesitamos es crear una cuenta en heroku, es gratuito para eso hay que ir a la página de heroku.com y registarse.
luego es necesario instalar la herramienta de linea de comandos para comunicarse con heroku.

```bash
wget -O- https://toolbelt.heroku.com/install-ubuntu.sh | sh
```

inciar Sesión en heroku, nos va a pedir el correo electronico y la passaword con la que nos registramos en el sitio.

```bash
heroku login
```

Generar el archivo **Procfile** donde indicamos que vamos a usar como servidor WEB una instancia de apache2 con php configurado apuntando **DocumentRoot** al directorio **public** de nuestra aplicación.  

```bash
echo "web: vendor/bin/heroku-php-apache2 public/" > Procfile
```

Heroku como la mayoría de servicios web en la nube, utilizan **GIT** para subir los archivos al repositorio del servidor. Por lo tanto es necesario iniciar un repositorio git dentro de nuestro proyecto **git init**, agregar todos los archivos del proyecto al repositorio **git add .** y por ultimo confirmar los que los archivos estan listos en nuestro repositorio **git commit -m "app: inicio de repositorio"**, todos esto es en nuestro repositorio local.

```bash
git init
git add .
git commit -m "app: inicio de repositorio"
```

crear la aplicación

```
heroku create adm-proy-app-germandcorrea
```

por defecto Heroku entiende que nuestra aplicación se va a desarrollar en **nodejs** de modo que debemos especificarle un build pack correspondientes a **PHP**

```bash
heroku buildpacks:set heroku/php
```

agregar PostgreSQL a nuestra aplicación en el servidor de heroku, luego podremos ver que en el heroku tenemos una variable de entorno **DATABASE_URL** que contiene los parámetros de conección hacia el servidor de base de datos.

```bash
heroku addons:create heroku-postgresql:hobby-dev
```

declaramos las variables que necesita laravel, para funcionar.
configuramos las variables de entorno en el servidor de heroku, es decir las variables que teniamos definidas en nuestro archivo **.env** tienen que definirse como variables de entorno en los servidores de heroku.

variables para la clave secreta de la aplicación.

```bash
heroku config:set APP_KEY=$(php artisan key:generate --show)
```


variables para la configuración de la Base de Datos.

si ejecutamos el siguiente comando podremos ver las variables de entorno que tiene nuestra instancia de heroku.
```bash
heroku config -s
```
el comando anterior nos mostrará una variable **DATABASE_URL** con un formato parecido al siguiente

**DATABASE_URL='postgres://usuario:contraseña@host:5432/base_de_datos'**

entonces debemos descomponer esa url, que tiene todos los parámetros de conexión a la base de datos juntos, que nos proporción heroku, para luego generar cada una de las variables que necesita laravel para conectarse.

```bash
heroku config:set DB_CONNECTION=pgsql
heroku config:set DB_HOST=host
heroku config:set DB_USERNAME=usuario
heroku config:set DB_PASSWORD=contraseña
heroku config:set DB_DATABASE=base_de_datos
```

una vez configurada la application es hora de subir la aplicación al servidor de Heroku, nuevamente mediante **GIT**

```bash
git push heroku master
```

Corremos las migraciones en el servidor de heroku

```bash
heroku run php artisan migrate
```

Lanzamos nuestra aplicaión en el navegador web
```bash
heroku open
```
