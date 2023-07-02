# Definicion
## quickstart-23.0.0.Final
Enrutador
## API Gateway
API Gateway (Puerta de enlace de API) es un componente de software clave  en la arquitectura de microservicios, proporciona una capa que simplifica  la comunicación y la integración en un entorno distribuido de servicios y clientes que se encuentran detrás de el.
Es un punto de entrada único para todas las solicitudes de API entrantes, se encarga de realizar el enrutamiento.
Facilita la comunicación y la interoperatibilidad en un entorno distribuido de servicios.
## Nginx
Es un servidor web de código abierto, así como un servidor proxy inverso y un servidor de equilibrio de carga. Se ha vuelto muy popular debido a su rendimiento, escalabilidad y capacidad para manejar altas cargas de tráfico, se utiliza ampliamente para manejar y distribuir el tráfico web en entornos de producción.

## PostgreSQL
Es un sistema de gestión de bases de datos relacionales de código abierto y gratuito. Se ha ganado una reputación por su robustez, estabilidad y capacidad para manejar grandes volúmenes de datos y cargas de trabajo complejas.
## Docker Engine …ñ.lllkoDesktop–
Es una herramienta que permite a los desarrolladores crear, probar y ejecutar aplicaciones en contenedores Docker en sus estaciones de trabajo locales. Proporciona una forma conveniente de utilizar Docker en entornos de desarrollo y permite la creación y gestión de contenedores de manera fácil y eficiente.

# Instalacion

## Ejecucion  
En la raiz del proyecto

Ejecutamos:

```yaml
docker-compose up
```
## Error
Se presenta un error porque no puede construirse el archivo docker-compose.yml, se realiza la modificación del archivo.
## Modificar
Reemplazar el archivo docker-compose.yml

```yaml
version: '3.6'

services:
  srvdb:
    image: postgres
    container_name: srvdb
    hostname: srvdb
    environment:
      POSTGRES_USER: consultas
      POSTGRES_DB: consultas
      POSTGRES_PASSWORD: QueryConSql.pwd
    ports:
      - 5433:5433
    networks:
      - demo_kcs_net

  srvwildfly:
    image: myapp
    build: 
      context: .
      dockerfile: Dockerfile
    container_name: srvwildfly
    hostname: srvwildfly
    platform: linux/amd64
    ports:
      - 8080:8080
      - 9990:9990
    depends_on:
      - srvdb
    networks:
      - demo_kcs_net
  
  pgadmin:
    image: dpage/pgadmin4
    environment:
      PGADMIN_DEFAULT_EMAIL: info@jtux.ec
      PGADMIN_DEFAULT_PASSWORD: clave
    ports:
      - 5050:80
    depends_on:
      - srvdb
    networks:
      - demo_kcs_net

  nginx:
    container_name: nginx
    hostname: nginx
    cpus: 1
    image: nginx
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
    ports:
      - 8000:8000
    platform: linux/amd64
    networks:
        - demo_kcs_net
networks:
  demo_kcs_net:
```
## Configuracion del servidor Nginx
1. Creamos una carpera en la raiz del proyecto
2. Creamos un archivo con la extension  ```.conf```
3. El archivo contiene lo siguiente:

```yaml
user nginx;
worker_processes 1;
events {
  worker_connections 1024;
}
http {
  server {
    listen 8000;
    listen [::]:8000;    
    location / {
        proxy_http_version 1.1;
        proxy_pass http://srvwildfly:8080; 
        proxy_redirect     off;
        proxy_set_header   Host $host;
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Host $server_name;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_cache_revalidate on;
        proxy_cache_min_uses 3;
        proxy_cache_use_stale error timeout updating http_500 http_502
        http_503 http_504;
        proxy_cache_background_update on;
        proxy_cache_lock on;
    }
  }
}
```
# DockerFile

```yaml
FROM jboss/wildfly:20.0.1.Final
LABEL maintainer="jorgelojam@gmail.com"

ENV WILDFLY_USER jloja
ENV WILDFLY_PASS JlojaPassword

ENV DS_NAME KitchensinkAngularJSQuickstartDS
ENV DS_USER consultas
ENV DS_PASS QueryConSql.pwd
ENV DS_URI jdbc:postgresql://srvdb/consultas

ENV JBOSS_CLI $JBOSS_HOME/bin/jboss-cli.sh
ENV DEPLOYMENT_DIR $JBOSS_HOME/standalone/deployments/

RUN echo "Adding WildFly administrator"
RUN $JBOSS_HOME/bin/add-user.sh -u $WILDFLY_USER -p $WILDFLY_PASS --silent

# Configure Wildfly server
RUN echo "Starting WildFly server" && \
      bash -c '$JBOSS_HOME/bin/standalone.sh -c standalone.xml &' && \
      bash -c 'until `$JBOSS_CLI -c ":read-attribute(name=server-state)" 2> /dev/null | grep -q running`; do echo `$JBOSS_CLI -c ":read-attribute(name=server-state)" 2> /dev/null`; sleep 1; done' && \
      curl --location --output /tmp/postgresql-42.2.16.jar --url https://jdbc.postgresql.org/download/postgresql-42.2.16.jar && \
      $JBOSS_CLI --connect --command="module add --name=org.postgres --resources=/tmp/postgresql-42.2.16.jar --dependencies=javax.api,javax.transaction.api" && \
      $JBOSS_CLI --connect --command="/subsystem=datasources/jdbc-driver=postgres:add(driver-name="postgres",driver-module-name="org.postgres",driver-class-name=org.postgresql.Driver)" && \
      $JBOSS_CLI --connect --command="data-source add \
        --name=${DS_NAME} \
        --jndi-name=java:jboss/datasources/${DS_NAME} \
        --user-name=${DS_USER} \
        --password=${DS_PASS} \
        --driver-name=postgres \
        --connection-url=${DS_URI} \
        --min-pool-size=10 \
        --max-pool-size=20 \
        --blocking-timeout-wait-millis=5000 \
        --statistics-enabled=true \
        --enabled=true" && \
      $JBOSS_CLI --connect --command=":shutdown" && \
      rm -rf $JBOSS_HOME/standalone/configuration/standalone_xml_history/ $JBOSS_HOME/standalone/log/* && \
      rm -f /tmp/*.jar

COPY target/kitchensink-angularjs.war /opt/jboss/wildfly/standalone/deployments/
EXPOSE 8080
EXPOSE 9990
```

# DockerFile para Nginx

```yaml
FROM nginx
RUN rm /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/conf.d/default.conf
```


# Ejecutamos  
```
docker-compose up
```

Se levanta todo el contenedor de la aplicación.
# Elecuramos la aplicación
```
http://localhost:8080/kitchensink-angularjs/#home
```
# Ejecutamos la aplicación enrutador
``` 
http://localhost:8000/kitchensink-angularjs/#/home
```

# Conclusión
La implementación de un API Gateway proporciona una capa de abstracción y control adicional sobre los servicios de API, lo que resulta en una arquitectura más segura, escalable y fácil de gestionar. Al centralizar la gestión de API, mejorar la seguridad, el rendimiento y la monitorización, un API Gateway puede ayudar a optimizar la entrega de servicios y mejorar la experiencia del usuario final.
