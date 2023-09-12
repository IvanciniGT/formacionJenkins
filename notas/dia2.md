# Aplicación de Gestión de Animalitos Fermín!

Backend servidores propios                              Frontales

    BBDD    <   Servicios Web                   <       App Web (JS: Angular, React)
                                                        App iOS
                altaAnimalito                           App Android
                consultarAnimalitos                     App OK Google / Alexa / Siri

Servicios Web

 Desarrollo -> commit + push (GIT)
 Jenkins -> lee el REPO de git
    -> compilación                                                    <- Desarrollo ágil
        maven
    -> pruebas                                                        <- Integración continua
        junit mockito
    -> Guardar la versión actual que está lista (Artifactory, nexus)  <- Entrega Continua
    -> Instalarlo en mi servidor                                      <- Despliegue continuo
        comunicación con un servidor de apps java

App WEB

 Desarrollo -> commit + push (GIT)
 Jenkins -> lee el REPO de git
    -> compilación                                                    <- Desarrollo ágil
        npm angular-cli
        mocha jasmine
    -> pruebas                                                        <- Integración continua
    -> Guardar la versión actual que está lista (Artifactory, nexus)  <- Entrega Continua
    -> Instalarlo en mi servidor                                      <- Despliegue continuo

App iOS / App Android
 Desarrollo -> commit + push (GIT)
 Jenkins -> lee el REPO de git
    -> compilación                                                    <- Desarrollo ágil
        gradle
    -> pruebas                                                        <- Integración continua
    -> Guardar la versión actual que está lista (Artifactory, nexus)
    -> Publicar la nueva versión en el Store de Turno                 <- Entrega Continua

# Definición de pipelines

## Estilo libre

Esto se hace con formularios... y tiene limitaciones:

Proceso secuencial:
    Tareas / Steps (Pasos)
    Post-Tareas

Algunos plugins se ejecutan como tareas
Mientras que otras se ejecutan como post-tareas
Eso es algo que decide el autor del plugin

Si una tarea falla, dejan de ejecutarse el resto de tareas
Pero se ejecutan en cualquier caso las post-tareas

Las post-tareas están pensadas para cosas como:
- Recopilar información de lo que ha pasado (pruebas, compilación)
- Mandar emails
- Publicar cosas (artefacto) en un repositorio

Pero... aquí ya me encuentro con una limitación... o 2:
- Qué pasa si quiero enviar un email en medio de las tareas que estoy ejecutando
    Claro... que tenemos trucos rastreros: Plugin al canto (Any build step)

- Y si quiero un conjunto de tareas con sus post tareas... y después otro conjunto de tareas... con sus post-tareas?
NASTI DE PLASTI !!!
    Tendría que montar 2 proyectos... y vincularlos de forma que cuando 1 acabe, arranque el otro.
    En serio ????

Estas limitaciones no las tenemos con la forma GUAY (PIPELINES)

## Integración continua... Para qué?

Se centra en la ejecución automatizada de pruebas.
Para qué?
- Para saber qué tal voy
- Para ser capaces de IDENTIFICAR lo antes posible los FALLOS y DEFECTOS... y no arrastrarlos en el código

Y DEBE ACABAR CON UN INFORME !!!!!

### Vocabulario en el mundo del testing

- Errores       Los humanos cometemos ERRORES (por estar cansados, por falta de conocimiento, malas decisiones)
- Defectos      Al cometer un error, introducimos un DEFECTO en un producto
- Fallos        que en un momento dado PUEDE manifestarse como un FALLO

### Para qué sirven las pruebas de software?

- Para ver si se cumplen unos requisitos
- Para identificar FALLOS.
  - Y en el caso de que haya un FALLO, los desarrolladores deben identificar el DEFECTO que causa el fallo (DEPURACION O DEBUGGING)... y para ello, interesa tener a mano TODA LA POSIBLE INFORMACION que me ayude a identificar ese DEFECTO rápido.
- Ayudar en el proceso de DEBUGGING
- Para identificar DEFECTOS < Pruebas estáticas (que no necesitan ejecutar un código) SONARQUBE
- Haciendo un análisis de causas raíces, los testers, desde los DEFECTO pueden identificar los ERRORES que 
  están provocando los DEFECTOS --> Acciones preventivas
- Para saber qué tal voy... y para esto nos sirve la INTEGRACION CONTINUA... entre otras cosas


# Manifiesta Ágil

"El software funcionando es la medida principal de progreso."

La MEDIDA principal que debe utilizarse para determinar el grado de progreso de un proyecto es "El software funcionando"
                                                        ¿qué tal vamos?
¿Qué métrica/indicador utilizo?
    "El software funcionando"
¿Quién dice que el software funciona?
    LAS PRUEBAS

HOY EN DIA MIRAMOS EL NUMERO DE PRUEBAS SUPERADAS POR SEMANA

En las metodologías tradicionales (waterfall-en casada) para medir el avance?
- Preguntar al desarrollador < NO ME PUEDO FIAR !!!!!
- Número de lineas escritas de código en una semana!

# Entrega continua

Poner en manos de mi cliente la última versión funcional de mi sistema, librería, programa...


https://SERVIDOR_JENKINS/
        job/
            Maven_Ivan/
                lastSuccessfulBuild/
                    artifact/
                        proyectos/proyectomaven1/target/proyecto1.jar

https://SERVIDOR_JENKINS/
        job/
            Maven_Ivan/
                8
                    /artifact/
                    proyectos/proyectomaven1/target/proyecto1-1.0-SNAPSHOT.jar                        

## Pipelines de Jenkins

Me permiten definir un Job/Tarea/Proyecto mediante un fichero de texto "SCRIPT"... que llamaremos un Jenkinsfile

Jenkins ofrece 2 sintaxis diferentes para este menester.
- Sintaxis declarativa (Jenkinsfile declarative)
- Sintaxis scripting (Usa el lenguaje de programación groovy)

Dentro de un Job PIPELINE de Jenkins podemos meter el script en el formulario = RUINA GIGANTE !!!
Lo que haré será guardarlo en un ÇREPO de git (o equivalente)
Y darle la ruta del fichero a Jenkins