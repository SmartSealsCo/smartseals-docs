---
title: Test
keywords: sample
summary: "This is just a sample topic..."
permalink: guide_pgsql_replication_with_haproxy_step1_server1.html
layout: default_print
tab_step1_server1_db: [["DB01", guide_pgsql_replication_with_haproxy_step1_server1_db01 ], ["DB02", guide_pgsql_replication_with_haproxy_step1_server1_db02]]
---

**Paso 1**

Primero confirmamos que bases de datos existen en nuestro servidor con el comando

>`sudo pg_lsclusters`

En nuestra configuración inicial solo nos aparecera el cluster __Main__, en este ejemplo generaremos 2 clusters, db01 y db02:

>`sudo pg_createcluster 10 db01`
<br>`sudo pg_createcluster 10 db02`

Crear la siguiente carpeta de no existir:

>`sudo -H -u postgres mkdir /var/lib/postgresql/pg_log_archive`

procederemos a crear los directorios de archivadores para cada cluster

>`sudo -H -u postgres mkdir /var/lib/postgresql/pg_log_archive/db01` 
<br>`sudo -H -u postgres mkdir /var/lib/postgresql/pg_log_archive/db02`

Ahora procederemos a crear la configuracion para cada cluster:

{% include tabs.html tab_names=page.tab_step1_server1_db tab_title="" %}
