# Qué es Jenkins?

Servidor de automatización < Servidor de CI/CD
    CI  Integración continua
    CD  Entrega y Despliegue continuos: Continuous Delivery / Deployment

Podemos ver a Jenkins como un ORQUESTADOR de tareas -> Automatización de Segundo nivel!

Jenkins es una herramienta Opensource y Gratuita.

## Nos permite

- Herramienta que puede trabajar con distintos lenguajes de programación en distintas máquinas para ejecutar JOBs
- Realizar tareas rutinarias
- Los desarrolladores pueden crear esos JOBs, que luego ejecutan otros perfiles (operadores)

## Otras herramientas equivalentes:

- Cloudbees (es la versión de pago de Jenkins)
- Bamboo (El serv. de autom. de Atlassian)
- TeamCity
- Gitlab CI/CD
- Github Actions
- Travis

---

# Devops

La palabra viene de DESarrollo y OPEraciones.

DEVOPS no es un perfil, al menos en su origen.

Devops es una cultura, un movimiento, una filosofía en PRO DE la AUTOMATIZACION, desde El desarrollo a la operación.
DEV -> OPS

                    AUTOMATIZABLE?              HERRAMIENTAS
    Plan                poco
    Code                ahi vamos (poco)                                            ----> SCM (git)
    Build               SI
                                                JAVA: Maven, Gradle, SBT
                                                JS:   NPM, webpack, yarn
                                                .net: MSBuild, dotnet, nuget
------------- Si automatizo hasta aquí ------------------------------------------------> Desarrollo ágil
    Test
        Definición      NO (poco)
        Ejecución       SI
                                                UI: Selenium, Appium, Katalon, UFT
                                                Servicios web: SoapUI, Postman, karate, readyAPI
                                                Calidad de código: SonarQube
                                                Rendimiento, carga: JMeter, LoadRunner
                                                Frameworks pruebas funcionales:
                                                    Java: JUnit, TestNG
                                                    JS: Mocka
                                                    .net: MSTest
                                                    Python: Unittest
        Para ejecutar esas pruebas qué necesito?
            Software listo
            Pruebas listas
            + Entorno de Pruebas donde instalar el software: INTEGRACION, Calidad, Test, Pruebas
              Le tengo creado de antemano? Antiguamente SI... la tendencia a día de hoy es NO...
                                           Hoy en día tenemos entornos de pruebas de USAR Y TIRAR -> Contenedores

        La creación     SI                      docker, kubernetes, openshift, helm
        de entornos                             ansible
        de pruebas                              terraform
                                                vagrant
                                                scripts de la bash(shell), python
------------- Si automatizo hasta aquí -------------------------------------------------> Integración Continua
    Release             SI
                                                make
                                                ant
                                                docker -> Imágenes de contenedor
------------- Si automatizo hasta aquí -------------------------------------------------> Entrega Continua
    Deployment          SI                      \ 
------------- Si automatizo hasta aquí -------------------------------------------------> Despliegue Continuo
    Operation           SI                      / kubernetes, openshift
    Monitor             SI                      prometheus/grafana
                                                elastic   /kibana

DEVOPS: Perfil nuevo que hoy en día nos hace falta... que es quién MONTA EL PROCESO DE ORQUESTACION DE TODOS LOS AUTOM.
        El que configura el Jenkins.

# Metodologías de desarrollo de software

Antiguamente usábamos metodologías en cascada y sus variantes: V, en espiral... <- MET. TRADICIONALES

    TOMA REQUISITOS
        ANALISIS / PLAN
            DESARROLLO
                PRUEBAS
                    IMPLANTACION
                        MNTO (bugs, actualizaciones de software)

    Esas etapas eran en las que dividíamos la gestión de UN PROYECTO de software

    Antiguamente, un proyecto de software ( o un software) considerábamos que estaba en UNA DE ESAS ETAPAS

Hoy en día nos hemos ido a metodologías AGILES.

## Metodologías ágiles

Realizan una entrega incremental del producto de software.. con una finalidad, TENER UNA RAPIDA RETRO-ALIMENTACION!

