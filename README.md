# Apuntes Laravel
___

Al aparecer esta definición `@method('DELETE')` se refiere a un helper que redefine el metodo por el cual es mandado una petición, en este caso se tenía: 
~~~php 
<form method="post">
  @method('DELETE')
</form>
~~~

Cuando se utiliza `@csrf` es para verificar que el formulario pertenece a este proyecto, genera un token para validarlo.

`@guest` Dentro de las vistas, se utiliza para preguntar si es un invitado haz tal cosa:

~~~php
<!-- Authentication Links -->
@guest
    ...
@else
    ...
@endguest
~~~

Variables de sesion tipo flash, esto quiere decir que estará presente una única vez, hasta que la página sea recargada:
~~~php
return back()->with('status', 'Creado con éxito');
// será retornada la vista anterior con back(), con el estatus 'creado con exito', el primer parametro 'status' será creado como una variable de tipo flash


@if (session('status')) // verifica que exista esta variable de sesion
  <div class="alert alert-success" role="alert">
      {{ session('status') }} //la imprime
  </div>
@endif
~~~

~~~php
class Post extends Model
{
    use Sluggable;

    protected $fillable = [
        'title', 'body', 'iframe', 'image', 'user_id'
    ]; // Se utiliza este atributo para dictar cuales son los campos que serán aceptados para la inserción, así laravel detecta que estos son los campos que nosotros establecimos en el front debido al uso del metodo all() el cual acepta todos los datos que vengan del formulario del front

  ...
}
~~~
### Rutas
---
- `Route::resuorce('pages', 'PageController')` Crea 7 vistas posibles: pages.index, pages.store, pages.create, pages.show, pages.update, pages.destroy, pages.edit. Las vistas necesarias para un sistema completo.
- `Route::post('posts', 'PageController@store')->name('posts.store')` El metodo name define un alias para invocar a esa ruta a traves del helper `route('posts.store')` 

### Middleware
---
A traves de esta capa se puede agregar seguridad. En Laravel, normalmente se utiliza en las rutas, definiendola de esta forma:

~~~php
Route::resource('users', 'UserController')->middleware('auth');
~~~

El ejemplo anterior hace referencia a la protección del **UserController** a traves del archivo definido en: **app/Http/Kernel**, el cual establece en su variable `$routeMiddleware` la clase a la cual será solicitada la ayuda. En este caso es `'auth' => \App\Http\Middleware\Authenticate::class`.

## Comandos
---

