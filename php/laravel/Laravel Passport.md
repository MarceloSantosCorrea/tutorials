# Laravel Passport

## Instalação
````php
composer require laravel/passport
````

````php
php artisan migrate
````
para adicionar as tabelas do passport ao banco de dados

Criar as chaves publicas e privadas e adicionar clientes no banco de dados
Utilizar o parâmetro `--uuids` para gerar as ID's no formato uuid

````php
php artisan passport:install --uuids
````

`database/seeders/DatabaseSeeder.php`
````php
public function run(): void
    {
        ...

        \App\Models\User::factory()->create([
            'name' => 'Marcelo Corrêa',
            'email' => 'marcelocorrea229@gmail.com',
        ]);
    }
````

`app/Models/Traits/Uuid.php`
````php
<?php

namespace App\Models\Traits;

use Illuminate\Support\Str;

trait Uuid
{
    public static function bootUuid()
    {
        static::creating(function ($model) {
            $model->{$model->getKeyName()} = $model->{$model->getKeyName()} ?: (string) Str::orderedUuid();
        });
    }
}
````

Em `app/Models/User.php`
Substituir `use Laravel\Sanctum\HasApiTokens;` por `use Laravel\Passport\HasApiTokens;`

Adicionar a trait Uuid e adicionar as propriedades para que a chave primária seja no formato string e não seja mais auto increment.
````php
class User extends Authenticatable
{
    use ..., Uuid;

    public $incrementing = false;
    protected $keyType = 'string';

    ...
}
````

`api.http`
````php
POST http://localhost/oauth/token
Accept: application/json
Content-Type: application/json

{
    "grant_type":"password",
    "client_id": "98935f1b-4288-4fcc-a609-224163743386",
    "client_secret": "HsB5eo5XnBSbVcqQ1Ec0aZKs9jSWYtEtVGNpSwy1",
    "username": "marcelocorrea229@gmail.com",
    "password": "password",
    "scope": ""
}

###
POST http://localhost/oauth/token
Accept: application/json
Content-Type: application/json

{
    "grant_type":"refresh_token",
    "client_id": "98935f1b-4288-4fcc-a609-224163743386",
    "client_secret": "HsB5eo5XnBSbVcqQ1Ec0aZKs9jSWYtEtVGNpSwy1",
    "refresh_token": "def50200870a26869d1d71717342c1db7495840ef36dcc44c2d1513e921a2a5ea93b2ef0648ca26a5a97b0b6f2faf69e849b4b2295663736778d6490bbd988f4a270c994bf60ecbd92d729391d8494e56305c00fb69edc566008af874f9ea10386531e2fd62262c83d96647fa0dd4abd0c290ec27078fcf663f5acd910d506c8f36107eb15eb5cff73defc65b1da8fe5c576534dd0eea8d48ad81568287c619ab88b23d8098e6419a01ffc5304d5f1457a8ffc5fb08f2839af2ba32b95713cf2fa67dfdcdc08158631b0d6330839f69edc9d9bb2da02fe9d56564c850f65665114d95ef522d4025632888d64b2620aeb6fe13fe31ca53d8d5c231466878fb66123d5ced1c0930e7fdd3283ad1bd23e3be614ad91f1548f12079792b7ad410310e971e3738b4ff11019f125ed115f82aed05f0f578b5656dfee5a8012322e2cba3993b1a8369d139af135f554b9ff879f1b624753a3c751caaae15bd22381a63620da8b8e97e3e026bbf371ed2cdae8405509278f2989af0845df3eae1419325c76e2bc9e0231d2da4d3746cb0d8aa9ad7b8549c666f48a4deb128609b1947f54b746e6d77a5ff830c5",
    "scope": ""
}
````