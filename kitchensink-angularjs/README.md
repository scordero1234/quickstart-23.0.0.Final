# title
Pasos para ejecutar la aplicación en el entorno de desarrollo de JBoss
# Proceso
Modificar el archivo docker-compose.yml
se agregar las siguientes lineas

build , context ., dockerfile=Dockerfile
-- Nos ubicamos en la carpeta de la aplicación y ejecutamos docker-compose up.
# NGinx
Creamos una carpeta con el nombre de nginx y creamos el archivo nginx.conf en la carpeta de la aplicación.
# Ejecutamos  docker-compose up
Se levanta todo el contenedor de la aplicación.
# Elecuramos la aplicación
http://localhost:8080/kitchensink-angularjs/#home
# Ejecutamos la aplicación enrutador
http://localhost:8000/kitchensink-angularjs/#/home


