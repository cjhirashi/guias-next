# Crear un SEED con Prisma

Este nos sirve para establecer o restablecer los datos de partida de la base de datos, este será ejecutable desde la consola, ***Solo se utiliza en modo desarrollo, hay que tener cuidado de no ejecutar el seed en producción, ya que puede borrar toda la base de datos que se tenga en la aplicación para reiniciarla***

Un ejemplo de cómo lo podemos implementar:

1. Crear una carpeta llamada ***seed*** dentro de la carpeta raiz de nuestra aplicación `src/seed/`

2. Dentro de la carpeta creada, crear el siguiente archivo `src/seed/seed-database.ts`

```typescript
import { initialData } from './seed';
import prisma from '../lib/prisma';

async function main() {

    const { objects1, objects2, objects3 } = initialData;

    // Borrar registros previos
    await Promise.all([
        prisma.object1.deleteMany(),
        prisma.object2.deleteMany(),
        prisma.object3.deleteMany(),
        // todas las tablas que se quieran restablecer a 0 registros
    ]);

    // Crear registro objects1
    const objects1Data = objects1.map( (name) => ({name}) );

    await prisma.object1.createMany({
        data: objects1Data
    })

    // Crear registro objects2
    objects2.forEach( async(object) => {

        const { ...rest } = object;

        const dbObject = await prisma.object2.create({
            data: {
                ...rest
            }
        })
    })

    console.log('Seed ejecutado correctamente');
}

(() => {

    if (process.env.NODE_ENV === 'production') return; // No se permite que este archivo se ejecute en modo produccion

    main();

})();
```

3. Para que este archivo pueda ser ejecutado desde la consola, instalar la dependencia en modo desarrollo ***ts-node*** ejecutando el siguiente comando, este permite ejecutar archivos de *typescript* directamente desde *node*

```bash
npm i -D ts-node
```

4. En el archivo `package.json` del proyecto, en la parte de los *scripts* crear el siguiente *script*

```JSON
{
    "scripts": {
        "seed": "ts-node src/seed/seed-database.ts"
    }
}
```

5. Entrar a la ruta del *seed* `src/seed/` en la consola y ejecutamos el siguiente comando para crear dentro de esta el archivo de configuración `src/seed/tsconfig.json`

```bash
src/seed/> npx tsc --init
```

6. Crear el archivo `src/seed/seed.ts`, con la data que será inyectada desde el ***seed*** a la *base de datos*

```typescript
interface Object2 {
    name: string;
    description: string;
    atr1: number;
    atr2: string;
}

interface SeedData {
    objects1: string[];
    objects2: Object2[];
    objects3: string{};
}

export const initialData: SeedData = {

    objects1: [
        'name-1', 'name-2', 'name-3'
    ],
    objects2: [
        {
            name: 'name_1';
            description: 'description_1';
            atr1: 1;
            atr2: 'atr_1';
        },
        {
            name: 'name_2';
            description: 'description_2';
            atr1: 1;
            atr2: 'atr_2';
        },
        {
            name: 'name_3';
            description: 'description_3';
            atr1: 1;
            atr2: 'atr_3';
        },
        {
            name: 'name_4';
            description: 'description_4';
            atr1: 1;
            atr2: 'atr_4';
        },
    ],
    objects3: [
        'name-1', 'name-2', 'name-3'
    ],
}
```