---
title: Replicación de base de datos Postgresql (Version 10) con HAProxy y Nodejs
keywords: replication postgresql haproxy node
sidebar: mydoc_sidebar
permalink: guide_pgsql_replication_with_haproxy.html
tags: [page, pgsql, haproxy, node]
tab_step1_servers: [["Servidor #1", guide_pgsql_replication_with_haproxy_step1_server1], ["Servidor #2", guide_pgsql_replication_with_haproxy_step1_server2]]
tab_step3_servers: [["Servidor #1", guide_pgsql_replication_with_haproxy_step3_server1], ["Servidor #2", guide_pgsql_replication_with_haproxy_step3_server2]]
---

{% include important.html content="Esta guia usa como administrador a NodeJS lo cual lo hace un proceso muy manual, para optimizar este proceso implementaremos una [integracion con Corosync y Pacemaker](https://www.postgresql.org/docs/10/app-pgdump.html)" %}

## Resumen

En esta guía aprenderemos a como realizar una replicación de una base de datos, para esto usaremos las tecnologías de Postgresql (Para replicar la información), HAProxy (para redireccionar a las bases de datos funcionales) y NodeJS (para administrar los procesos de la base de datos), se tendrá en cuenta que ya entiende el funcionamiento de estas tecnologías y como configurarlas, para mas información sobre estas visitar los siguientes enlaces:

+ [Replicacion por transmisión (Streaming Replication)](https://www.postgresql.org/docs/10/app-pgdump.html).
+ [Ranuras de replicación (Replication Slots)](https://www.postgresql.org/docs/10/app-pgdump.html).
+ [Replicación de recuperación con pg_rewind (Replication Failback with pg_rewind)](https://www.postgresql.org/docs/10/app-pgdump.html).
+ [Información general configuración HAProxy](https://www.postgresql.org/docs/10/app-pgdump.html).

La arquitectura de software que se va a implementar para esta guía es la siguiente:

![pgsql_replication_architecture](/images/pgsql/replication/pgsql_replication_architecture.png){:class="center_image"}

Contamos con la siguiente configuración:

+ 1 servidor en EC2 con HAProxy
+ 2 servidores en EC2:
    - Servidor 1 con ip = ip1 y 2 bases de datos
        + 1 modo Read-Write (RW) en el puerto 5432 (Master)
        + 1 modo Read-Only (RO) en el puerto 5433 (Slave)
        + 1 Servidor NodeJS para verificación de servicios en el puerto 8080

    - Servidor 2 con ip = ip2 y 2 bases de datos
        + 1 modo Read-Write (RW) en el puerto 5432 (Slave)
        + 1 modo Read-Only (RO) en el puerto 5433 (Slave)
        + 1 Servidor NodeJS para verificación de servicios en el puerto 8080

## Casos de fallo

| Caso (Al menos uno)                                                                                         | Acción                                             |
|-------------------------------------------------------------------------------------------------------------|----------------------------------------------------|
| - Base de datos DB01(Master - RW) caída <br> - Base de datos DB02(Slave - RO) caída <br> - Servidor 1 caído |  HAProxy redirecciona todas las peticiones del servidor 1 <br> al servidor 2 y NodeJS se encarga de promover DB03 a master <br> para su correcto funcionamiento |
| - Base de datos DB03(Slave - RW) caída <br> - Base de datos DB04(Slave - RO) caída <br> - Servidor 2 caído  |  HAProxy Permanece en el servidor 1 |

## **1. Configurando PostgreSQL**

{% include tabs.html tab_names=page.tab_step1_servers tab_title="" %}

## **2. Configurando HAProxy**

En nuestro servidor de NodeJS(EC2) instalaremos HAProxy y generaremos la siguiente configuracion, esta configuracion debe estar ejecutandose como servicio

    global
        maxconn 2030
 
    defaults
        log global
        mode tcp
        retries 2
        timeout client 5s
        timeout connect 3s
        timeout server 30s
        timeout check 3s
 
    listen stats
        mode http
        bind *:7000
        stats enable
        stats uri /

    frontend read_write
        bind *:5000
        acl no_db01_rw nbsrv(db01) eq 1
        acl no_db02_ro nbsrv(db02) eq 1
        use_backend db03 unless no_db01_rw no_db02_ro
        default_backend db01

    frontend read_only
        bind *:5001
        acl no_db01_rw nbsrv(db01) eq 1
        acl no_db02_ro nbsrv(db02) eq 1
        use_backend db04 unless no_db01_rw no_db02_ro
        default_backend db02

    backend db01
        mode tcp
        option httpchk GET /
        http-check expect status 200
        default-server inter 2s fall 2 rise 2 on-marked-down shutdown-sessions
        server db01 ip1:5433 check addr localhost port 5101

    backend db02
        option httpchk GET /
        http-check expect status 200
        default-server inter 2s fall 2 rise 2 on-marked-down shutdown-sessions
        server db02 ip1:5434 check addr localhost port 5102

    backend db03
        option httpchk GET /
        http-check expect status 200
        default-server inter 2s fall 2 rise 2 on-marked-down shutdown-sessions
        server db03 ip2:5433 check addr localhost port 5103

    backend db04
        option httpchk GET /
        http-check expect status 200
        default-server inter 2s fall 2 rise 2 on-marked-down shutdown-sessions
        server db04 ip2:5434 check addr localhost port 5104
        server dbo3 ip2:5433 check addr localhost port 5103 backup

    listen db01_tcp
        bind *:5101
        mode http
        server db01 ip1:8321 check

    listen db02_tcp
        bind *:5102
        mode http
        server db02 ip1:8322 check

    listen db03_tcp
        bind *:5103
        mode http
        acl no_db01_rw nbsrv(db01) eq 1
        acl no_db02_ro nbsrv(db02) eq 1
        http-request add-header custom_data db1 unless no_db01_rw no_db02_ro
        server db03 ip2:8321 check

    listen db04_tcp
        bind *:5104
        mode http
        server db04 ip2:8321 check

## **3. Configurando NodeJS en las bases de datos**

{% include tabs.html tab_names=page.tab_step2_servers tab_title="" %}