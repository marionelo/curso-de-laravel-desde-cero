Notas acerca del curso
====

Para poder hacer ejejcuacion de las pruebas unitarias es importante tener
en cuenta la instrucción de 

```sh
vendor/bin/phpunit
```

SI queremos ejecutar solo una prueba en especifico, tenemos lo siguiente

```sh
vendor/bin/phpunit --filter it_loads_the_users_details_page
```

Que ese ultimo parametro esta dentro del archivo de `tests/Feature/UsersModuleTest.php`,

```php
/** @test */
function it_loads_the_users_details_page()
{
    $this->get('/usuarios/5')
        ->assertStatus(200)
        ->assertSee('Mostrando detalle del usuario: 5');
}
```

Ahora bien nada mas para poder tener unas notas que no olvide cuando
tenemos las cosas frescas del curso sobre todo las cosas que este fulano
ejecuta

```sh
php artisan make:test UsersModuleTest
```

Lo cual crea un test en: `tests/Feature/UsersModuleTest.php`,
teniendo un ejemplo como este

```php
<?php

namespace Tests\Feature;

use Tests\TestCase;
use Illuminate\Foundation\Testing\RefreshDatabase;

class UsersModuleTest extends TestCase
{
    /** @test */
    function it_loads_the_users_list_page()
    {
        $this->get('/usuarios')
            ->assertStatus(200)
            ->assertSee('Usuarios');
    }
    
    /** @test */
    function it_loads_the_users_details_page()
    {
        $this->get('/usuarios/5')
            ->assertStatus(200)
            ->assertSee('Mostrando detalle del usuario: 5');
    }
    
    /** @test */
    function it_loads_the_new_users_page()
    {
        $this->get('/usuarios/nuevo')
            ->assertStatus(200)
            ->assertSee('Crear nuevo usuario');
    }
}

```

Database
----

Ahora bien para la base de datos tenemos que las instrucciones que podemos usar son

```sh
php artisan migrate
php artisan migrate rollback
php artisan migrate:refresh --seed

php artisan make:migration create_tablename_table
or
php artisan make:migration new_tablename_table --table=tablename
```

Para poder crear una llave foranea

```sh
php artisan make:migration add_profession_id_to_users_table
```

Para poder hacer referencia a una llave foranea, tenemos

```php
$table->foreing('profession_id')->references('id')->on('professions');
```

Ahora para el metodo down se tien que hacer un `dropForeing` y despues un `dropColumn`

Se puede usar el 

```sh
php artisan migrate:fresh
```

El cual borra primero las tablas y luego crea las migraciones
`php artisan db:seed`

Tinker
---

Para este proceso se puede ejecutar lo sigiente:

```sh
php artisan tinker
```

En donde basicamente lo que se tiene es una terminal interactiva con las clases y metodos que hay dentro del proyecto

```php
$professions = Profession::all();
$profession->first(); // Obtenemos el primer resultado
$profession->last(); // Obtenemos el último resultado
$profession->random(1); // Obtenemos un resultado aleatorio

$professions->pluck('title');
```

Crear una coleccion de forma manual
```php
collect(['Duilio', 'Styde', 'Laravel']);
```

Tambien se pueden hacer metodos para hacer queries con llamada statica

```php
    public static function findByEmail($email)
    {
        return static::where('email', $email)->first();
    }

    // la forma en la que se hace la llamada
    User::findbyEmail('email@aqui.com')
```

Pruebas unitarias con DB
-----

Para las pruebas unitarias con base de datos debemos tener en consideracion 2 cosas

* La primera es que se usan 2 bases de datos
    * Para que este cambio se pueda hacer correctamente se modifica el archivo de phpunit.xml el donde dice DB_DATABASE

```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit backupGlobals="false"
         backupStaticAttributes="false"
         bootstrap="vendor/autoload.php"
         colors="true"
         convertErrorsToExceptions="true"
         convertNoticesToExceptions="true"
         convertWarningsToExceptions="true"
         processIsolation="false"
         stopOnFailure="false">
    <testsuites>
        <testsuite name="Feature">
            <directory suffix="Test.php">./tests/Feature</directory>
        </testsuite>

        <testsuite name="Unit">
            <directory suffix="Test.php">./tests/Unit</directory>
        </testsuite>
    </testsuites>
    <filter>
        <whitelist processUncoveredFilesFromWhitelist="true">
            <directory suffix=".php">./app</directory>
        </whitelist>
    </filter>
    <php>
        <env name="APP_ENV" value="testing"/>
        <env name="CACHE_DRIVER" value="array"/>
        <env name="SESSION_DRIVER" value="array"/>
        <env name="QUEUE_DRIVER" value="sync"/>
        <env name="DB_DATABASE" value="curso_styde_tests"/> -- este es el bueno
    </php>
</phpunit>

```