Entrega 1: 30 días de comienzo del proyecto
    5% de la funcionalidad (100% funcional)
        PRUEBAS A NIVEL DE PRODUCCION !!!!!
        PASO A PRODUCCION

Entrega 2: 45 días de comienzo del proyecto
    +15% de la funcionalidad (100% funcional)
        PRUEBAS A NIVEL DE PRODUCCION !!!!!
        PASO A PRODUCCION

...
Entrega n: 100 días de comienzo del proyecto
    +10% de la funcionalidad (100% funcional)
        PRUEBAS A NIVEL DE PRODUCCION !!!!!
        PASO A PRODUCCION

Hoy en día, un proyecto de software está en TODAS LAS ETAPAS SIMULTANEAMENTE: DESARROLLO, PRUEBAS, OPERACION...

Nuevo problema que tengo: Pruebas y pasos a producción se multiplican!
De dónde saco la pasta, recursos, tiempo? No lo hay... ni pasta, ni gente, ni tiempo!
    Este problema solo tiene una solución: AUTOMATIZAR

---

# Jenkins

Jenkins es una app WEB (está desarrollada en JAVA).
Tendremos que instalarla:
    - Opción 1: Instalación a hierro!   <<<< ERA la tendencia hace unos CUANTOS años !!!
    - Opción 2: A través de un contenedor con Docker: Pruebas conceptuales, está OK... aprender.. jugar...
    - Opción 3: Instalarlo mediante contenedores en un cluster de Kubernetes <<<< ESTA ES LA TENDENCIA A DIA DE HOY

En Jenkins montamos (definimos) PIPELINES, JOBS, TAREAS: Script de automatización (lo que antes montábamos con un script .sh, .bat, .ps1)

En ese script, iremos invocando a programas externos: MAVEN, ANSIBLE, SELENIUM

Jenkins, por su parte, ejecuta ese script. Dónde?
- El script será ejecutado por un EJECUTOR
- Pero ese EJECUTOR necesita recursos: CPU, RAM, MEMORIA.... + PROGRAMAS 
- En definitiva, el ejecutor necesita un ENTORNO donde ejecutarse

En Jenkins, es una MUY MALA PRACTICA que los ejecutores se ejecuten en el mismo entorno donde Jenkins está instalado y ejecutándose.

Una instalación TIPICA DE Jenkins en un entorno de producción:

    Built-in node                                   Nodos Agentes
    - Donde he instalado y se ejecuta Jenkins       - Donde se ejecutan los Ejecutores, que ejecutan los trabajos (Jobs)

OPCION 1
    - Un servidor (físico o virtual)                - Otros servidores (PRECREADOS ESTATICAMENTE) 
                                                    - o contenedores (docker) creados dinámicamente.
OPCION 3
    - Un POD dentro de Kubernetes                   - Otros PODs que se crearán DINAMICAMENTE para los ejecutores

## Ejemplo de lo que sería un pipeline de Integración Continua:

* Integración Continua: Cuando tengo en mi proyecto que :
    Lo último que los desarrolladores van dejando en su repo de GIT (u otro SCM)
    sea:
        - compilado 
        - desplegado en un entorno de integración (que quizás ha sido creado dinámicamente)
        - probado
    de forma AUTOMATIZADA

### Tenemos una app JAVA , que incluye un backend desarrollado con Microservicios (Spring)

- PASO 1: Descargar desde GIT el código fuente que han hecho los desarrolladores
- PASO 2: Compilarlo con MAVEN
- PASO 3: Ejecutar pruebas Unitarias                                                 MAVEN -> (JUNIT+ Mockito)
- PASO 4: Ejecutar pruebas de calidad de código y un análisis de COBERTURA de código (SONAR)
- PASO 5: Recopilar los informes de las pruebas
- PASO 6: Ver si las pruebas se han superado... porque si no... corto! NO SIGO
- PASO 7: Montar un entorno de pruebas (Integración) contenedor (docker o en kubernetes)... o quizás lo tengo ya precreado
- PASO 8: Empaquetar la aplicación (.war) MAVEN
- PASO 9: Desplegar la app en ese entorno de pruebas
- PASO 10: Otras pruebas:
            - Integración
            - Sistema
              - Postman
              - Karate
              - SoapUI
            - Rendimiento
              - JMeter: Linea base (Con el sistema vacío y sólo 1 petición, cuánto tarda) -> Si el dato es NOK -> DESARROLLO
              - JMeter: Pruebas en carga (Con el sistema petado y muchas peticiones) -> Si el dato es NOK -> DESARROLLO o SISTEMAS (middleware)
                                          50%, 100%, 150% - 200% 
