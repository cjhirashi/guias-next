# Gestión de Bases de datos

## Configuración de Prisma

Ver documentación de [Prisma](https://www.prisma.io/)

1. Instalar el ***Prisma CLI*** dentro del proyecto

```bash
npm install prisma --save-dev
```

2. Ahora inicializamos la configuración de Prisma ejecutando el siguiente comando en la consola, indicamos en este comando el proveedor de base de datos que será utilizado, en este ejemplo será *PostgresQL*

```bash
npx prisma init --datasource-provider PostgreSQL
```

3. Esto creará un achivo en la siguiente ruta `prisma/schema.prisma`, en este archivo será donde deberemos crear los schemas de las tablas que requerimos gestionar

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

4. Esto generará también la variable de entorno `DATABASE_URL` dentro del archivo `.env` en la cual se deberá asignar la ruta de conexión a la base de datos

```env
DATABASE_URL="postgresql://[USER]:[PASSWORD]@localhost:5432/[NAME]?schema=public"
```

5. Si la base de datos a la que nos estamos conectando ya cuenta con algunas tablas creadas, ejecutar el siguiente comando para que Prisma jale al proyecto la imágen de los schemas de las tablas existentes en la base de datos

```bash
prisma db pull
```

6. En caso que queramos crear Tablas nuevas en el proyecto, deberemos crear *Schemas* de las tablas que vamos a utilizar, cada que queramos migrar estos nuevos esquemas a la base de datos o modificarlos, deberemos ejecutar una migración de esto a la base de datos ejecutando el siguiente comando. En `--name` indicar el nombre que se le asignará a la migración, si se trata de la primer migración, podremos indicarla como `init` para identificar que es la primera, esto solo es para identificar las diferentes migraciones que realicemos al proyecto

```bash
npx prisma migrate dev --name init
```

7. Si realizamos un `pull` o un `migrate` a la base de datos, tendremos que ejecutar el siguiente comando para crear los CLientes para la gestión de las tablas creadas o modificadas dentro de la base de datos para poder gestionarlas dentro del proyecto

```bash
prisma generate
```

8. Crear un archivo en la siguiente ruta de la carpeta raiz del proyecto `src/lib/prisma.ts` para gestionar a los clientes de tablas de ***Prisma***

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

## Ejemplo de Schemas en Prisma

Se muestra un ejemplo de de Schemas con puntos relacionales

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

