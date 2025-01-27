<!-- README.md is generated from README.Rmd. Please edit that file -->

# R Al Parque

Que es lo que obtienes cuando organizas una agrupación (cluster) de
roqueros (rockers)? Un festival!

R Al Parque es una prueba de concepto para coordinar contenedores Docker
y hacerlos trabajar en conjunto como un grupo (*EN:cluster*) de
trabajadores R bajo coordinación del paquete `snow`. En este momento no
le tenemos una aplicación otra que casos ejemplos, y si tienes una
aplicación para la aproximación que describimos aquí nos encantaría
escucharlo.

Para hacerlo usamos contenedores [Docker](https://docker.com), con
imágenes para R como disponibles en
[Rocker](https://github.com/rocker-org/rocker) a los cuales incluimos
[snow](https://cran.r-project.org/web/packages/snow/index.html).

[esquema de la solución propuesta](esquema.png)

## Referencia

En la carpeta `"manual/referencia` hay un ejemplo de hacer un
snow-cluster dentro de una misma maquina. En nuestro caso lo usamos para
hacer correr el código en un mismo contenedor, y lo usamos como
referencia para comparar el resultado después.

Para correrlo sigue los siguientes pasos (asumimos que tienes
[Docker](https://docker.com) instalado y funcionando).

    $ cd manual/referencia
    $ docker build -t referencia .
    $ docker run --name ejemplo -i referencia

Verás en como parte del output, después de ver las funciones que
asignamos a cada uno de los tres maquinas asignadas:

    [1] "Listo! Termine el script del Organizador en la implementacion 'referencia'"
    [1] "el resultado es: \n"
             1          2          3          4          5          6          7 
      7.662232  14.892228  28.978089   8.820645  22.477977  49.578876  34.956109 

Estos valores los tenemos que volver a ver cuando logremos crear un
conjunto de contenedores separados.

# Orquestración manual

## El Parque

El primero paso es crear una red para los contenedores que vamos a
iniciar. En otras palabras: necesitamos un parque para poner el podio.
En Docker lo podemos [hacer
así](http://stackoverflow.com/questions/27937185/assign-static-ip-to-docker-container/35359185#35359185)

    $ docker network create --subnet=172.18.0.0/16 simonbolivar

## Los artistas

Nuestros artistas son los contenedores secundarios. Se construyen como
instancias de un mismo Dockerfile y hay que hacerlo antes de arrancar el
organizador. Sin artistas no hay nada que se pueda organizar.

### Arranca los artistas

Primero construimos la imagen genérica para los artistas

    $ cd manual/artistas
    $ docker build -t artista .

Y después incluimos tres actos en el festival

    $ docker run -d -P --net simonbolivar --ip 172.18.0.2 --name aterciopelados -it artista
    $ docker run -d -P --net simonbolivar --ip 172.18.0.3 --name fabulosos_cadillacs -it artista
    $ docker run -d -P --net simonbolivar --ip 172.18.0.4 --name mana -it artista

## El organizador

Necesitamos un [Mario
Duarte](https://es.wikipedia.org/wiki/Rock_al_Parque) para tomar la
iniciativa. En nuestro caso es el contenedor Docker central que corre
como el contenedor primario que dirige contenedores secundarios.

### Arranca el organizador

El organizador tiene su Dockerfile propio.

    $ cd manual/organizador
    $ docker build -t organizador .
    $ docker run --net simonbolivar --ip 172.18.0.10 --name duarte -it organizador

Lo arrancamos dentro del parque simonbolivar, porque de otra forma no
podria comunicarse con los artistas.

Ahora, el organizador se tiene que conectar con todos los artistas para
estar seguro de que están, y para intercambiar llaves. Por eso al
arrancar el organizador va a preguntar si se puede conectar con el
artista, y va a preguntar por la clave:

    The authenticity of host '172.18.0.2 (172.18.0.2)' can't be established.
    ECDSA key fingerprint is SHA256:oyaLzw1rfFs3r//hQE6ScyNmxk5JXiFUR3M0yidKcpI.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added '172.18.0.2' (ECDSA) to the list of known hosts.
    root@172.18.0.2's password: 

En el Dockerfile incluimos una clave estándar que es “artista”, la cual
es la respuesta por los tres.

Si miras el codigo r en `organizador.R` veras porque pide por la clave
de los artistas ya definidos arriba: incluimos la direccion IP en el
código usando la funcion `makeCluster`.

## Por hacer

#### Conneccion automatica a los artistas

TODO: Intenté hacer el paso arriba “passwordless”, pero aún no funciona.
Lo que esperaba era que al usar ssh-copy-d para los tres artistas
`snow::makeCluster` haría la coneccion de forma automatica.

#### Orquestación Automática

TODO: Orquestar los contenedores para que arranquen en conjunto usando
Docker Compose, y quizás despues en combinación con Swarm. TODO:
Orquestar todos los contenedores dentro de Kubernetes.