- PASO 11: Recopilar los informes de las pruebas
- PASO 12: Analizar esos informes . SI NO: CORTO !
- PASO 13: Publico en un Nexus/Artifactory
- PASO 14: Mandar email / Creo una entrada en una web

### Para poder ejecutar esto...

Una cosa es el programa (script) que defino en Jenkins...
Pero qué necesito para ejecutarlo?
- Entorno que tenga: java + maven
- Un entorno con Sonarqube
- Necesito otro entorno donde poder desplegar la app (un servidor de aplicaciones JAVA: Tomcat, wAS, Librety, JBOSS)
- Necesito un entorno con postman o karate o soapUI
- Necesito un entorno con JMeter

Tradicionalmente, en las infraestructuras JENKINS, teníamos NODOS DEDICADOS con un software preinstalado muy concreto
para ejecutar distintos tipos de tareas

* Calidad de código
Que el código cumpla con una serie de estándares... y esté libre de defectos evidentes
* Cobertura de código
Qué porcentaje del código que tiene asociadas pruebas unitarias (pasen o no)

---

# Clasificación (tipos) de Pruebas de software:

Las pruebas las clasificamos según diversas taxonomías (paralelas entre si)
Cualquier prueba debe probar UNA UNICA característica concreta

## En base al nivel de la prueba

- Unitarias     Se centra en una característica de un componente AISLADO del sistema

                                   H2 (bbdd integrada que hay en JAVA que tiene soporte en memoria)
                                   v
    TEST --  a  -> Servicio Web -> | MariaDB
         <-  b  --

                                Test Double (Mock, Spy, Fake, Stub)
                                v
    TEST --  a  -> Servicio Web | -> MariaDB
         <-  b  --

- Integración   Se centra en la COMUNICACION entre componentes

    TEST --  a  -> Servicio Web -> | MariaDB
         <-  b  --
 
- Sistema       Se centra en el COMPORTAMIENTO del sistema en su conjunto

## En base al objeto de prueba

- Funcionales
- No funcionales
  - Carga
  - Estrés
  - HA
  - Rendimiento
  - UX
  - UI


Tal petición (que detallo) al sistema instalado en TAL entorno, y estando sometido el sistema a TAL carga
debe tener un Percentil 95 inferior a 100ms en contestar, cuando la ejecuto TANTAS VECES

Cuanto tengo de latencia de red entre el servidor y la BBDD? 60ms ( pues si tengo que ir más de 3 veces... jodido voy)
---

# Contenedores

Los contenedores SON LA ALTERNATIVA a la hora de poner software a funcionar dentro de una máquina!
Quiero tener MySQL dentro de un entorno -> Lo monto mediante un contenedor
Quiero tener JMETER dentro de un entorno -> Lo monto mediante un contenedor
Quiero tener MAVEN 3.6 + java 21 dentro de un entorno -> Lo monto mediante un contenedor

Una alternativa a las formas tradicionales de hacer esto. Por ejemplo: Instalación a hierro.

Ofrecen multitud de ventajas frente a otros métodos de instalación / despliegue de software.

---

# Definición de jobs (pipelines, scripts, tareas) dentro de Jenkins

