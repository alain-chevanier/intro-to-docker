# Introducción a Docker

## Objetivo
El objetivo de esta actividad es que el alumno instale docker y que pueda ejecutar sobre él algunas comando para familiarizarse con la herramienta.
También es importante entender algunos conceptos elementales de las herramientas de contenerización y su uso en la industria del software.

## Introducción
Antes de realizar de pasar a la sección de actividades te recomendamos ver los siguientes videos para familiarizarte con algunos conceptos elementales de la herramienta:

* [What is Docker?](https://www.youtube.com/watch?v=jPdIRX6q4jA&list=PLy7NrYWoggjzfAHlUusx2wuDwfCrmJYcs)
* [What is a Docker Container](https://www.youtube.com/watch?v=GeqaTjKMWeY&list=PLy7NrYWoggjzfAHlUusx2wuDwfCrmJYcs&index=2)
* [How to install Docker?](https://www.youtube.com/watch?v=wH9XesmPUOk&list=PLy7NrYWoggjzfAHlUusx2wuDwfCrmJYcs&index=3)
* [8 basic Docker Commands](https://www.youtube.com/watch?v=xGn7cFR3ARU&list=PLy7NrYWoggjzfAHlUusx2wuDwfCrmJYcs&index=4)

## Entregable
En la sección de actividades se describen varios pasos en los que hay que crear varios archivos cuyo contenido es provisto a lo largo de la explicación, coloca dicho archivos en un directorio llamado `actividades`, en dicho directorio también agrega un pequeño reporte (`reporte.md`) con tus impresiones sobre la herramienta, tu experiencia y dificultades para interactuar con ella, adjun algunos pantallazos de la terminal al ejecutar algunos de los comandos que se describen. Si tienes alguna duda aún sobre el funcionamiento de la herramienta por favor posteala en el canal de slack `#laboratorio`.
Nota sobre git, sube todo el contenido de tu entrega en la rama `main`, github classroom ya creo por tu un pull request llamado feedback donde se compara la rama `main` contra la versión original del código del repositorio.

## :computer:  Actividades

### Antes de empezar :exclamation::exclamation:
*Nota: Antes de comenzar deberemos tener instalados docker, docker compose, kubectl y colima, o en su defecto el ambiente docker de nuestra preserencia 

``` 
MAC:
brew install colima
brew install docker
brew install kubectl
brew install docker-compose
```

```
WINDOWS
Install docker desktop or rancher
https://www.docker.com/products/docker-desktop/
https://docs.rancherdesktop.io/getting-started/installation/
```

```
LINUX
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
sudo docker run hello-world
```

para verificar que se instalaron correctamente puedes utilizar los siguientes comando.

```
colima -v
docker --version
docker-compose --version
```
### 1. Crea una carpeta vacia.
*Nota: puedes hacerlo desde tu interfaz grafica o utlizar el siguiente comando en la terminal*
```
mkdir <nombre-de-la-carpeta>
```
### 2. Crear el archivo de aplicación app.py

En la carpeta que acabas de crear crea el archivo app.py con el siguiente contenido
```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello World! I have been seen'
```


### 3. Definimos el archivo de requerimientos 
Dentro de la misma carpeta crearemos un archivo llamado `requirements.txt`

*Nota: Estos archivos puedes hacerlos desde una interfaz grafica o puedes hacerlo desde tu terminal.*

```
touch requirements.txt
```
```
nano requirements.txt
```

Dentro de `requirements.txt` agregamos:
```
flask
```

### 4. Definimos el archivo `Dockerfile`

```
nano Dockerfile
```
Y ponemos lo siguiente dentro de `Dockerfile`

```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run"]
```

### 5. Construimos la imagen en docker
```bash
docker build --tag python-docker .
```
En este punto el motor de docker abra definido el plano para la nueva imagen usando las instrucciones en el archivo DOckerfile

### 6. Listamos las imagenes que existen en nuestro ambiente local.

```bash
docker images
```
Estas son todas las imagenes que es posible instanciar en tu ambiente local como contenedores, si instalaste Rancher o Docker Desktop es probable que te encuentres con más imagenes de la que hemos construido, son parte del ambiente de dichos programas.

### 7. Creamos una etiqueta de la imagen
```bash
docker tag python-docker:latest python-docker:v1.0.0
```
Listamos de nuevo las imagenes, noten que ahora hay 2 etiquetas de las imagenes.

Importante: Etiquetar una imagen no crea una nueva imagen, solo nos permite referenciar a la misma imagen con un nombre distinto, al volver a lista las imágenes podremos ver que el id de las imágenes en ambas etiquetas es el mismo.

### 8. Removemos la etiqueta que acabamos de crear

```
docker rmi python-docker:v1.0.0
```

### 8. Instanciamos un contenedor de la imagen que creamos 

``` 
docker run python-docker
```

### 9. Intentamos llamar  ala imagen que creamos, ya sea de la terminal o del navegador

```
curl --request GET \
--url http://localhost:8080/
```

Notemos que la llamada ha sido rechazada, esto se debe a que la aplicacion intenta llamar un puerto que no está expuesto al host del contenedor

### 10. Listamos los contenedores

```
docker ps
```

### 11. Detenemos al contenedor, recordemos que el nombre puede variar dependiendo del resultado del comando de listado

``` 
docker stop trusting_beaver

# docker stop <nombre_contenedor>
```

### 12. Volvemos a instanciar, ahora exponiendo el puerto en el que está corriendo la aplicación

``` 
docker run --publish 8080:5000 python-docker

# docker run --publish <puerto_maquina_local>:<puerto_contenedor_docker>
```

Volvemos a probar la aplicacion

```
curl --request GET \
--url http://localhost:8080/

# o abrir el navegador en http://localhost:8080/
```

Detenemos de nuevo el contenedor

``` 
docker stop trusting_beaver

docker stop <nombre_contenedor>
```

### 12. Ejecutamos el contenedor en modo deatached 

Correr el contenedor en `segundo plano` 

``` 
docker run -d -p 8080:5000 python-docker
```

En este modo la sesion de terminal no quedará asociada al proceso de docker ejecutando el contenedor

Volvemos a detener el contenedor `docker stop <nombre_contenedor>`, listando primero para obtener el nombre

### 13. Reiniciaremos un contenedor

Listamos todos los contenedores, incluyendo aquellos que no están corriendo

``` 
docker ps -a
```

Reiniciamos el contenedor

```
docker restart trusting_beaver

# docker restart <nombre_contenedor>
```

### 14. Eliminamos el contenedor

``` 
docker rm trusting_beaver modest_khayyam lucid_greider

# docker rm <nombre_contenedor_1> <nombre_contenedor_2> <nombre_contenedor_3>
```

### 15. Ejecutamos el contenedor asignandole un nombre y un parametro de autolimpieza

``` 
docker run --rm -d -p 8080:5000 --name python-server python-docker

# docker run --rm -d -p 8080:5000 --name <nombre_a_asignar_contenedor> <nombre_imagen>
```

En este punto podemos detener el contenedor




