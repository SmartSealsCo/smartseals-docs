---
title: Test
keywords: sample
permalink: guide_pgsql_replication_with_haproxy_step1_server1_db01.html
layout: default_print
---

en el archivo `/etc/postgresql/10/db01/postgresql.conf` modificaremos los siguientes campos:

>port = 5433
<br>wal_level = replica 
<br>wal_log_hints = on 
<br>archive_mode = on # (change requires restart) 
<br>archive_command = 'test ! -f /var/lib/postgresql/pg_log_archive/db01/%f && cp %p /var/lib/postgresql/pg_log_archive/db01/%f' 
<br>max_wal_senders = 10 
<br>wal_keep_segments = 64 
<br>hot_standby = on

{% include warning.html content='Esta configuración es solo para prueba, no se recomienda usarla en producción, para entender como configurar este archivo leer el articulo [postgres.conf](pgsql_config_postgres.html){:target="_blank"}' %}

En el archivo `/etc/postgresql/10/db01/pg_hba.conf` agregar al comienzo de las lineas de replication el siguiente texto:

>`local replication rep_user trust`
<br>`host replication rep_user 0.0.0.0/0 trust`

{% include warning.html content='Esta configuración es solo para prueba, no se recomienda usarla en producción, para entender como configurar esta conexión leer el articulo [pg_hba.conf](pgsql_config_hba.html){:target="_blank"}' %}

Creamos el usuario que configuramos en el archivo pg_hba que nos permitira realizar la replicacion:

>`sudo -H -u postgres psql -p 5433 -c "CREATE USER rep_user WITH replication;"`

Configuramos los replication slots:

>`sudo -H -u postgres psql -p 5433 -c "select * from pg_create_physical_replication_slot('db02');"`
<br>`sudo -H -u postgres psql -p 5433 -c "select * from pg_create_physical_replication_slot('db03');"`

