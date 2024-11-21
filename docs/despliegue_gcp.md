# Despliegue

`./google-cloud-sdk/bin/gcloud init`

Listar configuración del proyecto

`./google-cloud-sdk/bin/gcloud config list`


- Adiciona el archivo .gcloudignore

- Adicionamos el archivo app.yaml
`runtime: java21`
`instance_class: F2`

Paso siguiente ejecutamos `gcloud app deploy`

Nos retorna la url

`https://famous-cursor-442219-q6.rj.r.appspot.com`

Despues que funcione todo

Colocar la seguridad

1ro Es eliminar la ip publica

cloud SQL Admin API

y le damos habliitar 

Ya funciona pero hemos dejado la aplicacion expuesta, entonces debemos hacer unos ajustes

Despues vamos a spring gcp y nos traemos la dependencia 

Despues vamos al archivo application.properties y comentamos la url

Vamos a cerrar las conexiones para que sea seguro.

Despues buscamos cloud sql admin api

luego spring boot tiene un conjunto de herramientas para comunicarce con otras plataformas. en el caso de spring cloud gcp

Adicionamos la dependencia 

`implementation("com.google.cloud:spring-cloud-gcp-starter-sql-postgresql")`

Ahora vamos a cambiar la conexion entonces buscamos cadena de conexion 

Adicionamos dos propiedades 

spring.cloud.gcp.sql.database-name=kotlindb
spring.cloud.gcp.sql.instance-connection-name=famous-cursor-442219-q6:us-central1:kotlinbase

y despues ejecutamos con gcloud app deploy


Vamos a IAM Google cloud 

despues ejecutamos el siguiente comando `gcloud auth application-deafult login`

Despues ejecutamos kotlin de manera local y podemos acceder a la base de datos


