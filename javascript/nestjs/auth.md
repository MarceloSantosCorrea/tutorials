## Authentication && Authorization

Pacotes utilizados
- NestJs
- MySQL com Prisma
- JWT
- Docker (MySQL)

Criando projeto
````js
nest new nestjs-jwt-auth
cd nestjs-jwt-auth
````

Instalando Prisma
````js
npm install prisma --save-dev
````
Instalando o Prisma gerador cliente
````js
npm install @prisma/client
````
Iniciando Prisma com MySQL
````js
npx prisma init --datasource-provider mysql
````
Exemplo arquivo `./prisma/scheme.prisma`
```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}

model User {
  id       String @id @default(uuid())
  email    String @unique
  name     String
  password String

  @@map("users")
}
```
Exemplo arquivo `./.env`
```.dotenv
DB_HOST=localhost
DB_PORT=3306
DB_DATABASE=nest_auth
DB_USERNAME=nestjs
DB_PASSWORD=password

DATABASE_URL="mysql://${DB_USERNAME}:${DB_PASSWORD}@${DB_HOST}:${DB_PORT}/${DB_DATABASE}"
```

Criar arquivo `docker-compose.yaml`
```yaml
version: '3'
services:
  mysql:
    image: 'mysql/mysql-server:8.0'
    ports:
      - '${FORWARD_DB_PORT:-3306}:3306'
    environment:
      MYSQL_ROOT_PASSWORD: '${DB_PASSWORD}'
      MYSQL_ROOT_HOST: "%"
      MYSQL_DATABASE: '${DB_DATABASE}'
      MYSQL_USER: '${DB_USERNAME}'
      MYSQL_PASSWORD: '${DB_PASSWORD}'
      MYSQL_ALLOW_EMPTY_PASSWORD: 1
```

Rodar comando `docker-compose up -d`

Alterar no arquivo `./.env` a variável `DB_USERNAME` para `root`

Rodar o arquivo de migration para a criação das tabelas no Banco de dados
```js
npx prisma migrate dev --name init
```
Criar arquivo `src/prisma.service.ts`
```js
import { INestApplication, Injectable, OnModuleInit } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit {
  async onModuleInit() {
    await this.$connect();
  }

  async enableShutdownHooks(app: INestApplication) {
    this.$on('beforeExit', async () => {
      await app.close();
    });
  }
}
```
Criar módulo de usuários
```js
nest g resource users
```
Escolha as opções
```
What transport layer do you use? > REST API
Would you like to generate CRUD entry points? Y
```
Colocar os campos recebidos na request no arquivo `src/users/dto/create-user.dto.ts`
```ts
export class CreateUserDto {
  name: string;
  email: string;
  password: string;
}
```

Alterar arquivo `src/users/user.service.ts`
```ts
@Injectable()
import { PrismaService } from 'src/prisma.service';

export class UsersService {
  constructor(private prisma: PrismaService) {}

  create(data: CreateUserDto) {
    return this.prisma.user.create({
      data,
    });
  }

  findAll() {
    return this.prisma.user.findMany();
  }

  ...
}
```
Adicionar `PrismaService` no arquivo `users/users.module.ts`
```ts
import { PrismaService } from 'src/prisma.service';

@Module({
  ...,
  providers: [UsersService, PrismaService],
  ...,
})
```
Criando usuário para teste
Criar arquivo `api.http`
````http request
POST http://localhost:3000/users
Accept: application/json
Content-Type: application/json

{
  "name": "Usuário Teste",
  "email": "email@test.com",
  "password": "password"
}
````
Resposta do servidor
```http request
HTTP/1.1 201 Created
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Content-Length: 116
ETag: W/"74-N8q0vn8mo7fz4SALbsErTlpT6ko"
Date: Wed, 22 Feb 2023 20:35:23 GMT
Connection: close

{
  "id": "0fb3e6dd-17a4-4a88-8224-f48275a493c4",
  "email": "email@test.com",
  "name": "Usuário Teste",
  "password": "password"
}
```
Listar todos os usuários
adicionar em `api.http`
```http request
###
GET  http://localhost:3000/users
Accept: application/json

```
Resposta do servidor
```http request
HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Content-Length: 118
ETag: W/"76-dOBXUbVoXVbMpf7JdKii/tG1tQo"
Date: Wed, 22 Feb 2023 20:37:18 GMT
Connection: close

[
  {
    "id": "0fb3e6dd-17a4-4a88-8224-f48275a493c4",
    "email": "email@test.com",
    "name": "Usuário Teste",
    "password": "password"
  }
]
```