- `php artisan migrate ` Toma la configuración establecida en config/database/migrations, estas son tablas las cuales serán ejecutadas en nuestra base de datos (la cual está configurada en el archivo de configuración .env). Esto para que sea desplegada correctamente la base de datos en todos los entornos. ***Solo funciona si es la primera vez que se hace en un archivo***
- `php artisan migrate:refresh` Remueve las tablas y las vuelve a generar (eliminando todos los datos dentro de ellas)
- `php artisan migrate:refresh --seed` Aparte de ejecutar las migraciones, ejecuta los seeders
- `php artisan route:list ` Muestra todas las rutas que tenemos configuradas así como sus caracteristicas
- `php artisan make:controller {nombre_controlador} ` Crea un controlador, el nombre entre llaves será usado tanto como para el archivo php y la clase (por mantener un estandar en la programación)
- `php artisan make:middleware Subscribed` creación del Middleware Suscribed
- `php artisan make:controller PageController --resource` Crea los metodos que son generados con `Route::resource`
- `php artisan make:controller PageController --resource --model=Page` en adicion al comando de arriba, este genera un modelo (Page) y los vincula para que puedan trabajar en conjunto
- `php artisan make:request {nuevo_archivo_nombre}` Crea una carpeta llamada **Requests** dentro de Controllers y el archivo nuevo es colocado dentro de la nueva carpeta. Este archivo sirve para validar los requerimientos, como un email valido o una contraseña con ciertos criterios. [Reglas de validación disponibles](https://laravel.com/docs/6.x/validation#available-validation-rules)
- `php artisan make:model Post -m -f` Este comando creara el modelo **Post** junto con su archivo de factory (-f) y migration (-m). El modelo es creado en singular debido a que la consola lo convierte a plural; de Post a Posts.
- `php artisan storage:link` crea un link simbolico de storage dirigido a public, así es posible utilizar los documentos de la carpeta storage y mostrarlos al público


## Tinker 
#### php artisan tinker : Consola interna para manipular nuestros archivos de Laravel
---

- `factory(App\User::class, {numero_de_usuarios})->create() ` Crea el numero de usuarios definido utilizado la configuración dada en el archivo database/factories/UserFactory.php

## Funciones
---
- `back() ` Se usa en el controlador para regresar a la vista en la que previamente se estaba.
- `old({valor_viejo})` esta función trae el último dato que se giardó en la variable de valor_viejo, ejemplo:

  ~~~php
  <input type="text" name="email" class="form-control" placeholder="Nombre" value="{{ old('email') }}">
  ~~~
  Este ejemplo, regresa el último valor registrado en el input con el name = 'email'.
- `$errors->any()` la variable **errors** es global de laravel que registra los errores y al llamar a su método **any** le hacemos saber que queremos saber si se generó cualquier tipo de error.
- `$errors->all()` genera un **array** con todos los errores generados
- `bcrypt({valor_a_encriptar})` usando esta funcion, se puede encriptar el parametro brindado.
- `dd()` muestra el contenido de una variable y detiene la aplicación
- `all()` es un metodo definido en la clase Model para conseguir todos los datos del modelo que lo este llamando, ej:
  
  ~~~php
  $posts = Post::all();

  foreach ($posts as $post) {
      echo "$post->id : $post->title <br>";// el resultado es un objeto
  }
  ~~~
  Por otro lado podemos usar diferentes fuciones: 

  ~~~php
  $posts = Post::where('id', '>=', '20')
        ->orderBy('id', 'desc') // Ordena los resultados como la sentencia en sql
        ->take(3) // Toma solo 3 resultados de esa consulta
        ->get(); // Realiza la consulta

  foreach ($posts as $post) {
      echo "$post->id : $post->title <br>";
  }
  ~~~
- `$users->contains('email', null, 'cleve.robel@example.net')` verifica si existe el correo en la key **email** del array de `$users`. Si se coloca solo así `$users->contains(5)` verificará automaticamente si se encuentra el 5 en la key **id**
- `$users->except([1,2,3]);` Filtra el array, excluyendo los id inpuestos, en este caso `[1,2,3]`
- `$users->only([1,2,3]);` Filtra el array, obteniendo solo los id inpuestos, en este caso `[1,2,3]`
- `$users->find(5, 'error: give me chetos')` Busca en el array el 5 en la key **id** y en caso de no encontrarla, devuelve "error: give me chetos"
- `$users->load('posts')` Utiliza la cadena de usuarios pero le añade la propiedad de "relations", donde se encuentran los posts asociados a ese usuario.
- `User::with('Posts')` Metodo propio de Eloquent el cual, es parecido a `$users->load('posts')` solo que este genera la union de los datos en la BBDD y despues los devuelve.
- `$users->toArray()` convierte el objeto a array
- `$users->toJson()` convierte un objeto a un json se le llama **Serialización**
- `format('d M Y')` se utiliza para darle formato a una fecha
- `Post::with('user')->latest()->paginate()` En este ejemplo se están obteniendo los resultados de la tabla post con su respectivo usuario que los creó en una variable `$posts`. La función `latest()` ordena desde el archivo más nuevo al más viejo y puede cambiarse por `oldest()` que funciona a la inversa, despues se observa el metodo `paginate()` el cual dará una vista de paginación, pero debe hacerse en conjunto con `$posts->links()` que son el número de páginas creadas que el pagindor necesita
- `$request->file('file')->store('posts', 'public');` guardará un archivo con el nombre que otorge `$request->file('file')` y lo guardará en la carpeta public/posts
- `Storage::disk('public')->delete($post->image);` Esta función hace uso de la clase definida en `Illuminate\Support\Facades\Storage;` la cual permite manipular archivos dentro del proyecto, en este caso dentro de la carpeta public borrará la ruta que llega de la base.

## Blade: sistema de plantillas avanzado
---
- `@extends('app')` Esto quiere decir que utilizará lo escrito en el archivo app.blade.php
- `@section('content')` - `@endsection` Todo lo marcado entre estos términos, podrán ser usados en cualquier otro archivo usando `@yield('content')` (entre el parentesis de estos dos, está el nombre al que harán referencia) pero no sin antes utilizar el metodo `@extends` haciendo referencia al archivo que llama a yield.
- `@include('header')` Como php, lo que hace, es escribir exactamente lo que incluye el archivo, en este caso **"header"**

## Componente: Laravel/UI
---
Para instalarlo usamos composer `composer require laravel/ui --dev`.

Al realizar la instalación, se agregan nuevos comandos en laravel, el cual se puede utilizar de distintas maneras, la sencilla es esta `php artisan ui:auth` la cual genera el sistema de autenticación pero sin estilos de página, los siguientes comandos les darán sus respectivos estilos:
- `php artisan ui bootstrap --auth` 
- `php artisan ui vue --auth`
- `php artisan ui react --auth`

Este comando crea la ruta, el controlador y varias vistas que son necesarias para el sistema de LogIn o Registro. Al principio no se verán con estilos las vistas, debemos correr los comandos que el propio resultado anterior nos indica, en el ejemplo con bootstrap, nos sugiere utilizar estos dos: ***npm install && npm run dev***. Lo que hacen estos comando es compilar los archivos css y js para que funcionen en el front, por ejemplo un archivo sass a css

## Eloquent: ORM
---
Es la forma de interpretar sentencias SQL a traves del sistema de laravel. Todos los archivos configurados en migrate son hechos con el fin, de al momento de utilizar el comando `php artisan migrate` sean generadas esas tablas en nuestra base de datos.

Para crear una tabla se utiliza el siguiente metodo, el cual esta definido en su archivo de migración:

~~~php
public function up()
{
  Schema::create('posts', function (Blueprint $table) {
    $table->bigIncrements('id'); // big int con incremento automatico

    $table->unsignedBigInteger('user_id'); // big int sin signo

    $table->string('title'); // campo title definido como string

    $table->string('slug')->unique(); // campo que si es repetido se añade un indice al final

    $table->foreign('user_id')->references('id')->on('users'); // Se añade la llave foranea de user_id

    $table->timestamps(); // campos de fechas created_at y updated_at
  });
}
~~~

Al colocar de esta manera una llave foranea se le hace saber a la base de datos, pero no a laravel. La forma de hacerselo saber es declarando en sus respectivos modelos de Usuario a Post:

~~~php
class User extends Authenticatable
{

  // ...

  public function posts()
  {
    return $this->hasMany(Post::class); // Este metodo hace saber que Usuario puede tener muchos Posts (se define en plural el metodo para mejor lejibilidad)
  }
}
~~~
~~~php
class Post extends Model
{
  // ...
  public function user() 
  { 
    return $this->belongsTo(User::class); // Este metodo indica que un Post pertenece a un Usuario por esa razon el metodo user() se define en singular
  }
}
~~~

Así pueden ser usados de estas formas: 

~~~php
Route::get('eloquent-relationship', function () {
  $posts = Post::get();

  foreach ($posts as $post) {
      echo "$post->id : {$post->user->name} : $post->title <br>"; //$post->user->name se llama al nombre del usuario referenciado en la tabla post
  }
});

Use App\User;
Route::get('users-relationship', function () {
  $users = User::get();

  foreach ($users as $user) {
      echo "$user->id : $user->name : {$user->posts->count()} <br>"; // $user->posts->count() se cuentan los posts de un usuario
  }
});
~~~

Es posible crear nuevos atributos de una clase, por ejemplo la de **User** utillizando esta convención:

~~~php
// Nuevo atributo de User upper_name (no está definmido como columna en la BBDD)
$users = User::get();

foreach ($users as $user) {
  echo "$user->upper_name <br>"; // El objetivo es regresar el atributo name pero en mayusculas
}
~~~

~~~php
// Es definido el metodo que creará el atributo en la clase User

// todo método que se use para esto, debe comenzar con "get" y terminar con "Attribute", el nombre del nuevo atributo será definido por lo que exista en medio de estas dos palabras, en este caso "UpperName" será transformado al atributo "upper_name"
public function getUpperNameAttribute() 
{
  return strtoupper($this->name); // Aqui dentro serán manipulados los datos como sea conveniente
}
~~~

Es posible configurar la manera en que se salvan los datos, esto con el fin de mantener un estandar en la base de datos. Básicamente es el mismo procedimiento de arriba sobre la creacion de nuevos atributos.

~~~php
// Será definida de una forma similar con la excepción de que ahora será comenzado por set, como no queremos crear un nuevo atributo, si no, rehacer uno que ya existe, sera puesto ese nombre, en este caso en medio de las palabras clave (set y Attribute) se colocó Name, el cual es un atributo existente de esta clase, aspi que será sobreescrito.

// Recibe la variable $value la cual es el valor que será insertado a la base de datos
public function setNameAttribute($value)
{
  $this->attributes['name'] = strtolower($value); // esta sentencia dicta que a los atributos de esta clase en su posicion "name" será colocado el valor proveniente pero todo en minusculas
}
~~~
## Seeders
---
Archivo que permite hacer uso de los factories para rellenar las tablas.