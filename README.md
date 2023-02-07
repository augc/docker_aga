# docker_aga
Despliega un proyecto de docker que contendrá aplicación AGA funcionando con Symfony 6

## Introducción

Este proyecto pone en marcha 2 contenedores con todo lo necesario para crear un nuevo proyecto de Symfony 6. 

Los contenedores son los siguientes:

- `nginx:latest`
- `php:8.2-fpm`

**nginx** y **php** son creados a partir de las imágenes definidas en los Dockerfile's `Dockerfile-ngnix` y `Dockerfile-php` que les añade Composer y los módulos de php necesarios para Symfony6. Después el fichero `docker-composer.yaml` crea la red y los volúmenes necesarios para que todo funcione.

## Pasos previos

Lo primero es **crear un directorio** en la máquina local o remota desde donde va a correr la aplicación. Por ejemplo: `/home/user/my-project`.

        mkdir my_project

A continuación nos situamos en dicho directorio y **descargamos** (hacemos clone) desde github, el contenido de este repositorio.

	cd my_project
        git clone https://github.com/augc/docker_aga.git .
        
### Crea el archivo y los directorios siguientes:

Renombra el archivo .env_dummy como .env y actualiza las variables de entorno que se utilizarán en el proyecto

`/home/user/my-project/.env`

        PATH_TO_PROJECT=full-path-to-project-in-local # /home/user/my-project/
        APP_NAME=augc_aga
        POSTGRES_USER=?
        POSTGRES_PASSWORD=?
        POSTGRES_DB=?
        POSTGRES_HOST=?
        POSTGRES_VERSION=?
        SYMFONY_VERSION=6.2
        STABILITY=stable
        
### Crea los siguientes directorios

        mkdir php_socket
        
Verifica que tienes la siguiente estructura de archivos: 

        .
        ├── Dockerfile-nginx
        ├── Dockerfile-php
        ├── build
        │   └── nginx
        │       └── default.conf
        │       └── nginx.conf
        ├── php_socket
        └── docker-compose.yml
        
`php_socket` no está en este repositorio porque es el directorio mapeado con el directorio de trabajo del contenedor. Dentro de `php_socket` tendrás otro repositorio independiente que contendrá el proyecto en Symfony de la aplicación aga.

El proyecto de Symfony se creará y ejecutará en el contenedor de php. El directorio de trabajo en dicho contenedor (WORKDIR) es `/var/www/symfony/augc_aga`

Todo el proyecto se encuentra mapeado en nuestro directorio local `./php_socket` (proyecto symfony)

## Orquestando los contenedores

### Construir las imágenes y lanzar los contenedores:

Vamos a crear las imágenes definidas en Dockerfile-ngnix y Dockerfile-php (sólo tienes que hacerlo la primera vez):

        docker compose build

Si todo ha ido bien, sin mensajes de error, se habrán creado las imágenes symfony-aga_nginx y symfony-aga_php.

Ahora podemos lanzar los contenedores:

        docker compose up -d

Docker-compose crea una red interna con IP fijas para los 4 contenedores:

        augc_aga_php        172.20.0.2
        augc_aga_nginx      172.20.0.3
        
Es importante que las IP sean fijas para evitar tener que modificar el archivo de configuración con los datos de conexión a la base de datos.

## Preparando tu entorno de Symfony 6:

Si todo ha ido bien tendremos a los cuatro contenedores funcionando:

        docker ps --format "table {{.ID}}\t{{.Status}}\t{{.Ports}}\t{{.Names}}"

        CONTAINER ID   STATUS          PORTS                                         NAMES
        1d34071293d7   Up 54 minutes   0.0.0.0:4444->80/tcp, 0.0.0.0:4443->443/tcp   augc_aga_nginx
        9c92f8ceb3b7   Up 54 minutes   9000/tcp                                      augc_aga_php

Accedemos al contenedor `my-project_php`

        docker exec -it augc_aga_php bash
        root@3fa9b676624c:/var/www/symfony#
        
Configurar los datos del usuario que vayas a utilizar y tenga permisos de edición en el repositorio de AGA (el comando `symfony new` crea un repositorio, con el código de nuestro proyecto de symfony)

        git config --global user.email "you@example.com"
        git config --global user.name "Your Name"
        git config --global credential.helper 'cache --timeout 3600'

## Clonar el repositorio que contiene AGA

Clonamos

        root@3fa9b676624c:/var/www/symfony# git clone https://github.com/augc/aga.git augc_aga
        
Accede al directorio

        root@3fa9b676624c:/var/www/symfony# cd augc_aga

Instala las dependencias

        root@3fa9b676624c:/var/www/symfony/augc_aga# composer install (o composer install --no-dev --optimize-autoloader si estás en producción)

# Fin

Si no has cambiado el valor de los puertos en `docker-compose.yml` tendrás la aplicación funcionando en http://localhost:4444/

**¡Ya tienes tu proyecto Symfony funcionando!**
