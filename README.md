##Bot Telegram Comuni_bot

[![Build Status](https://travis-ci.org/sergiocaceres/IV.svg?branch=master)](https://travis-ci.org/sergiocaceres/IV)

[![Build Status](https://snap-ci.com/sergiocaceres/IV/branch/master/build_image)](https://snap-ci.com/sergiocaceres/IV/branch/master)  
####Proyecto elegido y breve descripción

Para el proyecto de esta asignatura he decidido realizar un bot de Telegram. Con este bot se pretende resolver el problema de los jugadores de Comunio(una página web en la que se juegan ligas internas en la que existen una serie de puntuaciones para cada jugador según haya jugado en dicha jornada de liga) en la que se podrá consultar el mercado de fichajes, la alineación que tiene para esa jornada y muchas más opciones que se irán indicando en hitos posteriores.

#### Servicios necesarios

- Servidor de base de datos, que posiblemente se realizara en MySQL : El servidor de bases de datos almacenará el nombre de usuario y contraseña de Comunio para así, si ya ha realizado alguna petición al bot, no sería necesario volver a pedirle usuario y contraseña.

- Despliegue en la nube

- Monitorización

- Herramienta aún por determinar para escrapear los datos: Posiblemente se realizarán consultas dependiendo de lo que el usuario pida y éste devuelve la información necesaria para poder ser mostrada

- Usaremos el lenguaje Python para el desarrollo del bot

Probablemente se alojará en una cuenta de Amazon

Usaremos TravisCI para realizar test continuos

Automatizaremos tareas, realizaremos despliegue continuo,.. pero esto se irán explicando en los siguientes hitos.

####Integración continua

En este hito hemos visto como añadir integración continua a nuestro proyecto.

Deberemos subir nuestro proyecto y sincronizarlo con Travis-CI para que realice la integración continua. Primero necesitamos enlazar la cuenta de GitHub al servicio, una vez realizado esto, debemos indicarle que repostorios queremos que controle y una vez hecho esto, indicarle como se compila, las dependencias que debe usar y la ejecución del test. Esto es simplemente añadir un fichero .travis.yml con lo descrito anteriormente. Mi archivo .travis.yml es el siguiente:

```
language: python
python:
  - "2.7"

# command to install dependencies
install: make install

# command to run tests
script: make test
```

Le estamos indicando el lenguaje que usamos, la instalación de las dependencias y la ejecución mediante un fichero Makefile. Dicho fichero es el siguiente:

```
install:
	pip install -r requirements.txt

test:
	cd bot_telegram && python test.py

execute:
	cd bot_telegram && python bot.py
```

Nos fijamos en cada opción del Makefile. La primera opción "install" nos instala las dependencias que necesitamos, las cuales las tenemos en el fichero requirements.txt (más adelante lo mostraré). La siguiente opción es "test", llamando a test.py que es donde lo tenemos definido. Por último la opción "execute" que nos ejecutaría el bot

Por tanto, Travis analizará cada commit que se realice al repositorio y realizará un test para ver si está correcto o no

![Imagen 1](http://i68.tinypic.com/htd0ur.png)

Y travis nos dice lo siguiente:

![Imagen 2](http://i63.tinypic.com/1z64n6c.png)

![Imagen 3](http://i67.tinypic.com/2evck13.png)

Podemos ver que ha funcionado correctamente y por eso se nos genera el icono en verde que pone "build passing", insertado al inicio de este documento.

![Imagen 4](http://i66.tinypic.com/6gvh8j.png)

También le hemos dicho a travis que tenemos una variable de entorno y que queremos que guarde su valor. Para ello, accedemos a los ajustes del repositorio donde lo queremos y añadimos el nombre y su valor. Quedaría algo asi:

![Imagen 5](http://i64.tinypic.com/rkos42.png)


####Despliegue en un PaaS

Para este hito he decidido realizar el despliegue en Heroku. Para ello, nos hemos tenido que registrar en la web y a partir de ahi crear una aplicación con una base de datos. La base de datos que he usado ha sido la propia de Heroku (PostgreSQL) y que está alojada en AmazonAWS. Para acceder a dicha base de datos realizamos lo siguiente (en el código de nuestro botesta hecho):

```
con_bd = psycopg2.connect(database='d6f0n6kc34qjo7',user=(os.environ["USR_BD"]),password=(os.environ["PASS_BD"]),host='ec2-54-225-117-56.compute-1.amazonaws.com')
```
donde, para ello, nos hemos definido dos variables de entorno, llamadas ```USR_BD y PASS_BD``` para no mostrarlo públicamente y que alguien nos pueda modificar las tablas.

He hecho exactamente lo mismo con el Token del Bot. 

Añadir que para que Travis pase los test y Heroku funcione correctamente, en el apartado Settings de ambos servicios debemos configurar estas variables con su valor.

Una vez hecho esto, accedemos a la web de Heroku y seleccionamos la casilla que dice que tras pasar los test de Travis nuestra aplicación se subirá a Heroku. Esto es marcando simplemente esta casilla:

![Imagen 6](http://i68.tinypic.com/2m4ocab.png)

Por tanto, cada vez que hagamos un push a nuestro repositorio se esperará a que pase los test de Travis. Si lo logra, la aplicación se despliegue en Heroku.

Para el despliegue necesitaremos un fichero llamada Procfile, cuyo contenido es el siguiente:

```
worker: cd bot_telegram && python bot.py 
```

Como se trata de un bot de Telegram, necesitamos poner "worker" o "bot" para que a los 60 segundos sin recibir peticiones no se caiga. SI tuviesemos páginas webs simplemente con poner web bastaría.

Este fichero debe encontrarse en la raíz y es el archivo que Heroku ejecuta, por tanto, debemos decirle que ejecute el bot, en mi caso el fichero bot.py.

A parte, he creado un archivo llamado runtime.txt que contiene la versión de Python que estamos usando. El contenido de dicho fichero es el siguiente:

```
python-2.7.12 
```

Una vez que tengamos nuestros ficheros de forma correcta, nos disponemos a subirlo a Heroku. Para ello simplemente nos creamos una nuesva aplicación, le damos el nombre que queramos y configuramos su despliegue como expliqué anteriormente, sincronizado con GitHub.

Para acceder a la base de datos y ahi poder crear las tablas que deseemos, tenemos que poner la orden 

```
heroku logs --tail --ps postgres --app <nombre_app>
```
o, podemos acceder a las características de nuestra base de datos(que se encuentra en la página de Heroku), y copiar la url que aparece en Heroku CLI en nuestra terminal. 

Una vez que tenemos el bot listo, solo falta comprobar que está lanzado y ejecutándose. Lo vemos en los logs:

![Imagen 7](http://i67.tinypic.com/35brp7b.png)

Ahi podemos ver que nuestro bot está lanzado y funcionando.

En definitiva, hemos visto como configurar el despliegue en Heroku, usar la base de datos que Heroku nos proporciona como es PostgreSQL y los diferentes archivos de configuración que necesitamos para que la aplicación funcione correctamente. 

Podremos probar nuestra aplicación hablando al bot por el nombre de **@Comuni_bot**. Veamos simplemente un /start para ver que nos responde(podemos ver las diferentes funcionalidades en el fichero bot.py):

![Imagen 8](http://i66.tinypic.com/r0nq50.jpg)

Por tanto, podemos ver que el despliegue está bien hecho y que nuestra aplicación esta funcionando.

También he realizado test usando Snap-Ci. Heroku desplegará la aplicación automaticamente si dicho test se ha pasado correctamente. Podemos ver el resultado en la siguiente imagen(aparte del botón de la build)

![Imagen 9](http://i63.tinypic.com/15o75so.png)
