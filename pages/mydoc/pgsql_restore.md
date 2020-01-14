---
title: Dump y Restore de Database PostgreSQL (Version 10)
keywords: dump restore
summary: "Aprende a realizar un dump y restore en postgresql"
sidebar: mydoc_sidebar
permalink: pgsql_restore.html
simple_map: true
map_name: sts_poda_login_map
box_number: 2
folder: mydoc
tags: [page]
---

En este post se explica como hacer un backup en la base de datos de PostgreSQL con las opciones principales para hacer un correcto backup, por lo tanto, no contiene todas las funciones para generar un **Dump** y un **Restore**, la documentación completa para su revisión se encuentran en las siguientes url:

+ [pg_dump](https://www.postgresql.org/docs/10/app-pgdump.html).
+ [pg_restore](https://www.postgresql.org/docs/10/app-pgrestore.html).

## Dump

Para generar el dump de una base de datos se debe ejecutar el comando `pd_dump`, este comando se ejecuta de la forma:

>pg_dump [**connection-option**...] [**option**...] [**dbname**]
Realizar Dump y Restore en base de datos PostgreSQL
Las opciones son las que nos permiten saber como queremos generar el dump entre entras se encuentran:

- ### Requeridos
    - **d**: Nombre de la base de datos
    - **h**: Url del host de la Base de datos
    - **F**: Formato de el dump
        - **p**: Formato tipo texto plano ( Con este formato **no** se puede generar `pg_restore` )
        - **c**: Formato tipo custom (Recomendado)
        - **d**: Formato tipo directorio
        - **t**: Formato tipo tar

- ### Opcionales
    - **U**: Usuario para autorización a la base de datos
    - **W**: Solicitar contraseña para el usuario
    - **w**: **No** solicitar contraseña para el usuario
    - **n**: Esquema a realizar dump

Esto lo enviaremos a un archivo donde vamos a generar el dump (Usualmente un archivo formato **.dump**)

- ### Ejemplo

>`$ pg_dump -Fc -d [DATABASE-NAME] -n [SCHEMA-NAME] -h [DATABSE-HOST] -U [DATABASE-USER] -W > [FILE-ROUTE]`

## Restore

Para generar el restore de una base de datos se debe ejecutar el comando `pd_restore`, este comando se ejecuta de la forma:

>pg_restore [**connection-option**...] [**option**...] [**filename**]

Las opciones son las que nos permiten saber como queremos realizar el restore entre entras se encuentran:

- ### Requeridos
    - **d**: Nombre de la base de datos
    - **F**: Formato de el dump
        - **c**: Formato tipo custom (Recomendado)
        - **d**: Formato tipo directorio
        - **t**: Formato tipo tar
    - **h**: Url del host de la Base de datos

- ### Opcionales
    - **C**: Recrear la base de datos desde el dump
    - **c**: Eliminar base de datos antes de realizar el restore
        - **if-exists**: Eliminar base de datos solo si existe
        - **no-publications**: No publicar en la consola el log del restore
    - **n**: Esquema a realizar dump
    - **U**: Usuario para autorización a la base de datos
    - **W**: Solicitar contraseña para el usuario
    - **w**: **No** solicitar contraseña para el usuario

Esto lo cargaremos del archivo donde de encuentra el dump ( Usualmente un archivo formato **.dump** )

- ### Ejemplo

>`$ pg_restore -c --if-exists -Fc -d [DATABASE-NAME] -n [SCHEMA-NAME] -h [DATABSE-HOST] -U [DATABASE-USER] -W [FILE-ROUTE] --no-publications`
