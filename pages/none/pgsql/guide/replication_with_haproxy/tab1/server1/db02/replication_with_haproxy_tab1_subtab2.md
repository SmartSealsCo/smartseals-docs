---
title: Test
keywords: sample
permalink: guide_pgsql_replication_with_haproxy_step1_server1_db02.html
layout: default_print
---

en el archivo `/etc/postgresql/10/db02/postgresql.conf` modificaremos los siguientes campos

>port = 5434
<br>wal_level = replica 
<br>wal_log_hints = on 
<br>archive_mode = on # (change requires restart) 
<br>archive_command = 'test ! -f /var/lib/postgresql/pg_log_archive/db02/%f && cp %p /var/lib/postgresql/pg_log_archive/db02/%f' 
<br>max_wal_senders = 10 
<br>wal_keep_segments = 64 
<br>hot_standby = on

{% include warning.html content='Esta configuración es solo para prueba, no se recomienda usarla en producción, para entender como configurar este archivo leer el articulo [postgres.conf](pgsql_config_postgres.html){:target="_blank"}' %}

En el archivo `/etc/postgresql/10/db02/pg_hba.conf` agregar al comienzo de las lineas de replication el siguiente texto:

>`local replication rep_user trust`

{% include warning.html content='Esta configuración es solo para prueba, no se recomienda usarla en producción, para entender como configurar esta conexión leer el articulo [pg_hba.conf](pgsql_config_hba.html){:target="_blank"}' %}

Eliminamos los archivos de bases de datos anteriores, esto debido a que necesitamos una base de datos completamente limpia

>`sudo -H -u postgres rm -rf /var/lib/postgresql/10/db02/*` 

Procedemos a ejecutar la sincronización de los datos de las bases de datos

>`sudo -H -u postgres pg_basebackup -D /var/lib/postgresql/10/db02 -p 5433 -U rep_user -w -P -R`

Ahora procedemos a configurar el archivo de recuperación `/var/lib/postgresql/10/db02/recovery.conf`

>restore_command = 'cp /var/lib/postgresql/pg_log_archive/db02/%f %p' 
<br>recovery_target_timeline = 'latest'
<br>primary_slot_name = 'db02'