Jenkins ofrece distintas formas de definir estos scripts:
- Estilo libre: Son las más sencillas de crear... y NO VALEN PARA APRENDER DE JENKINS  !!!!!
  - Tienen limitaciones TAN GORDAS que invalidan su uso en entornos reales (de producción)
    Toda la configuración se hace mediante FORMULARIOS, muy sencillos:
        - Punto bueno: Sencillo
        - Punto MUY MALO: 
          - Limitado a los 4 campos que tenga en el formulario
          - Si los jobs los defino metiendo cosas en un formulario... quién hemos dicho que mete esos datos en el formulario? YO, un humano ... dónde queda la AUTOMATIZACION?
        No perdaís de vista que lo que vamos a crear en JENKINS son:  AUTOMATIZACION = PROGRAMAS = SCRIPT
        Dónde debe quedar un código? SIEMPRE En un repo de git o similar
        Los formularios de Jenkins no se guardan en un REPO -> No tengo control de versión
        Y mi script irá cambiando con el tiempo. v1 -> v2

        Como software que es... lo primero que haré antes de ejecutar ese software en el entorno de producción serán PRUEBAS

        Y esas las haré en el mismo entorno que el de producción ? NO DEBERIA...
        Y si tengo un jenkins de integración y uno de producción...
        y las cosas las meto a mano en formularios.
- Pipeline: Estes es el bueno
    Toda la configuración se hace mediante SCRIPTS de programación (Un lenguaje propio de JENKINS / GROOVY)
    Estos scripts los guardamos en un fichero llamado Jenkinsfile
    Y ese fichero irá a git! donde tengo control de versiones

# Tipos de software

- Sistema operativo
- Driver
- Servicio
- Aplicación desktop
- Demonio
- Script***
- Comando

# Plugins de Jenkins

    Script de Jenkins           ---> Jenkins ---> Tareas -> Ejecutores
    - Proyecto de estilo libre                              (nodos)
    - Archivo Jenkinsfile                                      v
                                                              Plugin de Jenkins  
                                                               v
                                                            Instalaremos el software que sea requerido para nuestras tareas: Ansible, Terraform, Sonarqube, JMeter, Maven, Node

TODA la funcionalidad y características de Jenkins se extienden mediante plugins.
- Queremos poder compilar proyectos con gradle: Jenkins -> PLUGIN -> Gradle
- Queremos mandar un proyecto a Sonarqube -> Plugin -> Sonarqube
- Queremos ejecutar un playbook de ansible -> Plugin -> Ansible
- Queremos crear una infraestructura en un cloud con terraform -> Plugin -> Terraform

Los plugins no sirven solamente para comunicar Jenkins con Otras herramientas que me hagan falta al ejecutar tareas
También:
- Crear dinámicamente NODOS
- Cambiar la apariencia completa de Jenkins: Blue Ocean

# Pantallas de Jenkins

Pantalla principal:
- Listado de tareas definidas -> Detalle de una tarea > Listado de ejecuciones pasadas de la tareas
                                        v                                       v
- Nueva tarea -> Tipo de tarea -> Formulario de Configuración           Detalle de una ejecución
                                                                                v
                                                                        Salida (registro - log) de la ejecución 

Jenkins al ejecutar una tarea lo único que hace es ejecutar un script en una terminal de comandos.

# Workspaces de Jenkins

Cada proyecto (tarea, script, pipeline, job) tiene su propia carpeta dentro de la máquina de Jenkins
Habitualmente se encuentra dentro de:
/var/jenkins_home/workspace/<NOMBRE-CADA-PROYECTO>

En POSIX, la carpeta /var/ es para: DATOS DE PROGRAMAS que cambian con el tiempo.

Cada vez que ejecuto un proyecto, se ejecuta sobre esa carpeta.
En esa carpeta no tengo un historial: -> Todas las ejecuciones de la misma tarea (proyecto)  comparten carpeta

Habitualmente, los proyectos, extraerán información de un repo de git. 
Jenkins clonará la información de ese repo de git en el workspace asociado al proyecto.
Cuando compilemos un proyecto... el resultado se guardará en esa carpeta.
Cuando haga pruebas, los informes se guardarán en esa carpeta.

Hay que tener cuidado (y mucho) con que:
- La última ejecución siempre sobreescribe las anteriores

