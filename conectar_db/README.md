# Conexión Base de Datos

## Creación de base de datos Postgres sobre Docker
Para trabajar con una base de datos en modo desarrollo, se recomienda crear una en Docker, para realizar esto
crear un archivo en la raiz del proyecto ***docker-compose.yml*** con la siguiente estructura

```yml
version: '3.8'

services:
    postgres-db:
        image: postgres:15.3
        restart: always
        enviroment:
            POSTGRES_USER: ${DB_USER}
            POSTGRES_DB: ${DB_NAME}
            POSTGRES_PASSWORD: ${DB_PASSWORD}
        volumes:
            - ./postgres:/var/lib/postgresql/data
        ports:
            - 5432:5432
```

Crear el archivo para las variables de entorno ***.env*** en la raiz de nuestra carpeta del proyecto, dentro de este crear
las variables de entorno que se requieren para la creación de la base de datos del proyecto

```env
DB_USER=postgres base de datos
DB_NAME=nombre base de datos
DB_PASSWORD=password
```

Abrir *Docker Desktop*, una vez abierto, ejecutar en siguiente comando en la consola, dentro de la carpeta del proyecto

```bash
docker compose up -d
```

Cuando esto termine, en la aplicación de *Docker* aparecerá la base de datos activa

Al terminar todo este proceso, agregar al archivo ***.gitignore*** el archivo ***.env*** y la carpeta de ***postgres*** que se creo, para que esto no se suba al repositorio de git

```gitignoer
.env
/postgres/
```

## Configuración de Prisma, para conexión con DB

Ver documentación de [Prisma](https://www.prisma.io/)

Instalar el ***Prisma CLI*** dentro del proyecto

```bash
npm install prisma --save-dev
```

Ahora inicializamos la configuración de Prisma ejecutando el siguiente comando en la consola, indicamos en este comando el proveedor de base de datos que será utilizado, en este ejemplo será *PostgresQL*

```bash
npx prisma init --datasource-provider PostgreSQL
```

Esto generará la variable de entorno `DATABASE_URL` dentro del archivo ***.env*** en la cual se deberá asignar la ruta de conexión a la base de datos

```env
DATABASE_URL="postgresql://[USER]:[PASSWORD]@localhost:5432/[NAME]?schema=public"
```
También el comando anterior creará una nueva carpeta ***prisma*** en la raiz del proyect y dentro de esta se encontrará el archivo ***schema.prisma***, en este archivo será donde deberemos crear los schemas de las tablas que requerimos gestionar

```prisma
generator client {
    provider = "prisma-client-js"
}

datasource db {
    provider = "postgresql"
    url = env("DATABASE_URL")
}

// Nuevos Schemas...
```

Si la base de datos a la que nos estamos conectando ya cuenta con algunas tablas creadas, ejecutar el siguiente comando para que Prisma cree los schemas de las tablas existentes en la base de datos, dentro del proyecto

```bash
prisma db pull
```

En caso que queramos desear Tablas nuevas en el proyecto, deberemos crear Schemas de las tablas que vamos a utilizar, cada que queramos migrar estos nuevos esquemas a la base de datos o modificarlos, deberemos ejecutar una migración de esto a la base de datos ejecutando el siguiente comando. En *--name* indicar el nombre que se le asignará a la migración, si se trata de la primer migración, podremos indicarla como init para identificar que es la primera, esto solo es para identificar las diferentes migraciones que realicemos al proyecto

```bash
npx prisma migrate dev --name init
```

Si realizamos un *Pull* o una *Migración* a la base de datos, posterior a esto deberemos ejecutar el siguiente comando para crear los CLientes para la gestión de las tablas creadas o modificadas dentro de la base de datos para poder gestionarlas dentro del proyecto

```bash
prisma generate
```

Crear un archivo en la siguiente ruta la siguiente ruta en la carpeta raiz del proyecto `src/lib/prisma.ts`

***prisma.ts***

```typescript
import { PrismaClient } from '@prisma/client';

// const prisma = new PrismaClient();

const prismaClientSingleton = () => {
    return new PrismaClient()
}

type PrismaClientSingleton = ReturnType<typeof prismaClientSingleton>

const globalForPrisma = globalThis as unknown as {
    prisma: PrismaClientSingleton | undefined
}

const prisma = globalForPrisma.prisma ?? prismaClientSingleton()

export default prisma

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma
```

### Schemas de Prisma

Se muestra un ejemplo de un Schema de Prisma

```prisma
enum Size {
    XL
    S
    M
    L
    XL
    XXL
    XXXL
}

enum Gender {
    men
    women
    kid
    unisex
}

model Category {
    id          String      @id @default(uuid())
    name        String      @unique

    Product     Product[]   // Esta lista se genera, ya que hay una relación con la tabla Producto
}

model Product {
    id          String      @id @default(uuid())
    title       String
    description String  
    inStock     Int
    price       Float       @default(0)
    sizes       Size[]      @default([])
    slug        String      @unique
    tags        String[]    @default([])
    gender      Gender

    category    Category    @relation(fields: [categoryId], references: [id])   // Creación de llave relacional de Categorías
    categoryId  String

    ProductImage    ProductImage[]

    @@index([gender])
}

model ProductImage {
    id          Int         @default(autoincrement())
    url         String

    product     Product     @relation(fields: [productId], references: [id])
    productId   String
}
```

