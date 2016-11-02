<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [RRHH](#rrhh)
	- [LAMP](#lamp)
	- [Base de datos (MySQL)](#base-de-datos-mysql)
	- [Servidor WEB Apache 2](#servidor-web-apache-2)
	- [Scripts PHP con acceso bases de datos MySQL](#scripts-php-con-acceso-bases-de-datos-mysql)
	- [Ejercicios](#ejercicios)
		- [Ejercicios Resueltos](#ejercicios-resueltos)
			- [Ejercicios 1 y 2](#ejercicios-1-y-2)
			- [Ejercicios 3](#ejercicios-3)
			- [Ejercicios 4](#ejercicios-4)

<!-- /TOC -->
# RRHH
## LAMP
**L**inux, **A**pache, **M**ySQL y **P**HP

Para el curso vamos a usar una maquina virtual que tiene instalado lubuntu 16.04, con la mayoria de los componentes necesarios para poder desarrollar aplicaciones web.

Para lo cual tendremos un usuario que usamos normalmente para trabajar en la maquina virtual.

usuario: **web**
contraseña: **web**

## Base de datos (MySQL)

usuario: **root**
contraseña: **datos**

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

si cuando restauramos la base de datos nos muestra en error de **storage_engine** es necesario editar el archivo de employees.sql y reemplazar donde aparece la palabra **storage_engine** por **default_storage_engine**.
abrimos el editor atom en el directorio actual.

```bash
atom .
```
comprobar la estructura y datos de la base de datos.
```bash
time mysql -u root -p -t < test_employees_md5.sql
```

conectar mysql Workbench para trabajar con la base de datos

## Servidor WEB Apache 2
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
## Scripts PHP con acceso bases de datos MySQL

para poder acceder a la base de datos de MySQL es conveniente usar la libreria PDO [http://php.net/manual/es/book.pdo.php](http://php.net/manual/es/book.pdo.php)
quien tendrá de responsablidad de proporcionar una capa de abstracción para manipular bases de datos independientemente del fabricante es decir el código php será el mismo sin importar si la Base de Datos es MySQL, PostgreSQL, Sqlite, Sql Server, Oracle, etc. no así la sintaxis SQL implementada en cada motor.

vamos a crear una carpeta en el DocumentRoot del servidor Apache (**/var/www/html**) con el nombre de la comisión por ejemplo **3k4**.

```bash
mkdir /var/www/html/3k4
```

ahora abrimos el editor de texto atom en ese directorio.

```bash
atom /var/www/html/3k4
```

generamos un archivo denominado **cnn.php** donde pondremos los parametros de conexión a la base de datos.

```php
<?php
$dsn='mysql: host=localhost; dbname=employees';
$usuario='root';
$password='datos';
try{
  $cnn=new PDO($dsn,$usuario,$password);
}catch(PDOException $e){
  die('Error al conectarse a la base de datos: <br>'.$e->getMessage());
}
```

ahora generamos el archivo **departamentos.php** que tendrá la responsabilidad de mostrar un listado de los departamentos de la base de datos.

```php
<?php
/*
incluimos el archivo de la conexión a la base de datos.
entonces ahora podremos usar las variables definidas en el archivo cnn.php como por ejemplo $cnn;
*/

include 'cnn.php';

/*
generamos una variable $sql que contiene la consulta sql que enviaremos a la base de datos.
*/
$sql = 'select * from departments';
try{
  /*
  preparamos la consulta sql
  */
  // http://php.net/manual/es/pdo.prepared-statements.php
  $query=$cnn->prepare($sql);
  /* ejecutamos la consulta */
  //http://php.net/manual/es/pdostatement.execute.php
  $query->execute();
  /* obtenemos los todos los registros de la consulta y lo almacenamos en la variable $departamentos */
  //http://php.net/manual/es/pdostatement.fetchall.php
  $departamentos=$query->fetchAll();
  /*
  nuevamente podríamos cometer algún error al enviar la consulta sql y obtener los datos desde la base de datos. entonces debemos capturar la excepción y tratarla
  */
}catch(PDOException $e){
  die('Error en la consulta: <br>'.$e->getMessage());
}
//podemos ver el array retornado con la siguiente línea
//print_r($departamentos);
?>
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>departamentos</title>

    <!-- Bootstrap -->
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/css/bootstrap.min.css">
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/font-awesome/4.4.0/css/font-awesome.min.css">

    <!-- HTML5 Shim and Respond.js IE8 support of HTML5 elements and media queries -->
    <!-- WARNING: Respond.js doesn't work if you view the page via file:// -->
    <!--[if lt IE 9]>
      <script src="https://oss.maxcdn.com/libs/html5shiv/3.7.2/html5shiv.js"></script>
      <script src="https://oss.maxcdn.com/libs/respond.js/1.4.2/respond.min.js"></script>
    <![endif]-->
  </head>
  <body>
    <div class="container">
      <div class="table-responsive">
        <table class="table table-bordered table-striped table-hover">
          <tr>
            <th>#</th>
            <th>departamento</th>
          </tr>
          <!-- http://php.net/manual/es/language.control-structures.php -->
          <!-- http://php.net/manual/es/control-structures.alternative-syntax.php -->
          <!-- iteramos sobre el array de departamentos ($departamentos) para generar una fila de una tabla con cada uno de los registros de la iteración, almacenamos en la variable $depto -->
          <?php foreach ($departamentos as $depto): ?>
            <tr>
              <td><?php echo $depto['dept_no']; ?></td>
              <td>
                <!-- se puede usar la notación alternativa para imprimir valores simples, pero tener cuidado no simpre está activa está opción (short_open_tags) en la configuración de php http://php.net/manual/es/language.basic-syntax.phptags.php -->
                <?= $depto['dept_name'] ?>
              </td>
            </tr>
          <?php endforeach; ?>
        </table>
      </div> <!-- /div.table-responsive -->
    </div><!-- /div.container -->
    <!-- jQuery (necessary for Bootstrap's JavaScript plugins) -->
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.11.3/jquery.min.js"></script>
    <!-- Include all compiled plugins (below), or include individual files as needed -->
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/js/bootstrap.min.js"></script>
  </body>
</html>
```
## Ejercicios

1. Mostrar un listado de los departamentos, indicando el nombre de los departamentos junto a los apellidos y nombres del jefe actual, como así también la fecha desde cuando ocupa ese cargo.
2. El Listado debé permitir filtrar los datos por nombre de departmento.
3. Al seleccionar un departamento mostrar apellido y nombre del jefe departamento junto a la fecha desde donde comenzó a trabajar en ese departamento, además un listado que contenga los mismos datos de cada empleado del departamento.
4. Al seleccionar un empleado, mostrar los departamentos donde fue jefe y el periodo, también mostrar todos los departamentos donde presto servicio junto al periodo.

### Ejercicios Resueltos

#### Ejercicios 1 y 2

generamos el archivo **departamentos.php**

```php
<?php
/*
incluimos el archivo de la conexión a la base de datos.
entonces ahora podremos usar las variables definidas en el archivo cnn.php como por ejemplo $cnn;
*/

include 'cnn.php';

/*
generamos una variable $sql que contiene la consulta sql que enviaremos a la base de datos.
*/
$NAME=$_GET['term'];
//$sql = 'select * from departments WHERE dept_name LIKE \'%serv%\'';
//$sql = "select * from departments WHERE dept_name LIKE :NAME ";
$sql="SELECT e.emp_no, e.first_name, e.last_name, d.dept_no, d.dept_name, dm.from_date
FROM departments d
INNER JOIN dept_manager dm ON d.dept_no = dm.dept_no
INNER JOIN employees e ON dm.emp_no = e.emp_no
WHERE
	dm.to_date = '9999-01-01'
	AND dept_name LIKE :NAME ";

try{
  /*
  preparamos la consulta sql
  */
  // http://php.net/manual/es/pdo.prepared-statements.php
  $query=$cnn->prepare($sql);
  /* ejecutamos la consulta */
  //http://php.net/manual/es/pdostatement.execute.php
  $query->execute([':NAME'=>'%'.$NAME.'%']);
  /* obtenemos los todos los registros de la consulta y lo almacenamos en la variable $departamentos */
  //http://php.net/manual/es/pdostatement.fetchall.php
  $departamentos=$query->fetchAll();
  //echo json_encode($departamentos);
  //die();
  /*
  nuevamente podríamos cometer algún error al enviar la consulta sql y obtener los datos desde la base de datos. entonces debemos capturar la excepción y tratarla
  */
}catch(PDOException $e){
  die('Error en la consulta: <br>'.$e->getMessage());
}
//podemos ver el array retornado con la siguiente línea
//print_r($departamentos);
?>
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>departamentos</title>

    <!-- Bootstrap -->
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/css/bootstrap.min.css">
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/font-awesome/4.4.0/css/font-awesome.min.css">

    <!-- HTML5 Shim and Respond.js IE8 support of HTML5 elements and media queries -->
    <!-- WARNING: Respond.js doesn't work if you view the page via file:// -->
    <!--[if lt IE 9]>
      <script src="https://oss.maxcdn.com/libs/html5shiv/3.7.2/html5shiv.js"></script>
      <script src="https://oss.maxcdn.com/libs/respond.js/1.4.2/respond.min.js"></script>
    <![endif]-->
  </head>
  <body>
    <div class="container">
     <form class="" action="?" method="get">
       <div class="panel panel-default">
         <div class="panel-heading">
           <h3 class="panel-title">Departamentos</h3>
         </div>
         <div class="panel-body">
            <div class="form-group">
              <label for="q">filtrar</label>
              <input id="q" type="text" class="form-control" placeholder="" name="term" value="<?php echo $NAME; ?>">
              <p class="help-block">Help text here.</p>
            </div>
         </div>
         <div class="panel-footer">
           <input type="submit" name="btn" value="enviar" class="btn btn-primary">
         </div>
       </div>
     </form>
      <div class="table-responsive">
        <table class="table table-bordered table-striped table-hover">
          <tr>
            <th>#</th>
            <th>departamento</th>
            <th>jefe</th>
            <th>desde</th>
          </tr>
          <!-- http://php.net/manual/es/language.control-structures.php -->
          <!-- http://php.net/manual/es/control-structures.alternative-syntax.php -->
          <!-- iteramos sobre el array de departamentos ($departamentos) para generar una fila de una tabla con cada uno de los registros de la iteración, almacenamos en la variable $depto -->
          <?php foreach ($departamentos as $depto): ?>
            <tr>
              <td><?php echo $depto['dept_no']; ?></td>
              <td>
                <!-- se puede usar la notación alternativa para imprimir valores simples, pero tener cuidado no simpre está activa está opción (short_open_tags) en la configuración de php http://php.net/manual/es/language.basic-syntax.phptags.php -->
                <a href='departamento-show.php?id=<?php echo $depto['dept_no']; ?>'>
                  <?= $depto['dept_name'] ?>
                </a>
              </td>
              <td>
                <a href='empleado-show.php?id=<?php echo $depto['emp_no']; ?>'>
                <?php echo $depto['last_name'].', '.$depto['first_name']; ?>
                </a>
              </td>
              <td><?php echo $depto['from_date']; ?></td>
            </tr>
          <?php endforeach; ?>
        </table>
      </div> <!-- /div.table-responsive -->
    </div><!-- /div.container -->
    <!-- jQuery (necessary for Bootstrap's JavaScript plugins) -->
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.11.3/jquery.min.js"></script>
    <!-- Include all compiled plugins (below), or include individual files as needed -->
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/js/bootstrap.min.js"></script>
  </body>
</html>
```
#### Ejercicios 3

generamos el archivo **departamento-show.php**

```php
<?php
/*
incluimos el archivo de la conexión a la base de datos.
entonces ahora podremos usar las variables definidas en el archivo cnn.php como por ejemplo $cnn;
*/

include 'cnn.php';

//obtenemos de la solicitud el parameto id
$ID=$_GET['id'];

/*
generamos una variable $sql que contiene la consulta sql que enviaremos a la base de datos.
*/
//$sql = 'select * from departments WHERE dept_name LIKE \'%serv%\'';
//$sql = "select * from departments WHERE dept_name LIKE :NAME ";
$sql="SELECT e.emp_no, e.first_name, e.last_name, d.dept_no, d.dept_name, dm.from_date
FROM departments d
INNER JOIN dept_manager dm ON d.dept_no = dm.dept_no
INNER JOIN employees e ON dm.emp_no = e.emp_no
WHERE
	dm.to_date = '9999-01-01'
	AND d.dept_no = :ID ";

$sqlEmpleados="SELECT e.emp_no,e.last_name,e.first_name,de.from_date
	FROM employees e
	INNER JOIN dept_emp de ON de.emp_no = e.emp_no
	WHERE
		de.to_date = '9999-01-01'
		AND de.dept_no = :ID
ORDER BY e.last_name,e.first_name";

try{
  /*
  preparamos la consulta sql
  */
  // http://php.net/manual/es/pdo.prepared-statements.php
  $query=$cnn->prepare($sql);
  /* ejecutamos la consulta */
  //http://php.net/manual/es/pdostatement.execute.php
  $query->execute([':ID'=>$ID]);
  /* obtenemos los todos los registros de la consulta y lo almacenamos en la variable $departamentos */
  //http://php.net/manual/es/pdostatement.fetchall.php
  $depto=$query->fetchAll();
	//si la cantidad de elementos del array retornado es igual a uno
	//almacenamos el primer elemento del array en la misma variable
	if(count($depto)==1){
		$depto=$depto[0];
	}

	/*
  preparamos la consulta sql para obtener los empleados
  */
  // http://php.net/manual/es/pdo.prepared-statements.php
  $queryEmp=$cnn->prepare($sqlEmpleados);
  /* ejecutamos la consulta */
  //http://php.net/manual/es/pdostatement.execute.php
  $queryEmp->execute([':ID'=>$ID]);
  /* obtenemos los todos los registros de la consulta y lo almacenamos en la variable $departamentos */
  //http://php.net/manual/es/pdostatement.fetchall.php
  $empleados=$queryEmp->fetchAll();
  //echo json_encode($departamentos);
  //die();
	//print_r($depto);
  /*
  nuevamente podríamos cometer algún error al enviar la consulta sql y obtener los datos desde la base de datos. entonces debemos capturar la excepción y tratarla
  */
}catch(PDOException $e){
  die('Error en la consulta: <br>'.$e->getMessage());
}
//podemos ver el array retornado con la siguiente línea
//print_r($departamentos);
?>
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>departamentos</title>

    <!-- Bootstrap -->
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/css/bootstrap.min.css">
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/font-awesome/4.4.0/css/font-awesome.min.css">

    <!-- HTML5 Shim and Respond.js IE8 support of HTML5 elements and media queries -->
    <!-- WARNING: Respond.js doesn't work if you view the page via file:// -->
    <!--[if lt IE 9]>
      <script src="https://oss.maxcdn.com/libs/html5shiv/3.7.2/html5shiv.js"></script>
      <script src="https://oss.maxcdn.com/libs/respond.js/1.4.2/respond.min.js"></script>
    <![endif]-->
  </head>
  <body>
    <div class="container">
			<div class="page-header">
			  <h1><?php echo 'Departamento '.$depto['dept_name']; ?> </h1>
			</div>
     	<div class="jefe">
				<h2>Jefe "<?php echo $depto['last_name'].', '.$depto['first_name']; ?>" desde <?php echo $depto['from_date']; ?></h2>
     	</div><!-- ./jefe -->
      <div class="table-responsive">
        <table class="table table-bordered table-striped table-hover">
          <tr>
            <th>#</th>
            <th>nombre empleado</th>
            <th>desde</th>
          </tr>
          <!-- http://php.net/manual/es/language.control-structures.php -->
          <!-- http://php.net/manual/es/control-structures.alternative-syntax.php -->
          <!-- iteramos sobre el array de departamentos ($departamentos) para generar una fila de una tabla con cada uno de los registros de la iteración, almacenamos en la variable $depto -->
          <?php foreach ($empleados as $emp): ?>
            <tr>
              <td><?php echo $emp['emp_no']; ?></td>
              <td>
                <a href='empleado-show.php?id=<?php echo $emp['emp_no']; ?>'>
                <?php echo $emp['last_name'].', '.$emp['first_name']; ?>
                </a>
              </td>
              <td><?php echo $emp['from_date']; ?></td>
            </tr>
          <?php endforeach; ?>
        </table>
      </div> <!-- /div.table-responsive -->
    </div><!-- /div.container -->
    <!-- jQuery (necessary for Bootstrap's JavaScript plugins) -->
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.11.3/jquery.min.js"></script>
    <!-- Include all compiled plugins (below), or include individual files as needed -->
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/js/bootstrap.min.js"></script>
  </body>
</html>
```

#### Ejercicios 4

generamos el archivo **empleado-show.php**

```php
<?php
/*
incluimos el archivo de la conexión a la base de datos.
entonces ahora podremos usar las variables definidas en el archivo cnn.php como por ejemplo $cnn;
*/

include 'cnn.php';

//obtenemos de la solicitud el parameto id
$ID=$_GET['id'];

/*
generamos una variable $sqlEmpleado que contiene la consulta sql que enviaremos a la base de datos.
*/
//$sql = 'select * from departments WHERE dept_name LIKE \'%serv%\'';
//$sql = "select * from departments WHERE dept_name LIKE :NAME ";
$sqlEmpleado="SELECT *
FROM employees
WHERE emp_no = :ID ";

$sqlDepartamentosDondeFueJefe="SELECT d.dept_no,d.dept_name,dm.from_date,dm.to_date
FROM dept_manager dm
    INNER JOIN departments d ON dm.dept_no = d.dept_no
WHERE
	dm.emp_no = :ID
ORDER BY dm.from_date";

$sqlDepartamentosDondeFueEmpleado="SELECT d.dept_no,d.dept_name,de.from_date,de.to_date
FROM dept_emp de
    INNER JOIN departments d ON de.dept_no = d.dept_no
WHERE
	de.emp_no = :ID
ORDER BY de.from_date";

try{
  /*
  preparamos la consulta sql
  */
  // http://php.net/manual/es/pdo.prepared-statements.php
  $query=$cnn->prepare($sqlEmpleado);
  /* ejecutamos la consulta */
  //http://php.net/manual/es/pdostatement.execute.php
  $query->execute([':ID'=>$ID]);
  /* obtenemos los todos los registros de la consulta y lo almacenamos en la variable $departamentos */
  //http://php.net/manual/es/pdostatement.fetchall.php
  $empleado=$query->fetchAll();
	//si la cantidad de elementos del array retornado es igual a uno
	//almacenamos el primer elemento del array en la misma variable
	if(count($empleado)==1){
		$empleado=$empleado[0];
	}

	/*
  preparamos la consulta sql para obtener los Departamentos Donde Fue Jefe
  */
  // http://php.net/manual/es/pdo.prepared-statements.php
  $queryDepartamentosDondeFueJefe=$cnn->prepare($sqlDepartamentosDondeFueJefe);
  /* ejecutamos la consulta */
  //http://php.net/manual/es/pdostatement.execute.php
  $queryDepartamentosDondeFueJefe->execute([':ID'=>$ID]);
  /* obtenemos los todos los registros de la consulta y lo almacenamos en la variable $departamentos */
  //http://php.net/manual/es/pdostatement.fetchall.php
  $DepartamentosDondeFueJefe=$queryDepartamentosDondeFueJefe->fetchAll();

	/*
  preparamos la consulta sql para obtener los Departamentos Donde Fue Jefe
  */
  // http://php.net/manual/es/pdo.prepared-statements.php
  $queryDepartamentosDondeFueEmpleado=$cnn->prepare($sqlDepartamentosDondeFueEmpleado);
  /* ejecutamos la consulta */
  //http://php.net/manual/es/pdostatement.execute.php
  $queryDepartamentosDondeFueEmpleado->execute([':ID'=>$ID]);
  /* obtenemos los todos los registros de la consulta y lo almacenamos en la variable $departamentos */
  //http://php.net/manual/es/pdostatement.fetchall.php
  $DepartamentosDondeFueEmpleado=$queryDepartamentosDondeFueEmpleado->fetchAll();
  //echo json_encode($departamentos);
  //die();
	//print_r($depto);
  /*
  nuevamente podríamos cometer algún error al enviar la consulta sql y obtener los datos desde la base de datos. entonces debemos capturar la excepción y tratarla
  */
}catch(PDOException $e){
  die('Error en la consulta: <br>'.$e->getMessage());
}
//podemos ver el array retornado con la siguiente línea
//print_r($departamentos);
?>
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>departamentos</title>

    <!-- Bootstrap -->
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/css/bootstrap.min.css">
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/font-awesome/4.4.0/css/font-awesome.min.css">

    <!-- HTML5 Shim and Respond.js IE8 support of HTML5 elements and media queries -->
    <!-- WARNING: Respond.js doesn't work if you view the page via file:// -->
    <!--[if lt IE 9]>
      <script src="https://oss.maxcdn.com/libs/html5shiv/3.7.2/html5shiv.js"></script>
      <script src="https://oss.maxcdn.com/libs/respond.js/1.4.2/respond.min.js"></script>
    <![endif]-->
  </head>
  <body>
    <div class="container">
			<div class="page-header">
			  <h1><?php echo 'Empleado '.$empleado['last_name'].', '.$empleado['first_name']; ?> </h1>
			</div>
			<div class="table-responsive">
				<table class="table table-bordered table-striped table-hover">
					<tr>
						<th>
							Apellidos
						</th>
						<td>
							<?php echo $empleado['last_name']; ?>
						</td>
					</tr>
					<tr>
						<th>
							Nombres
						</th>
						<td>
							<?php echo $empleado['first_name']; ?>
						</td>
					</tr>
					<tr>
						<th>
							Fecha de nacimiento
						</th>
						<td>
							<?php echo $empleado['birth_date']; ?>
						</td>
					</tr>
					<tr>
						<th>
							Sexo
						</th>
						<td>
							<?php echo $empleado['gender']; ?>
						</td>
					</tr>
					<tr>
						<th>
							Fecha de reclutamiento
						</th>
						<td>
							<?php echo $empleado['hire_date']; ?>
						</td>
					</tr>
				</table>
			</div>
      <div class="table-responsive">
				<h2>Oficinas donde fue Jefe</h2>
				<?php if (count($DepartamentosDondeFueJefe)==0): ?>
					<h3 class="text-center alert alert-info">Aún no fue Jefe de Departamento.</h3>
				<?php else: ?>
					<table class="table table-bordered table-striped table-hover">
	          <tr>
	            <th>Departamento</th>
	            <th>Desde</th>
							<th>Hasta</th>
	          </tr>
	          <!-- http://php.net/manual/es/language.control-structures.php -->
	          <!-- http://php.net/manual/es/control-structures.alternative-syntax.php -->
	          <!-- iteramos sobre el array de departamentos ($departamentos) para generar una fila de una tabla con cada uno de los registros de la iteración, almacenamos en la variable $depto -->
	          <?php foreach ($DepartamentosDondeFueJefe as $dj): ?>
	            <tr>
	              <td>
									<a href='departamento-show.php?id=<?php echo $dj['dept_no']; ?>'>
	                  <?= $dj['dept_name'] ?>
	                </a>
	              </td>
	              <td><?php echo $dj['from_date']; ?></td>
	              <td><?php echo $dj['to_date']; ?></td>
	            </tr>
	          <?php endforeach; ?>
	        </table>
				<?php endif; ?>
			</div> <!-- /div.table-responsive -->
			<div class="table-responsive">
				<h2>Oficinas donde fue Empleado</h2>
				<?php if (count($DepartamentosDondeFueEmpleado)==0): ?>
					<h3 class="text-center alert alert-info">Aún no fue Empleado de Departamento.</h3>
				<?php else: ?>
					<table class="table table-bordered table-striped table-hover">
	          <tr>
	            <th>Departamento</th>
	            <th>Desde</th>
							<th>Hasta</th>
	          </tr>
	          <!-- http://php.net/manual/es/language.control-structures.php -->
	          <!-- http://php.net/manual/es/control-structures.alternative-syntax.php -->
	          <!-- iteramos sobre el array de departamentos ($departamentos) para generar una fila de una tabla con cada uno de los registros de la iteración, almacenamos en la variable $depto -->
	          <?php foreach ($DepartamentosDondeFueEmpleado as $de): ?>
	            <tr>
	              <td>
									<a href='departamento-show.php?id=<?php echo $de['dept_no']; ?>'>
	                  <?= $de['dept_name'] ?>
	                </a>
	              </td>
	              <td><?php echo $de['from_date']; ?></td>
	              <td><?php echo $de['to_date']; ?></td>
	            </tr>
	          <?php endforeach; ?>
	        </table>
				<?php endif; ?>
			</div> <!-- /div.table-responsive -->

    </div><!-- /div.container -->
    <!-- jQuery (necessary for Bootstrap's JavaScript plugins) -->
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.11.3/jquery.min.js"></script>
    <!-- Include all compiled plugins (below), or include individual files as needed -->
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/js/bootstrap.min.js"></script>
  </body>
</html>
```