Por tanto:
- Cuando haya archivos que nos interese guardar en el historial, lo tendré que solicitar explícitamente
    En este caso, jenkins guardará los archivos que me interesen en una ruta independiente, que si va a asociada a la ejecución concreta y no al proyecto
- OJO: Imaginad que 
  - Dia 1, ejecuto una compilación OK
           ejecuto unas pruebas OK -> Informe
  - Dia 2, ejecuto una compilación NOK
           ya no ejecuto las pruebas... pero puede pasar
           que en la carpeta tenga un informe de pruebas.. cuál? El de la ejecución PASADA

Jenkins nos va a permitir:
- Eliminar toda o parte de esa carpeta: ANTES o DESPUES de cada ejecución

Como regla general nos interesaría:
- Antes o DESPUÉS de cada ejecución BORRAR LA CARPETA

Cuando no me interesa ésto?
En proyectos grandes, donde:
- Tarda mucho git en clonar el repo
- Tarda mucho en compilarse mi proyecto

Y puedo hacer uso de las "caches" que me ofrezcan las distintas herramientas que uso.
- Con git, la segunda vez que ejecute el proyecto, se traerá solo las novedades / cambios
- Con maven, si detecta que hay archivos ya compilados de un proyecto que no han cambiado, no los recompila: TIENE UNA CACHE DE COMPILACIONES

 # maven

 proyecto/
    src/         Código de la app
    target/
        classes/            Donde maven deja las compilaciones que va haciendo
        surefire-reports/   Donde junit deja los informes de pruebas

Jenkins, después de cada ejecución:
- Guarda los informes de pruebas que haya en la carpeta surefire-reports para su consulta futura
- Borra la carpeta surefire-reports/ de forma que no se mezclen los informes de una ejecución con los de otra

No sería lo mismo un pipeline que ejecute continuamente (entorno de desarrollo)
    Cada vez que desarrollo haga un commit a git, quiero que se compile y despliegue la app en un servidor de desarrollo
        Me aprovecho de la cache de maven... va más rápido
        Aunque me puede dar problemas
            Compilación 1: Programa.java -> Programa.class
            Compilación 2: Programita.java -> Programita.class
                Pero maven no habría borrado el fichero Programa.class de la carpeta target/classes/
                    lo que podría dar problemas (raro, pero podría)
que un pipeline que ejecute antes de un paso a producción (sprint de scrum)
    En este caso, querré asegurarme que se hace una compilación desde CERO del proyecto... y tardará
    Pero estoy seguro de lo que subo

Una cosa es quien configura maven, qué lo haría DESDE DESARROLLO
Otra cosa es quién configura Jenkins para hablar con maven: y borrar carpetas o no las borra o en según que escenarios.. o que carpetas... qué lo haría: El "devops"

---

# SCM?

## GIT?

Un sistema de control de versiones, que nos permite:
- Guardar distintas versiones PARALELAS (RAMAS) de un proyecto de software
- Sincronizar el trabajo de un equipo de desarrollo
- ...

Un concepto clave en los SCM es el concepto de RAMA:
- Una linea de evolución temporal de cambios paralela a otras lineas (ramas)

Versión 1 de mi CV 
    v
Versión 2 de mi CV 
    v
Versión 3 de mi CV

# Commit de GIT

Una foto COMPLETA del proyecto en un momento dado del tiempo.
    COPIA DE SEGURIDAD COMPLETA


# Versiones en software

    1.0.0
    1.1.0
    1.2.0
    v   v
    v   1.2.1 > 1.2.3 ----
    2.0.0
    2.1.0
    2.2.0
    2.2.1

main/master
dev




Banco 4 clusters de 200 maquinas  < PASTA - IBM CLOUD
Dpto mantener el cluster
Dpto devops < Equipo que ayuda a sus desarrolladores a migrar sus app a contenedores
Cambiar todo el sistema de monitorización


Proyecto GC
Nextcloud -> instalada a hierro
                                -> Kubernetes
                                                3 nodos
                                                Esa aplicación, que es una app estandar de mercado
