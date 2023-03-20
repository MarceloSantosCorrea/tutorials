## Prisma com NestJs

Comece instalando o Prisma CLI como uma dependência de desenvolvimento em seu projeto:
```js
npm install prisma --save-dev
```

Esse comando lista os recursos que o prisma-cli pode executar
````js
npx prisma
````

Configuração inicial do Prisma
```js
npx prisma init
```

O exemplo mostra para banco postgresql

Para SQLite, alterar o `.env` para `DATABASE_URL="file:./dev.db"`

Para Mysql pode utilizar
````js
npx prisma init --datasource-provider mysql
````