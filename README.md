# Tarea 3

Haz un fork de este repo

NOTA: Esta tarea se puede ejecutar en el playground de docker.

https://labs.play-with-docker.com/

## Actividad 1: Mi primer docker-compose

Revisa el archivo `docker-compose.yml`.

Revisa el archivo `.env-sample`

El archivo `.env-sample` define las variables de entorno usadas en el archivo `docker-compose.yml`.

Copia el archivo `.env-sample` a `.env`, en linux esto se hace así:

        cp .env-sample .env

Edita el archivo `.env`. Cambia los valores de las variables que quieras.
Graba el archiv `.env`. (En linux puedes usar el editor `nano` o `vim`).

Ejecuta docker-compose de este modo:

        docker-compose --env-file .env-sample up -d

Ahora revisa cuantos contenedores están corriendo de este modo:

        docker ps

Ingresa a la base de datos con este comando (reemplaza los valores que corresponde a tu configuración en el archivo `.env`):

        docker-compose exec -it postgres psql -U POSTGRES_USER POSTGRES_DB

Por ejemplo, usando los valores que están en el archiv `.env-sample`, debes hacer:


        docker-compose exec -it postgres psql -U postgres usuarios


Con esto puedes ejecutar comandos en psql para revisar la base de datos:

        psql (15.4)
        Type "help" for help.

        usuarios=# select * from users;


Para salir escribe `\q`.


Deten tus contenedores con este comando:

        docker-compose down

## Actividad 2: la aplicación chatroom

Edita el archivo `docker-compose.yml` y agrega los servicios de frontend y backend:

```
  frontend:
    build: chat-frontend
    restart: always
    environment:
      - FRONT_PORT
    expose: 
      - 80
    ports:
      - ${FRONT_PORT}:80
    volumes:
      - ./nginx-conf/default.conf:/etc/nginx/conf.d/default.conf
 
  users-svc:
    build: users-svc
    command: "node app.js" 
    restart: always
    expose:
      - 3000
    environment:
      - JWT_SECRET
      - POSTGRES_USER
      - POSTGRES_PASSWORD
      - POSTGRES_DB
      - POSTGRES_PORT=5432
      - POSTGRES_SERVER=postgres
      - PORT=3000
      - CONNECTION_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DB}
    depends_on:
      - postgres
  
  ws-server:
    build: ws-server
    command: "node index.js"
    restart: always
    expose:
      - 3001
    environment:
      - PORT=3001
```

