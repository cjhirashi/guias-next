# Creación de base de datos PostgresQl en docker

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

Al terminar todo este proceso, agregar al archivo ***.gitignore*** la referencia de ***.env*** y la carpeta de ***postgres*** que se creo, para que esto no se suba al repositorio de git

```gitignoer
.env
/postgres/
```