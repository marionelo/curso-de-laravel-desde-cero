Notas acerca del curso
====

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
    <php>
        <env name="APP_ENV" value="testing"/>
        <env name="CACHE_DRIVER" value="array"/>
        <env name="SESSION_DRIVER" value="array"/>
        <env name="QUEUE_DRIVER" value="sync"/>
        <env name="DB_DATABASE" value="curso_styde_tests"/>
    </php>
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
