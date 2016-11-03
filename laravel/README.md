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

### Migrar la base de datos
```bash
php artisan migrate
```
