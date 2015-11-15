# Desarrollando un Administrador de Proyectos en Laravel 5.1

## Indice de Contenido

<!-- TOC depth:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Desarrollando un Administrador de Proyectos en Laravel 5.1](#desarrollando-un-administrador-de-proyectos-en-laravel-51)
	- [Indice de Contenido](#indice-de-contenido)
	- [Preparar la Aplicación](#preparar-la-aplicacin)
		- [Crear una Nueva Aplicación en Laravel](#crear-una-nueva-aplicacin-en-laravel)
		- [Agregamos la dependencia laravel-debugbar](#agregamos-la-dependencia-laravel-debugbar)
		- [Agregar la dependencia **laracasts/generators**](#agregar-la-dependencia-laracastsgenerators)
		- [Habilitar la dependencia l5scaffold](#habilitar-la-dependencia-l5scaffold)
		- [Generar una base de datos](#generar-una-base-de-datos)
		- [Modificar el archivo de configuración de entorno **.env**](#modificar-el-archivo-de-configuracin-de-entorno-env)
		- [Layouts](#layouts)
			- [plantillas comunes](#plantillas-comunes)
	- [autentificación de usuarios](#autenticacin-de-usuarios)
		- [El modelo User](#el-mdelo-user)
			- [Migrar la Base de Datos](#migrar-la-base-de-datos)
			- [Poblando la Base de Datos](#poblando-la-base-de-datos)
			- [Interactuar con los datos](#interactuar-con-los-datos)
		- [Controladores](#controladores)
		- [Vistas](#vistas)
			- [Vistas para autentificación de usuario](#vistas-para-autenticacin-de-usuario)
	- [modelos de Proyecto y Tareas](#mdelos-de-proyecto-y-tareas)
			- [Generar el modelo Proyecto](#generar-el-mdelo-proyecto)
			- [Generar el modelo Tarea](#generar-el-mdelo-tarea)
			- [Definir las Relaciones](#definir-las-relaciones)
			- [Agregar un atributo al Modelo Tarea](#agregar-un-atributo-al-modelo-tarea)
			- [Datos de prueba](#datos-de-prueba)
	- [Controlador de Proyectos](#controlador-de-proyectos)
	- [Rutas para Proyectos](#rutas-para-proyectos)
	- [Vistas Proyecto](#vistas-proyecto)
		- [vista index para mostrar un listado de proyectos.](#vista-index-para-mostrar-un-listado-de-proyectos)
		- [vista para mostrar un Proyecto **show** asociado a todas sus tareas **listaTareas**](#vista-para-mostrar-un-proyecto-show-asociado-a-todas-sus-tareas-listatareas)
		- [vista de formulario para crear nuevo Proyecto](#vista-de-formulario-para-crear-nuevo-proyecto)
		- [vista de formulario para modificar Proyecto](#vista-de-formulario-para-modificar-proyecto)

<!-- /TOC -->

## Preparar la Aplicación

### Crear una Nueva Aplicación en Laravel

```
		laravel new admProyApp
```
si no funciona con el comando anterior intentar con **composer**.

```
		composer generate-project laravel/laravel admProyApp
```

### Agregamos la dependencia laravel-debugbar

https://github.com/barryvdh/laravel-debugbar

es una mini barra de debug que se activa sólo cuando la aplicación está en modo debug, que permite ver muchos datos necesarios a la hora de desarrollar en laravel.

para instalar

```
		composer require barryvdh/laravel-debugbar
```

editar el archivo **config/app.php**
agregar  al array de **'providers'**

```
		Barryvdh\Debugbar\ServiceProvider::class,
```

agregar al array de **'alias'**

```
		'Debugbar' => Barryvdh\Debugbar\Facade::class,
```

luego publicar los estilos

```
		php artisan vendor:publish
```
### Agregar la dependencia **laracasts/generators**
https://github.com/laracasts/Laravel-5-Generators-Extended

Con está librería vamos a agregar comandos a **artisan** para generar las migraciones de forma más simple.

```
		composer require laracasts/generators --dev
```

Registrar la librería para que funcione en el entorno de desarrollo, buscamos el archivo **app/Providers/AppServiceProvider.php** y en el método **register** agregamos

```
		if ($this->app->environment() == 'local') {
			$this->app->register(\Laracasts\Generators\GeneratorsServiceProvider::class);
		}
```

Los nuevos comandos de artisan son:
- make:migration:schema
- make:migration:pivot
- make:seed


### Habilitar la dependencia l5scaffold

https://github.com/laralib/l5scaffold

Scaffolding de Bootstrap 3 a Laravel 5
instalar el paquete l5scaffold

```
		composer require 'laralib/l5scaffold' --dev
```

Registrar el Service Provider en el archivo **config/app.php**, buscar el array
**'providers'** y agregar el elemento

```
		Laralib\L5scaffold\GeneratorsServiceProvider::class,
```

ahora se agregó un nuevo comando en artisan **make:scaffold**

### Generar una base de datos

accedemos a mysql con el siguiente comando, nos va a pedir la contraseña de usuario **root** de mysql

```
		mysql -u root -p
```

desde el shell de mysql, crear una base de datos más un usuario y asignarle los privilegios.

**base de datos: proy**  
**usuario: proy**  
**contraseña: claveSecreta**  

```
		create database proy;
		grant all privileges on proy.* to proy@localhost identified by 'claveSecreta';
		flush privileges;
		quit
```

### Modificar el archivo de configuración de entorno **.env**

```
		DB_DATABASE=proy
		DB_USERNAME=proy
		DB_PASSWORD=claveSecreta
```

### Layouts

las vistas escritas en blade deben guardarse en el directorio **resources/views/** en ese directorio vamos a generar tres subdirectorios para organizar el código de las vistas.
- **layouts**    donde guardaremos las plantillas de layouts de las páginas.
- **common**     donde ubicaremos los archivos de uso comun.
- **auth**       aquí almacenaremos las vistas relacionadas a la autentificación de ususarios.

#### plantillas comunes

generamos la plantilla de layout principal para esto creamos el archivo **resources/views/layouts/main.blade.php**

```
		<!DOCTYPE html>
		<html lang="es">
			<head>
				<meta charset="utf-8">
				<meta http-equiv="X-UA-Compatible" content="IE=edge">
				<meta name="viewport" content="width=device-width, initial-scale=1">
				<title>@yield('titulo','TITULO POR DEFECTO')</title>

				<!-- Bootstrap CSS -->
				<link href="//netdna.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css" rel="stylesheet">
				<!-- Font Awesome cientos de iconos en una sola fuente accesibles via clases de css-->
				<link href="https://maxcdn.bootstrapcdn.com/font-awesome/4.4.0/css/font-awesome.min.css" rel="stylesheet" type="text/css">
				<link href="https://bootswatch.com/readable/bootstrap.min.css" rel="stylesheet">
				{{--
				<link href="https://bootswatch.com/paper/bootstrap.min.css" rel="stylesheet">
				<link href="https://bootswatch.com/readable/bootstrap.min.css" rel="stylesheet">
				<link href="https://bootswatch.com/cerulean/bootstrap.min.css" rel="stylesheet" type="text/css">
		    <link href="https://bootswatch.com/cyborg/bootstrap.min.css" rel="stylesheet" type="text/css">
				--}}


				<!-- HTML5 Shim and Respond.js IE8 support of HTML5 elements and media queries -->
				<!-- WARNING: Respond.js doesn't work if you view the page via file:// -->
				<!--[if lt IE 9]>
					<script src="https://oss.maxcdn.com/libs/html5shiv/3.7.0/html5shiv.js"></script>
					<script src="https://oss.maxcdn.com/libs/respond.js/1.4.2/respond.min.js"></script>
				<![endif]-->

				<style>
						body {
									font-family: 'Raleway';
									margin-top: 25px;
						}

						.fa-btn {
										 margin-right: 6px;
						 }

						 .table-text div {
										 padding-top: 6px;
						 }
				 </style>

			</head>
			<body>
				<div class="container"> <!-- containerMenu -->
					<nav class="navbar navbar-default">
						<div class="container-fluid">
							<div class="navbar-header">
								<button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#navbar" aria-expanded="false" aria-controls="navbar">
									<span class="sr-only">Toggle navigation</span>
									<span class="icon-bar"></span>
									<span class="icon-bar"></span>
									<span class="icon-bar"></span>
								</button>
								<a class="navbar-brand" href="/">Admin Proyectos</a>
							</div>
							<div id="navbar" class="navbar-collapse collapse">
								<ul class="nav navbar-nav">
									&nbsp;
								</ul>

								<ul class="nav navbar-nav navbar-right">
									@if (Auth::guest())
									<li><a href="/auth/register"><i class="fa fa-btn fa-heart"></i>Registrarse</a></li>
									<li><a href="/auth/login"><i class="fa fa-btn fa-sign-in"></i>Iniciar Sesión</a></li>
									@else
									<li class="navbar-text"><i class="fa fa-btn fa-user"></i>{{ Auth::user()->name }}</li>
									<li><a href="/auth/logout"><i class="fa fa-btn fa-sign-out"></i>Cerrar Sesión</a></li>
									@endif
								</ul>
							</div>
						</div>
					</nav>
				</div><!-- /containerMenu -->
				<div class="container">
					@if(session('status'))
						<div class="alert alert-info">
							<button type="button" class="close" data-dismiss="alert" aria-hidden="true">&times;</button>
							<strong>{{ session('status') }}</strong>
						</div>
					@endif
					@section('main')
						<h1>Main</h1>
					@show
				</div>
				<!-- jQuery -->
				<script src="//code.jquery.com/jquery.js"></script>
				<!-- Bootstrap JavaScript -->
				<script src="//netdna.bootstrapcdn.com/bootstrap/3.2.0/js/bootstrap.min.js"></script>
			</body>
		</html>
```

Otro archivo Común que seguro usaremos en la mayoría de las vistas de formularios para carga de datos, es el que nos permite mostrar los mensajes de error de validaciones. Para esto generamos el archivo **resources/views/common/error.blade.php**

```
		@if (count($errors) > 0)
			<div class="alert alert-danger">
				<strong>Whoops! Algo está mal!!!</strong>
				<ul>
					@foreach ($errors->all() as $error)
						<li>{{ $error }}</li>
					@endforeach
				</ul>
			</div>
		@endif
```

## Autentificación de usuarios

### El modelo User

Por defecto laravel tiene definido un modelo denominado **User** utilizado para la autentificación de los usuarios, lo podemos encontrar en **app/User.php**, más dos migraciones  para generar las tablas correspondientes en la base de datos.

#### Migrar la Base de Datos

Una vez configurada la base de datos, podemos comenzar a interactuar desde **artisan** con la base de datos. de manera que si la base de datos está vacía podremos generar las tablas manejadas por el modelo que tenemos definido hasta este momento.

```
		php artisan migrate
```

#### Poblando la Base de Datos

Generamos un nuevo Seeder para el modelo User

```
		php artisan make:seeder UserTableSeeder
```

con el comando anterior se genera el archivo **database/seeds/UserTableSeeder.php**, en el mismo
tenemos que generar los datos agregando el siguiente código al método **run**

```
		//generamos 5 usuarios con datos
		for ($i=0; $i < 5 ; $i++) {
		  $u=new App\User();
		  $u->name='u'.$i;
		  $u->email='u'.$i.'@a.com';
		  $u->password=bcrypt('123456');
		  $u->save();
		} //for ($i=0; $i < 5 ; $i++) {
		//generamos 1 usuario más
		App\User::create(
		    [
		      'name' => 'german',
		      'email' => 'germandcorrea@gmail.com',
		      'password' => bcrypt('1234')
		    ]
		);
		//genera 10 usuarios según la definicion del modelo
		//en el archivo /database/factories/ModelFactory.php
		factory(App\User::class,10)->create();
```

luego habilitar Seeder en el archivo **/database/seeds/DatabaseSeeder.php**
agregando la línea en el método run.

```
		$this->call(UserTableSeeder::class);
```

regeneramos la base de datos con los nuevos datos de prueba. mucho cuidado con este comando por que
elimina por completo los datos de la base de datos.

```
		php artisan migrate:refresh --seed
```

#### Interactuar con los datos

luego podremos probar el funcionamiento del modelo desde la herramienta **tinker**

```
		php artisan tinker
```

ejemplos de consultas al modelo

```
		//obtiene una colección con todos los objetos usuario de la base de datos
		$ul=App\User::get();
		//retorna desde la base de datos el objeto usuario con id=1 o null sin no lo encuentra
		$u=App\User::find(1);
		//similar a la anterior pero si no encuentra genera una excepción
		$u=App\User::findOrFail(1);
		//retorna los usuarios que tienen correo terminado en @a.com
		$ul=App\User::where('email','like','%@a.com')->get();
```

### Controladores

Por otro lado también están definidos los controladores que permiten hacer inicio de Sesión en el sistema y también registrar nuevos usuarios.
Podemos ver los archivos  **App/Http/Controllers/Auth/AuthController.php** y **App/Http/Controllers/Auth/PasswordController.php**
soló es necesario agregar las rutas adecuadas para los controladores en el archivo **app/Http/routes.php**

```
		// Rutas de de autentificación
		Route::get('auth/login', 'Auth\AuthController@getLogin');
		Route::post('auth/login', 'Auth\AuthController@postLogin');
		Route::get('auth/logout', 'Auth\AuthController@getLogout');

		// Rutas para registrar Usuario
		Route::get('auth/register', 'Auth\AuthController@getRegister');
		Route::post('auth/register', 'Auth\AuthController@postRegister');
```

### Vistas

lo único que no incluye laravel por defecto son las vistas, entonces manos a la obra  a generar las vistas

#### Vistas para autentificación de usuario

por un lado es necesario el formulario de login de usuario en el archivo **resources/views/auth/login.blade.php**

```
		@extends('layouts.main')
		@section('titulo')
		Admin Proyectos - Iniciar Sesión
		@endsection
		@section('main')
		<div class="row">
		  <div class="col-md-6 col-md-offset-3">
		    <div class="panel panel-default">
		      <div class="panel-heading">
		        <h3 class="panel-title">Iniciar Sesión</h3>
		      </div>
		      <div class="panel-body">
		        <form class='form-horizontal' action="/auth/login" method="post">
		          {!! csrf_field() !!}
		          <div class="input-group">
		            <span class="input-group-addon">
		              <i class="fa fa-user fa-btn"></i>
		            </span>
		            <input type="email" name="email" id="email" value="{{ old('email') }}" class="form-control" placeholder="Usuario ..." >
		          </div>
		          <div class="input-group">
		            <span class="input-group-addon">
		              <i class="fa fa-key fa-btn"></i>
		            </span>
		            <input type="password" name="password" id="password" class="form-control" placeholder="Contraseña ..." >
		          </div>
		          <div class="checkbox">
		      			<label>
		      				<input type="checkbox" name="remember">
		      				Recordar
		      			</label>
		      		</div>
		          <div class="row">
		            &nbsp;
		          </div>
		          <button type="submit" class="btn btn-primary btn-block">Iniciar Sesión&nbsp;<i class='fa fa-sign-in fa-btn'></i></button>
		          {{-- <a href="{{url('auth/register')}}" class="btn btn-primary btn-block">Registrarse</a> --}}
		        </form>
		      </div>
		    </div>
		  </div>
		</div><!-- row -->
		@endsection

```
por otro lado es necesario el archivo con el formulario para registrar nuevos usuarios.
**resources/views/auth/register.blade.php**
```

		@extends('layouts.main')
		@section('titulo')
		Admin Proyectos - Registrarse
		@endsection
		@section('main')
		@include('common.error')
		<div class="row well well-lg">
		  <form action="" method="POST" class="form-horizontal" role="form">
						<div class="form-group">
							<legend>Registrarse</legend>
						</div>
		        {!! csrf_field() !!}
		        <div class="form-group">
		            <label for="name" class="col-sm-3 control-label">Nombre</label>
		            <div class="col-sm-9">
		              <input class="form-control" type="text" name="name" id="name" value="{{ old('name') }}">
		              <small class="text-danger">{{ $errors->first('name') }}</small>
		            </div>
		        </div>
		        <div class="form-group">
		            <label for="email" class="col-sm-3 control-label">Email</label>
		            <div class="col-sm-9">
		              <input class="form-control" type="email" name="email" id="email" value="{{ old('email') }}">
		              <small class="text-danger">{{ $errors->first('email') }}</small>
		            </div>
		        </div>

		        <div class="form-group">
		            <label for="password" class="col-sm-3 control-label">Contraseña</label>
		            <div class="col-sm-9">
		              <input class="form-control" type="password" name="password" id="password">
		              <small class="text-danger">{{ $errors->first('password') }}</small>
		            </div>
		        </div>

		        <div class="form-group">
		            <label for="password_confirmation" class="col-sm-3 control-label">Confirmar Contraseña</label>
		            <div class="col-sm-9">
		              <input class="form-control" type="password" name="password_confirmation" id="password_confirmation">
		              <small class="text-danger">{{ $errors->first('password_confirmation') }}</small>
		            </div>
		        </div>
		        <div class="row">
		          &nbsp;
		        </div>
		        <button type="submit" class="btn btn-primary btn-block">Registrar</button>
				</form>
		</div>
		@endsection
```

## modelos de Proyecto y Tareas

necesitamos crear los modelos Proyecto y Tarea para ello usaremos la librería **generators** para generar los modelos y migraciones pertinentes.

#### Generar el modelo Proyecto

el comando **make:migration:schema** tiene una convención donde indicamos el nombre de la migración **create_proyectos_table**, define cual es la operación (create) más el nombre de la tabla. y luego con la opción **--schema** definimos la estructura de la tabla. separado por comas precisamos la configuración de cada columna. por ejemplo,
 - **titulo:string:default('proyecto #1')** determina una columna de denominada **título** de tipo **string** con valor por defecto **Proyecto #1**.
 - **descripcion:text** especifica una columna denominada **descripcion** de tipo **text**.
 - **user_id:integer:unsigned:foreign** especifica una columna denominada **user_id** (según la convención el nombre debe definirse como **nombreDelmodelo_id** para que laravel sepa como hacer los **JOIN** entre las tablas cuando haga las consultas), el tipo de datos es **integer sin signo** y va a generar una clave foránea hacia la tabla del modelo **User**.

```

		php artisan make:migration:schema create_proyectos_table  --schema="titulo:string:default('Proyecto #1'),descripcion:text,user_id:integer:unsigned:foreign"

```

Siguiendo la convención de laravel donde el nombre del modelo es generado en singular y el nombre de la tabla asociada en la base de datos es en plural, el comando anterior genera dos archivos, el archivo **app/Proyecto.php** para el modelo y el archivo **database/migrations/2015_11_14_145553_create_proyectos_table.php**

#### Generar el modelo Tarea

```
		php artisan make:migration:schema create_tareas_table --schema="tarea:string,proyecto_id:integer:unsigned:foreign"
```

genera los archivos **app/Tarea.php** y el archivo **database/migrations/2015_11_14_152011_create_tareas_table.php**
#### Definir las Relaciones
en el archivo **app/User.php** agregar el método

```
		//marcamos la relación de uno a muchos entre User y Proyecto
		public function proyectos()
		{
				return $this->hasMany(Proyecto::class);
		}
```

en el archivo **app/Proyecto.php** agregar los métodos para las relaciones.

```
		//relacion pertenece a un User
		public function user()
		{
				return $this->belongsTo(User::class);
		}
		//relacion tiene muchas Tarea
		public function tareas()
		{
				return $this->hasMany(Tarea::class);
		}
```

en el archivo **app/Tarea.php** agregar los métodos para las relaciones.

```
		//relación pertenece a Proyecto
		public function proyecto()
		{
				return $this->belongsTo(Proyecto::class);
		}
```

#### Agregar un atributo al Modelo Tarea

Cada vez que modificamos la estructura de una tabla es preciso crear una nueva migración.  
Para agregar un atributo al modelo **Tarea** debemos agregar el atributo a la tabla **tareas** para ello necesitamos hacer uso de una nueva migración, mucho cuidado con el nombre de la migración ya que de acuerdo a la convención el comando sabrá que operaciones debe hacer. **add** agregar el campo **completo** **to_tareas_table** a la tabla tareas y con el esquema **--schema="completo:boolean"** el nombre del campo es **completo** y su tipo de datos **boolean**.

```
		php artisan make:migration:schema add_completo_to_tareas_table --schema="completo:boolean"
```

#### Datos de prueba

Agregamos al archivo **database/seeds/UserTableSeeder.php** en el método **run**

```
		//agregamos un usuario
		$u1=new User();
		$u1->name='user1';
		$u1->email='user1@a.com';
		$u1->password=bcrypt('123456');
		$u1->save();

		//creamos una instancia del servicio de faker para poder obtener valores aleatorios
		$f=Faker\Factory::create();
		//creamos 50 proyectos
		for ($j=0; $j < 50 ; $j++) {
			$p1= new App\Proyecto();
			$p1->titulo="proyecto $j :: ".$f->name;
			$p1->descripcion=$f->text;
			//asignamos el proyecto al usuario 1
			$u1->proyectos()->save($p1);
			//generamos entre 1 y 10 tareas para el proyecto
			for ($i=0; $i < $f->numberBetween(1,10) ; $i++) {
				$t=new App\Tarea();
				$t->tarea="Tarea : ".$f->name;
				$p1->tareas()->save($t);
			} //for ($i=0; $i < 5 ; $i++)
		} //for ($j=0; $j < 50 ; $j++)

```
ahora regeneramos la base de datos con el nuevo seeder

```

		php artisan migrate:refresh --seed

```


## Controlador de Proyectos

generamos el esqueleto del controlador con el comando de artisan **make:controller**

```
		php artisan  make:controller --plain ProyectoController

```
Luego es necesario definir el contenido de cada método del controlador, que especifica la funcionalidad de cada solicitud. para eso modificamos el archivo **app/Http/Controllers/ProyectoController.php** de modo que el contenido quede de la siguiente manera.

```
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

use App\Http\Requests;
use App\Http\Controllers\Controller;
use App\Proyecto;
use Auth;
use App\Tarea;

class ProyectoController extends Controller
{
  /**
  * Muestra todos los proyectos
  *
  * @return Response
  */
  public function index(Request $request)
  {
    $q=$request->has('buscar')?'%'.$request->buscar.'%':'%';
    /*
    //primera versión sin incluir los datos de las tareas asociadas
    $proyectos = Proyecto::where('user_id',Auth::user()->id)
                        ->orderBy('id', 'desc')
                        ->paginate(10);
    */
    //segunda versión asociando las tareas pasa de hacer 13 consultas a solo 4
    $proyectos = Proyecto::with('tareas') //obtener los objetos relacionados
                        ->where('user_id',Auth::user()->id) //solo los proyectos del usuario autenticado
                        ->Where('titulo','like',$q) //busca los que contengan en el titulo la palabra buscar
                        ->orderBy('id', 'desc') //en orden descendente por id
                        ->paginate(10); //genere la paginación

    return view('proyectos.index', compact('proyectos'));
  }

  /**
  * muestra el formulario para crear un Nuevo proyecto
  *
  * @return Response
  */
  public function create()
  {
    return view('proyectos.create');
  }

  /**
  * almacena un nuevo proyecto.
  *
  * @param Request $request
  * @return Response
  */
  public function store(Request $request)
  {
    $proyecto = new Proyecto();

    $proyecto->titulo = $request->input("titulo");
    $proyecto->descripcion = $request->input("descripcion");
    Auth::user()->proyectos()->save($proyecto);

    return redirect()->route('proyectos.index')->with('message', 'Nuevo Proyecto Guardado!!!');
  }

  /**
  * muestra un Proyecto
  *
  * @param  int  $id
  * @return Response
  */
  public function show($id)
  {
    $proyecto = Proyecto::findOrFail($id);

    return view('proyectos.show', compact('proyecto'));
  }

  /**
  * muestra el formulario para editar el Producto
  *
  * @param  int  $id
  * @return Response
  */
  public function edit($id)
  {
    $proyecto = Proyecto::findOrFail($id);

    return view('proyectos.edit', compact('proyecto'));
  }

  /**
  * Modificar el Proyecto
  *
  * @param  int  $id
  * @param Request $request
  * @return Response
  */
  public function update(Request $request, $id)
  {
    $proyecto = Proyecto::findOrFail($id);

    $proyecto->titulo = $request->input("titulo");
    $proyecto->descripcion = $request->input("descripcion");

    $proyecto->save();

    return redirect()->route('proyectos.index')->with('message', 'Proyecto Actualizado!!!');
  }

  /**
  * Elimina un Proyecto
  *
  * @param  int  $id
  * @return Response
  */
  public function destroy($id)
  {
    $proyecto = Proyecto::findOrFail($id);
    $proyecto->delete();

    return redirect()->route('proyectos.index')->with('message', 'Proyecto Eliminado!!!');
  }
  /**
  * Agrega una nueva Tarea al Proyecto
  *
  * @param  int  $id
  * @return Response
  */
  public function storeTarea(Request $request,$id)
  {
    $proyecto = Proyecto::findOrFail($id);
    $tarea = new Tarea();
    $tarea->tarea = $request->input("tarea");
    $proyecto->tareas()->save($tarea);
    return redirect()->route('proyectos.show',$id)->with('message', 'Nueva Tarea Guardada!!!');
  }

  /**
  * Elimina una Tarea
  *
  * @param  int  $id
  * @return Response
  */
  public function destroyTarea($id,$idTarea)
  {
    $tarea=Tarea::findOrFail($idTarea);
    $tarea->delete();
    return redirect()->route('proyectos.show',$id)->with('message', 'Tarea Eliminada!!!');
  }
  /**
  * actualiza una Tarea
  *
  * @param  int  $id
  * @return Response
  */
  public function updateTarea(Request $request,$id,$idTarea)
  {
    $tarea=Tarea::findOrFail($idTarea);
    $tarea->completo=$request->input('completo');
    $tarea->save();
    return redirect()->route('proyectos.show',$id)->with('message', 'Tarea Actualizada!!!');
  }
}

```

## Rutas para Proyectos

en el archivo de rutas **app/Http/routes.php** agregamos las rutas que faltan

```
//por defecto redirigir al listado de proyectos
Route::get('/', function () {
    return redirect()->route('proyectos.index');
});
//agrupamos todas las rutas que necesitan que el usuario este autenticado
Route::group(['middleware' => 'auth'], function () {
  // Rutas para el recurso proyecto básicamente las rutas que hacen ABM y listado de Proyectos
  Route::resource("proyectos","ProyectoController");
  // almacena la tarea nueva
  Route::post("proyectos/{proyectos}/tarea",['as' => 'proyectos.storeTarea','uses'=>'ProyectoController@storeTarea']);
  // elimina una tarea
  Route::delete("proyectos/{proyectos}/tarea/{idTarea}",['as' => 'proyectos.destroyTarea','uses'=>'ProyectoController@destroyTarea']);
  // modifica una tarea el verbo tiene que ser por método put
  Route::put("proyectos/{proyectos}/tarea/{idTarea}",['as' => 'proyectos.updateTarea','uses'=>'ProyectoController@updateTarea']);
}); //Route::group(['middleware' => 'auth'], function ()
```

podemos apreciar todas las rutas del sistema escribiendo el comando de artisan **route:list**


```
php artisan route:list
```

## Vistas Proyecto

ahora es necesario crear las vistas asociadas al modelo Proyecto, para ello creamos el directorio  **resources/views/proyectos** que contendrá todos los archivos de vistas del modelo proyecto.

### vista index para mostrar un listado de proyectos.

creamos el archivo de vista **resources/views/proyectos/index.blade.php**

```
		@extends('layouts.main')
		@section('main')
		<div class="page-header clearfix">
		  <h1>
		    <i class="fa fa-btn fa-align-justify"></i> Proyectos
		    <a class="btn btn-success pull-right" href="{{ route('proyectos.create') }}"><i class="fa fa-btn fa-plus"></i>Nuevo</a>
		  </h1>
		</div>
		<div class="row">
		  <div class="col-md-12">
		    <div class="search">
		      <form action="/proyectos" method="GET" class="form-horizontal">
		        <div class="form-group">
		          <label for="buscar" class="control-label col-sm-offset-1">Proyecto</label>
		          <div class="input-group col-sm-offset-1 col-sm-10">
		            <input type="text" name="buscar" id="buscar" class="form-control" value="{{ request()->buscar }}" placeholder="buscar Proyecto">
		            <span class="input-group-btn">
		              <button class="btn btn-default" type="submit"><i class="fa fa-btn fa-search"></i>buscar</button>
		            </span>
		          </div>
		        </div>
		      </form>
		    </div>
		    @if($proyectos->count())
		    <div class="list-group">
		      @foreach($proyectos as $proyecto)
		      <a href="{{ route('proyectos.show', $proyecto->id) }}" class="list-group-item">
		        <h4 class="list-group-item-heading">{{$proyecto->titulo}} <span class="badge">{{$proyecto->tareas->count()}}</span></h4>
		        <p class="list-group-item-text">{{$proyecto->descripcion}}</p>
		      </a>
		      @endforeach
		    </div>
		    <div class="text-center">
		    {!! $proyectos->render() !!}
		    </div>
		    @else
		    <h3 class="text-center alert alert-info">No Hay Proyectos!</h3>
		    @endif
		  </div>
		</div>
		@endsection

```

### vista para mostrar un Proyecto **show** asociado a todas sus tareas **listaTareas**

creamos el archivo de vista **resources/views/proyectos/show.blade.php**

```
		@extends('layouts.main')
		@section('main')
		<div class="page-header">
		  <h1>{{$proyecto->titulo}}</h1>
		  <form action="{{ route('proyectos.destroy', $proyecto->id) }}" method="POST" style="display: inline;" onsubmit="if(confirm('Estas seguro de Eliminar?')) { return true } else {return false };">
		    <input type="hidden" name="_method" value="DELETE">
		    <input type="hidden" name="_token" value="{{ csrf_token() }}">
		    <div class="btn-group pull-right" role="group" aria-label="...">
		      <a class="btn btn-warning btn-group" role="group" href="{{ route('proyectos.edit', $proyecto->id) }}" title="Editar Proyecto"><i class="fa fa-edit"></i></a>
		      <button type="submit" class="btn btn-danger" title="Eliminar Proyecto"><i class="fa fa-trash"></i></button>
		    </div>
		  </form>
		</div>
		<div class="row">
		  <div class="col-md-12">
		    <div class="form-group">
		      <label for="descripcion">DESCRIPCION</label>
		      <p class="form-control-static">{{$proyecto->descripcion}}</p>
		    </div>
		  </div>
		</div>
		@include('proyectos.listaTareas', ['tareas' => $proyecto->tareas])
		<div class="row">
		  <a class="btn btn-link" href="{{ route('proyectos.index') }}"><i class="fa fa-btn fa-backward"></i>Volver</a>
		</div>
		@endsection

```

creamos el archivo de vista **resources/views/proyectos/listaTareas.blade.php**

```
		<div class="row">
		  @if (count($tareas)>0)
		  <div class="panel panel-default">
		    <div class="panel-heading">
		      <h3>Tareas</h3>
		    </div>
		    <div class="panel-body">
		      <div class="">
		        <form class="nueva form-horizontal" action="{{ route('proyectos.storeTarea',$proyecto->id) }}" method="POST" >
		          {{ csrf_field() }}
		          <!-- tarea -->
		          <div class="form-group">
		            <label for="tarea" class="control-label col-sm-offset-1">Tarea</label>
		            <div class="input-group col-sm-offset-1 col-sm-10">
		              <input type="text" name="tarea" id="tarea" class="form-control" value="{{ old('tarea') }}" placeholder="nueva tarea">
		              <span class="input-group-btn">
		                <button class="btn btn-success" type="submit"><i class="fa fa-btn fa-plus"></i>agregar</button>
		              </span>
		            </div><!-- /input-group -->
		          </div> <!-- tarea -->
		        </form> <!-- /form.nueva -->
		      </div>
		      <table class="table table-striped">
		        <thead>
		        </thead>
		        <tbody>
		          @foreach ($tareas as $t)
		          <tr>
		            <td class="table-text">
		              <form class="modificar" action="{{route('proyectos.updateTarea',[$proyecto->id,$t->id])}}" method="POST">
		                {{ csrf_field() }}
		                {{ method_field('PUT') }}
		                <div class="checkbox">
		                  <label>
		                    <input
		                      type="checkbox"
		                      {{($t->completo==1)?'checked="checked"':''}}
		                      name="completo"
		                      value="1"
		                      onclick="$(this).parent().parent().parent().submit()"
		                      >
		                    {{ $t->tarea }}
		                  </label>
		                </div>
		              </form><!-- /form.modificar -->
		            </td>
		            <!-- eliminar -->
		            <td>
		              <form class="eliminar" action="{{route('proyectos.destroyTarea',[$proyecto->id,$t->id])}}" method="POST" onsubmit="return confirm('Está seguro de eliminar Tarea?')">
		                {{ csrf_field() }}
		                {{ method_field('DELETE') }}
		                <button type="submit" id="delete-task-{{ $t->id }}" class="btn btn-danger pull-right" title="Eliminar Tarea">
		                  <i class="fa fa-trash"></i>
		                </button>
		              </form><!-- /form.eliminar -->
		            </td>
		          </tr>
		          @endforeach
		        </tbody>
		      </table>
		    </div>
		  </div>
		  @else
		  <h3>No Hay Tareas</h3>
		  @endif
		</div>

```
### vista de formulario para crear nuevo Proyecto

creamos el archivo de vista **resources/views/proyectos/create.blade.php**

```

		@extends('layouts.main')
		@section('main')
		<div class="page-header">
		  <h1><i class="fa fa-plus"></i> Proyectos / Nuevo </h1>
		</div>
		@include('common.error')
		<div class="row">
		  <div class="col-md-12">
		    <form action="{{ route('proyectos.store') }}" method="POST">
		      <input type="hidden" name="_token" value="{{ csrf_token() }}">

		      <div class="form-group @if($errors->has('titulo')) has-error @endif">
		        <label for="titulo-field">Titulo</label>
		        <input type="text" id="titulo-field" name="titulo" class="form-control" value="{{ old("titulo") }}"/>
		        @if($errors->has("titulo"))
		        <span class="help-block">{{ $errors->first("titulo") }}</span>
		        @endif
		      </div>
		      <div class="form-group @if($errors->has('descripcion')) has-error @endif">
		        <label for="descripcion-field">Descripcion</label>
		        <textarea class="form-control" id="descripcion-field" rows="3" name="descripcion">{{ old("descripcion") }}</textarea>
		        @if($errors->has("descripcion"))
		        <span class="help-block">{{ $errors->first("descripcion") }}</span>
		        @endif
		      </div>
		      <div class="well well-sm">
		        <button type="submit" class="btn btn-primary">Crear</button>
		        <a class="btn btn-link pull-right" href="{{ route('proyectos.index') }}"><i class="fa fa-backward"></i>Atras</a>
		      </div>
		    </form>

		  </div>
		</div>
		@endsection

```
### vista de formulario para modificar Proyecto

creamos el archivo de vista **resources/views/proyectos/edit.blade.php**

```

		@extends('layouts.main')

		@section('main')
		<div class="page-header">
		  <h1><i class="fa fa-edit"></i> Proyectos / Editar #{{$proyecto->id}}</h1>
		</div>
		@include('common.error')

		<div class="row">
		  <div class="col-md-12">

		    <form action="{{ route('proyectos.update', $proyecto->id) }}" method="POST">
		      <input type="hidden" name="_method" value="PUT">
		      <input type="hidden" name="_token" value="{{ csrf_token() }}">

		      <div class="form-group @if($errors->has('titulo')) has-error @endif">
		        <label for="titulo-field">Titulo</label>
		        <input type="text" id="titulo-field" name="titulo" class="form-control" value="{{ $proyecto->titulo }}"/>
		        @if($errors->has("titulo"))
		        <span class="help-block">{{ $errors->first("titulo") }}</span>
		        @endif
		      </div>
		      <div class="form-group @if($errors->has('descripcion')) has-error @endif">
		        <label for="descripcion-field">Descripcion</label>
		        <textarea class="form-control" id="descripcion-field" rows="3" name="descripcion">{{ $proyecto->descripcion }}</textarea>
		        @if($errors->has("descripcion"))
		        <span class="help-block">{{ $errors->first("descripcion") }}</span>
		        @endif
		      </div>
		      <div class="well well-sm">
		        <button type="submit" class="btn btn-primary">Save</button>
		        <a class="btn btn-link pull-right" href="{{ route('proyectos.index') }}"><i class="fa fa-btn fa-backward"></i>  Back</a>
		      </div>
		    </form>
		  </div>
		</div>
		@endsection

```
