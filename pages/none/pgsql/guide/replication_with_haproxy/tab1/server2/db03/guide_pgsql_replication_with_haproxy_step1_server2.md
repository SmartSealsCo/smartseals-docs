---
title: Test
keywords: sample
summary: "This is just a sample topic..."
permalink: guide_pgsql_replication_with_haproxy_step1_server2.html
layout: default_print
tab_step1_server2_db: [["DB03", guide_pgsql_replication_with_haproxy_step1_server2_db03 ], ["DB04", guide_pgsql_replication_with_haproxy_step1_server2_db04]]
---

**Paso 1**

Primero confirmamos que bases de datos existen en nuestro servidor con el comando

>`sudo pg_lsclusters`

En nuestra configuración inicial solo nos aparecera el cluster __Main__, en este ejemplo generaremos 2 clusters, db03 y db04:

>`sudo pg_createcluster 10 db03`
<br>`sudo pg_createcluster 10 db04`

Crear la siguiente carpeta de no existir:

>`sudo -H -u postgres mkdir /var/lib/postgresql/pg_log_archive`

procederemos a crear los directorios de archivadores para cada cluster

>`sudo -H -u postgres mkdir /var/lib/postgresql/pg_log_archive/db03` 
<br>`sudo -H -u postgres mkdir /var/lib/postgresql/pg_log_archive/db04`

Ahora procederemos a crear la configuracion para cada cluster:

{% include tabs.html tab_names=page.tab_step1_server2_db tab_title="" %}