Asegurate de colocar un valor adecuado a la variable `FRONT_PORT`` en tu archivo `.env`

Levanta los contenedores con el comando:

        docker-compose up -d

Navega con browser a la dirección `localhost:FRONT_PORT`, deberias ver la pagina de login.

Registra un usuario, la aplicación debería estar corriendo correctamente.

## Preguntas

1. Revisa el archivo `Dockerfile` en la carpeta `users-svc` y compáralo con el mismo archivo en la carpeta `ws-server`. ¿Qué te llama la atención? Ahora revisa la sentencia `command` para los respectivos servicios en el archivo `docker-compose.yml`. ¿Qué concluyes?

R: Revisando los archivos 'Dockerfile" en las carpetas 'users-svs' y 'ws-server' es posible apreciar que son iguales. Dado que lo único que hace el dockerfile es generar un manifiesto de carga (crea una imagen). Estos dockerfiles no contienen la sentencia CMD en su estructura dado que son definidos directamente en el docker-compose (ejecución de múltiples contenedores).

El Dockerfile permite sólo definir y ejecutar sólo un contenedor, por eso razón para este caso de estudio se utiliza docker-compose, definiendo del archivo de configuración yml, con las repectivas relaciones con los archivos dockerfie. Archivo yml incluye los servicios (users-svs, ws-server, etc) con sus respectivos command (llamadas a las aplicaciones app.js, index.js).

2. Revisa el archivo `Dockerfile` en la carpeta `frontend`. ¿Qué te llama la atención? ¿En qué es diferente de los otros archivos `Dockerfile`?

R: 
Incluye este código adicional:

RUN npm run build

FROM nginx:stable-alpine AS build-release-stage

WORKDIR /

COPY --from=build-stage /usr/app/dist /usr/share/nginx/html

Importante:
A diferencia de los otros Dockerfile revisados anteriormente, este Dockerfile se utiliza para construir una imagen de Docker que incluye un servidor NGINX y copia los archivos generados previamente en la etapa de construcción, lo que permite servir una aplicación web estática a través de NGINX. 

La ejecución del comando npm run build se utiliza para compilar la aplicación web antes de copiar los archivos resultantes en la imagen final.

3. ¿Para qué sirve el servicio flyway? ¿Qué pasa al hacer `docker ps` con respecto a este servicio?

R: 
Flyway, en muchos casos, se ejecuta como parte del proceso de inicio de la aplicación, y cuando se inicia la aplicación, Flyway aplicará automáticamente las migraciones pendientes en la base de datos (si las hay). 
Por lo tanto, no se verá un contenedor específico de Flyway en la lista de contenedores en ejecución generada por docker ps.

Con docker ps -a, sí se ve, porque ahí es posible ver los que se estan ejecutando y los ejecutados.


4. ¿Cuantas imágenes se crean? ¿Cuántos contenedores están activos?

R:
Existen cuatro container-id (activos):
- 7e8c42d64ff5   tarea3-cna-2023-frontend    "/docker-entrypoint.…"   About an hour ago   Up About an hour   0.0.0.0:80->80/tcp   tarea3-cna-2023-frontend-1
- 0e7ee06da4bd   tarea3-cna-2023-users-svc   "docker-entrypoint.s…"   About an hour ago   Up About an hour   3000/tcp             tarea3-cna-2023-users-svc-1
- ad65eb2017b0   tarea3-cna-2023-ws-server   "docker-entrypoint.s…"   About an hour ago   Up About an hour   3001/tcp             tarea3-cna-2023-ws-server-1
- 726a9a2b9720   postgres:15-alpine          "docker-entrypoint.s…"   About an hour ago   Up About an hour   5432/tcp             tarea3-cna-2023-postgres-1

5. Deten los contenedores con `docker-compose down`, luego reinicia con `docker-compose up -d`. Ingresa a la base de datos. ¿Qué pasa con los datos? 

R:
Inicialmente:

id                  |       name       |        email        |                           password                           |  birthday  
--------------------------------------+------------------+---------------------+--------------------------------------------------------------+------------
 5c8d21dc-9881-46b5-9a31-01e47603bdd3 | Ignacio Gonzalez | igonzperz@gmail.com | $2b$10$4IrYj8PkDH6BRAL3f85MCOaGCS85GLHlxbpZnpk6rBg0CIm9X3DEa | 2000-01-01

Los datos se perdieron al bajar y reiniciar el docker-compose.

Después:

suarios=# select * from users;
 id | name | email | password | birthday 
----+------+-------+----------+----------
(0 rows)

6. Baja los contenedres. Crea un volumen para postgres agregando estas sentencias en el servicio `postgres`: 

```
 volumes:
   - ./data:/var/lib/postgresql/data
```

Reinicia los contenedores. Explica qué pasa con la base de datos después de hacer esto.

R: 
Mantiene los registros luego de bajar y volver a subir los contenedores.

                 id                  |     name      |       email       |                           password                           |  birthday  
--------------------------------------+---------------+-------------------+--------------------------------------------------------------+------------
 f75bc3aa-17ef-43a2-9af1-5c15857e9e97 | Felipe Flores | fflores@gmail.com | $2b$10$i1kePQBk5P1sOe9NBDy7b.7EJ1DAC939giGfu12KD.Wao2lXA66Iy | 2000-01-01
 76fe6cf5-52d1-4836-95c2-84df6e65aa04 | Gabriel Boric | gboric@gmail.com  | $2b$10$LzlKabdcn16tor9Jw534yeEKs0Xt/MZnbih2XmLBy.zhRA65bqcHy | 2000-01-01

¿Qué pasa con la carpeta `data`, qué crees que contiene?

R: 
Contiene los registros ingresados inicialmente. (No se pierden).

Integrantes:
- Ignacio J. González P.
- Felix Cifuentes Cid.