ahora internamento dentro delos tests `UsersModuleTest.php` se hace uso de un trait llamado
`RefreshDatabase` el cual se encarga de recrear la base de datos
cada vez y al mismo tiempo de llenarla si es necesario

```php
<?php

namespace Tests\Feature;

use App\User;
use Illuminate\Support\Facades\DB;
use Tests\TestCase;
use Illuminate\Foundation\Testing\RefreshDatabase;

class UsersModuleTest extends TestCase
{
    use RefreshDatabase;
```

Es importante considerar que teniendo el trait implementado no es necesario eliminar la base y
crearla o casas por el estilo, si no mas bien solo en cada metodo, el trait lo hace por nosotros.

Este es un ejemplo de archivo completo de como es que se ve el .php

```php
<?php

namespace Tests\Feature;

use App\User;
use Illuminate\Support\Facades\DB;
use Tests\TestCase;
use Illuminate\Foundation\Testing\RefreshDatabase;

class UsersModuleTest extends TestCase
{
    use RefreshDatabase;

    /** @test */
    function it_shows_the_users_list()
    {
        factory(User::class)->create([
            'name' => 'Joel'
        ]);

        factory(User::class)->create([
            'name' => 'Ellie',
        ]);

        $this->get('/usuarios')
            ->assertStatus(200)
            ->assertSee('Listado de usuarios')
            ->assertSee('Joel')
            ->assertSee('Ellie');
    }

    /** @test */
    function it_shows_a_default_message_if_the_users_list_is_empty()
    {
        $this->get('/usuarios')
            ->assertStatus(200)
            ->assertSee('No hay usuarios registrados.');
    }
    
    /** @test */
    function it_loads_the_users_details_page()
    {
        $this->get('/usuarios/5')
            ->assertStatus(200)
            ->assertSee('Mostrando detalle del usuario: 5');
    }
    
    /** @test */
    function it_loads_the_new_users_page()
    {
        $this->get('/usuarios/nuevo')
            ->assertStatus(200)
            ->assertSee('Crear nuevo usuario');
    }
}

```

url usando action
---

Esto es una opcion que desconocia pero es buena para poder
agregar informacion que necesitamos a un link

```php
    <a href='{{ action("UserController@index") }}'></a>
```

Manejo de errores y status
---

El manejo de errores y status llamese como el 200, 404, 500, 301
etc, a lo cual se hace de la siguiente manera

```php
    return response()->view('nombre.de.la.vista', [], 404);
```

Siendo el ultimo parametro el codigo de respuesta del sitio en cuestión.

Que para esto sale igual cuando se hace la validacion desde el request
de laravel por ejemplo

```php
public function miFunction(User $user) {
    ...
}
```

En en el caso de cuando el usuario no existe este redirecciona a la pagina de errores, que esta en `views/errors/404.blade.php`


Creacion de usuarios con TDD
----

Esto esta en la leccion 27 para poder ver lo que se refiere, además
de que tenemos la prueba unitaria ya hecha en la leccion 27 en el
archivo `UsersModuleTests.php`

```php
/** @test */
function it_creates_a_new_user()
{
    $this->withoutExceptionHandling();

    $this->post('/usuarios/', [
        'name' => 'Duilio',
        'email' => 'duilio@styde.net',
        'password' => '123456'
    ])->assertRedirect('usuarios');

    $this->assertCredentials([
        'name' => 'Duilio',
        'email' => 'duilio@styde.net',
        'password' => '123456',
    ]);
}
```

para poder verificar lo que hay dentro de la base de datos se puede hacer con

```php
function it_creates_a_new_user()
{
    $this->post('/usuarios/', [
        'name' => 'Duilio',
        'email' => 'duilio@styde.net',
        'password' => '123456'
    ])->assertRedirect('usuarios');

    // otra manera de verifica es usando
    $this->post('/usuarios/', [
        'name' => 'Duilio',
        'email' => 'duilio@styde.net',
        'password' => '123456'
    ])->assertRedirect(route('users.index'));

    /** 
     * este metodo verifica dentro de base de datos
     * El problema es que la contraseña como va encriptada no es igual nunca
     */
    $this->assertDatabaseHas('users',[
        'name' => 'Duilio',
        'email' => 'duilio@styde.net',
    ]);

    /**
     * Para este punto se usa mejor assertCredentials para hacer la verificacion
     */
    $this->assertCredentials([
        'name' => 'Duilio',
        'email' => 'duilio@styde.net',
        'password' => '123456',
    ]);
}
    
```

