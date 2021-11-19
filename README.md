set ups

1. プロジェクトディレクトリを作成し、そこに移動

```
mkdir hello-prisma
cd hello-prisma
```

2. TypeScript プロジェクトを初期化し、PrismaCLI を開発依存関係としてプロジェクトに追加

```
npm init -y
npm install prisma typescript ts-node @types/node --save-dev
```

3. tsconfig.json ファイルを作成し、それに次の構成を追加

```tsconfig.json
{
  "compilerOptions": {
    "sourceMap": true,
    "outDir": "dist",
    "strict": true,
    "lib": ["esnext"],
    "esModuleInterop": true
  }
}
```

4. PrismaCLI を呼び出す

```
npx prisma
```

5. Prisma スキーマファイルを作成し、Prisma プロジェクトを設定

```
npx prisma init
```

6. env に databaseurl を記述

```.env
DATABASE_URL="postgresql://johndoe:randompassword@localhost:5432/mydb?schema=public"
```

構成は下記

```
postgresql://USER:PASSWORD@HOST:PORT/DATABASE?schema=SCHEMA
```

- USER：データベースユーザーの名前
- PASSWORD：データベースユーザーのパスワード
- PORT：データベースサーバーが実行されているポート（通常 5432 は PostgreSQL の場合）
- DATABASE：データベースの名前
- SCHEMA：データベース内のスキーマの名前

prisma では下記のようにする

```
DATABASE_URL="postgresql://janedoe:janedoe@localhost:5432/janedoe?schema=hello-prisma"
```

7. prisma に model を記述

```prisma/schema.prisma
model Post {
  id        Int      @id @default(autoincrement())
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  title     String   @db.VarChar(255)
  content   String?
  published Boolean  @default(false)
  author    User     @relation(fields: [authorId], references: [id])
  authorId  Int
}

model Profile {
  id     Int     @id @default(autoincrement())
  bio    String?
  user   User    @relation(fields: [userId], references: [id])
  userId Int     @unique
}

model User {
  id      Int      @id @default(autoincrement())
  email   String   @unique
  name    String?
  posts   Post[]
  profile Profile?
}
```

```docker-compose.yaml
version: "3.1"

services:
  db:
    image: postgres:13-alpine
    container_name: blitz-blog-postgres
    ports:
      - "5432:5432"
    restart: always
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: password
      POSTGRES_DB: testdb
    volumes:
      - dbdata:/var/lib/postgresql/data
volumes:
  dbdata:

```

データベーススキーマにマップするには、prisma migrateCLI コマンド

```
npx prisma migrate dev --name init
```

8. prisma クライアントをインストール

```
npm install @prisma/client
```

9. index.ts にコードを記述

```index.ts
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

async function main() {
  // ... you will write your Prisma Client queries   const allUsers = await prisma.user.findMany()
  console.log(allUsers)
}

main()
  .catch((e) => {
    throw e
  })
  .finally(async () => {
    await prisma.$disconnect()
  })
```

コマンド実行で関数を走らせる

```
npx ts-node index.ts
// result -> []
```

```index.ts
async function main() {
  await prisma.user.create({
    data: {
      name: 'Alice',
      email: 'alice@prisma.io',
      posts: {
        create: { title: 'Hello World' },
      },
      profile: {
        create: { bio: 'I like turtles' },
      },
    },
  })

  const allUsers = await prisma.user.findMany({
    include: {
      posts: true,
      profile: true,
    },
  })
  console.dir(allUsers, { depth: null })
}
```

下記実行でデータを作成

```
npx ts-node index.ts
```

更新する場合は下記

```index.ts
async function main() {
  const post = await prisma.post.update({
    where: { id: 1 },
    data: { published: true },
  })
  console.log(post)
}
```

## 番外編

下記実行で prisma studio がローカルに建つ
DB 内のデータをビジュアル化できる

```
npx prisma studio
```

Next.js+Graphql のフルスタック構成
https://github.com/prisma/prisma-examples/tree/latest/typescript/graphql-nextjs

Next.js+REST API のフルスタック構成
https://github.com/prisma/prisma-examples/tree/latest/typescript/rest-nextjs-api-routes

grpc でサーバー構築
https://github.com/prisma/prisma-examples/tree/latest/typescript/grpc

Apollo Server
https://github.com/prisma/prisma-examples/tree/latest/typescript/graphql
