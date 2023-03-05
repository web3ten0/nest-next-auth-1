# nest-next-auth-1

NestJS + Next.js + NextAuth + Prisma + SQLite

## NestJS

### Create App

```
npm i -g @nestjs/cli
nest new backend
```

### Setting Prisma

```
npm i -D prisma
npx prisma init --datasource-provider sqlite
```

### Create Model

`backend/prisma/schema.prisma` にモデルを追加します。今回は認証を試すので User モデルを作成します。

```
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  password  String
  name      String
  profile   String?
  url       String?
  active    Boolean  @default(true)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

尚、VSCode で schema.prisma のフォーマットを有効にするには、[Prisma](https://marketplace.visualstudio.com/items?itemName=Prisma.prisma)という拡張をインストールしたら出来ました。

### Migrate

```
npx prisma migrate dev --name init
```

- SQLite の中身を VSCode で確認するには、[Database Client](https://marketplace.visualstudio.com/items?itemName=cweijan.vscode-database-client2)という拡張を使うと出来ました。
- SQLite の中身を CLI で確認するには、sqlite をインストールして使います。下記は ArchLinux の場合です。

```
sudo pacman -S sqlite
sqlite3 backend/prisma/dev.db
sqlite> .tables
sqlite> .schema User
sqlite> .quit
```

### nestjs-prisma を入れる

- [notiz-dev/nestjs-prisma](https://github.com/notiz-dev/nestjs-prisma)

```
nest add nestjs-prisma
```

- backend/src/app.module.ts` に PrismaModule を追加

```
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { PrismaModule } from 'nestjs-prisma';

@Module({
  imports: [PrismaModule.forRoot()],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

### Setting GraphQL(code first)

- graphql 関連をインストール

```
npm i @nestjs/graphql @nestjs/apollo graphql apollo-server-express
```

- backend/src/app.module.ts` に GraphQLModule を追加

```
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { PrismaModule } from 'nestjs-prisma';
import { GraphQLModule } from '@nestjs/graphql';
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';
import { join } from 'path';

@Module({
  imports: [
    PrismaModule.forRoot(),
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      autoSchemaFile: join(process.cwd(), 'src/schema.gql'),
      sortSchema: true,
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

- `nest-cli.json`に下記を追記して、GraphQL の CLI プラグインを有効にしてデコレータの簡潔な記法を可能にします。

```
{
  "collection": "@nestjs/schematics",
  "sourceRoot": "src",
  "compilerOptions": {
    "plugins": ["@nestjs/graphql"]
  }
}
```

### User リソースを Nest CLI で作成

```
nest g resource users
```

### GraphQL プレイグラウンド

- http://localhost:3000/graphql

### User のモデルを修正

`nest g resource` で生成した `users` の `DTO` と `Entity` を `schema.prisma`に合わせて修正します。

- backend/src/users/entities/user.entity.ts

```
import { ObjectType, Field, Int } from '@nestjs/graphql';

@ObjectType()
export class User {
  @Field(() => Int)
  id: number;
  email: string;
  password: string;
  name: string;
  profile?: string;
  url?: string;
  active: boolean;
  createdAt: Date;
  updatedAt: Date;
}
```

- backend/src/users/dto/create-user.input.ts

```
import { InputType } from '@nestjs/graphql';

@InputType()
export class CreateUserInput {
  email: string;
  password: string;
  name: string;
  profile?: string;
  url?: string;
  active: boolean;
}
```

### UserService を修正

- users.service に prisma を注入し SQLite の操作を可能にします。

## Next.js

### Create App

```
npx create-next-app@latest --typescript
```