Esta es otra prueba agregada para validar el nombre de usuario

```php
/** @test */
function the_name_is_required()
{
    $this->from('usuarios/nuevo')
        ->post('/usuarios/', [
            'name' => '',
            'email' => 'duilio@styde.net',
            'password' => '123456'
        ])
        ->assertRedirect('usuarios/nuevo')
        ->assertSessionHasErrors(['name' => 'El campo nombre es obligatorio']);

    $this->assertEquals(0, User::count());

//        $this->assertDatabaseMissing('users', [
//            'email' => 'duilio@styde.net',
//        ]);
}
```

Multiples validaciones Laravel TDD
-----

En este punto nosotros necsitamos poder validar un formulario completo con TDD y obtener
las diferentes respuestas necesarias para el mismo. (lesson 31) `UsersModuleTest.php`

Aqui tememos las pruebas unitarias

```php
/** @test */
function the_name_is_required()
{
    $this->from('usuarios/nuevo')
        ->post('/usuarios/', [
            'name' => '',
            'email' => 'duilio@styde.net',
            'password' => '123456'
        ])
        ->assertRedirect('usuarios/nuevo')
        ->assertSessionHasErrors(['name' => 'El campo nombre es obligatorio']);

    $this->assertEquals(0, User::count());
}

/** @test */
function the_email_is_required()
{
    $this->from('usuarios/nuevo')
        ->post('/usuarios/', [
            'name' => 'Duilio',
            'email' => '',
            'password' => '123456'
        ])
        ->assertRedirect('usuarios/nuevo')
        ->assertSessionHasErrors(['email']);

    $this->assertEquals(0, User::count());
}

/** @test */
function the_email_must_be_valid()
{
    $this->from('usuarios/nuevo')
        ->post('/usuarios/', [
            'name' => 'Duilio',
            'email' => 'correo-no-valido',
            'password' => '123456'
        ])
        ->assertRedirect('usuarios/nuevo')
        ->assertSessionHasErrors(['email']);

    $this->assertEquals(0, User::count());
}

/** @test */
function the_email_must_be_unique()
{
    factory(User::class)->create([
        'email' => 'duilio@styde.net'
    ]);

    $this->from('usuarios/nuevo')
        ->post('/usuarios/', [
            'name' => 'Duilio',
            'email' => 'duilio@styde.net',
            'password' => '123456'
        ])
        ->assertRedirect('usuarios/nuevo')
        ->assertSessionHasErrors(['email']);

    $this->assertEquals(1, User::count());
}

/** @test */
function the_password_is_required()
{
    $this->from('usuarios/nuevo')
        ->post('/usuarios/', [
            'name' => 'Duilio',
            'email' => 'duilio@styde.net',
            'password' => ''
        ])
        ->assertRedirect('usuarios/nuevo')
        ->assertSessionHasErrors(['password']);

    $this->assertEquals(0, User::count());
}

```

Aqui tenemos el como queda el controller `UserController`

```php
public function store()
{
    $data = request()->validate([
        'name' => 'required',
        'email' => ['required', 'email', 'unique:users,email'],
        'password' => 'required',
    ], [
        'name.required' => 'El campo nombre es obligatorio'
    ]);

    User::create([
        'name' => $data['name'],
        'email' => $data['email'],
        'password' => bcrypt($data['password'])
    ]);

    return redirect()->route('users.index');
}
```

Edit and update User
----

Esta es una seccion para poder editar usuarios y hacer las pruebas unitarias necesarias

```php
/** @test */
function it_loads_the_edit_user_page()
{
    $this->withoutExceptionHandling();

    $user = factory(User::class)->create();

    $this->get("/usuarios/{$user->id}/editar") // usuarios/5/editar
        ->assertStatus(200)
        ->assertViewIs('users.edit')
        ->assertSee('Editar usuario')
        ->assertViewHas('user', function ($viewUser) use ($user) {
            return $viewUser->id === $user->id;
        });
}

/**
 * Ahora se hace una prueba para actualizar un usuario
 */
/** @test */
function it_updates_a_user()
{
    $user = factory(User::class)->create();
    
    $this->withoutExceptionHandling();


    $this->put("/usuarios/{$user->id}",[
            'name' => 'Duilio',
            'email' => 'duilio@styde.net',
            'password' => '123456'
        ])->assertRedirect('usuarios');

    $this->assertCredentials([
        'name' => 'Duilio',
        'email' => 'duilio@styde.net',
        'password' => '123456',
    ]);
}
```


Validaciones para actualizar usuario
----

Estas validaciones en TDD son importantes para poder ver las respuestas que son necesarias dentro de las mismas

```php

    /** @test */
    function it_updates_a_user()
    {
        $user = factory(User::class)->create();

        $this->withoutExceptionHandling();

        $this->put("/usuarios/{$user->id}", [
            'name' => 'Duilio',
            'email' => 'duilio@styde.net',
            'password' => '123456'
        ])->assertRedirect("/usuarios/{$user->id}");

        $this->assertCredentials([
            'name' => 'Duilio',
            'email' => 'duilio@styde.net',
            'password' => '123456',
        ]);
    }

    /** @test */
    function the_name_is_required_when_updating_the_user()
    {
        $user = factory(User::class)->create();

        $this->from("usuarios/{$user->id}/editar")
            ->put("usuarios/{$user->id}", [
                'name' => '',
                'email' => 'duilio@styde.net',
                'password' => '123456'
            ])
            ->assertRedirect("usuarios/{$user->id}/editar")
            ->assertSessionHasErrors(['name']);

        $this->assertDatabaseMissing('users', ['email' => 'duilio@styde.net']);
    }

    /** @test */
    function the_email_must_be_valid_when_updating_the_user()
    {
        $user = factory(User::class)->create();

        $this->from("usuarios/{$user->id}/editar")
            ->put("usuarios/{$user->id}", [
                'name' => 'Duilio Palacios',
                'email' => 'correo-no-valido',
                'password' => '123456'
            ])
            ->assertRedirect("usuarios/{$user->id}/editar")
            ->assertSessionHasErrors(['email']);

        $this->assertDatabaseMissing('users', ['name' => 'Duilio Palacios']);
    }

    /** @test */
    function the_email_must_be_unique_when_updating_the_user()
    {
        //$this->withoutExceptionHandling();

        factory(User::class)->create([
            'email' => 'existing-email@example.com',
        ]);

        $user = factory(User::class)->create([
            'email' => 'duilio@styde.net'
        ]);

        $this->from("usuarios/{$user->id}/editar")
            ->put("usuarios/{$user->id}", [
                'name' => 'Duilio',
                'email' => 'existing-email@example.com',
                'password' => '123456'
            ])
            ->assertRedirect("usuarios/{$user->id}/editar")
            ->assertSessionHasErrors(['email']);

        //
    }

    /** @test */
    function the_users_email_can_stay_the_same_when_updating_the_user()
    {
        $user = factory(User::class)->create([
            'email' => 'duilio@styde.net'
        ]);

        $this->from("usuarios/{$user->id}/editar")
            ->put("usuarios/{$user->id}", [
                'name' => 'Duilio Palacios',
                'email' => 'duilio@styde.net',
                'password' => '12345678'
            ])
            ->assertRedirect("usuarios/{$user->id}"); // (users.show)

        $this->assertDatabaseHas('users', [
            'name' => 'Duilio Palacios',
            'email' => 'duilio@styde.net',
        ]);
    }

    /** @test */
    function the_password_is_optional_when_updating_the_user()
    {
        $oldPassword = 'CLAVE_ANTERIOR';

        $user = factory(User::class)->create([
            'password' => bcrypt($oldPassword)
        ]);

        $this->from("usuarios/{$user->id}/editar")
            ->put("usuarios/{$user->id}", [
                'name' => 'Duilio',
                'email' => 'duilio@styde.net',
                'password' => ''
            ])
            ->assertRedirect("usuarios/{$user->id}"); // (users.show)

        $this->assertCredentials([
            'name' => 'Duilio',
            'email' => 'duilio@styde.net',
            'password' => $oldPassword // VERY IMPORTANT!
        ]);
    }

    /** @test */
    function it_deletes_a_user()
    {
        $this->withoutExceptionHandling();

        $user = factory(User::class)->create();

        $this->delete("usuarios/{$user->id}")
            ->assertRedirect('usuarios');

        $this->assertDatabaseMissing('users', [
           'id' => $user->id
        ]);

        //$this->assertSame(0, User::count());
    }

```
