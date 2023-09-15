## Sintaxis declarativa

pipeline {

    agent

    tools

    parameters

    stages {
        stage('NOMBRE1') {
            steps{
                // TRABAJOS CONCRETOS. Los hace siempre un PLUGIN.
                // Al instalar un plugin se activan ciertos "COMANDOS"
                // O bien a través de la documentación o de ejemplos... o del Generador de Snippets
                // Me busco la vida para generar las entradas en este fichero
            }
            post {
                always{
                    // TRABAJOS CONCRETOS
                }
                success{}
                failure {}
            }
            agent{}
            environment{}
            when{}
        }
        stage('NOMBRE2') {
            /*
            matrix{
                axis{}
                exlusions{}
                stages{}
            } Bucle
            */
            parallel {
                stage('SubEtapa1') {
                    steps{}
                    post {
                        always{}
                        success{}
                        failure {}
                    }
                }
                stage('SubEtapa2') {
                    steps{}
                    post {
                        always{}
                        success{}
                        failure {}
                    }
                }
            }
            post {
                always{}
                success{}
                failure {}
            }
        }
    }

}


Pipeline de Jenkins

                                                                             despliegue de la app
                                                                                    ^
                                                                                    |
    ----t1-----t2----------t3------t4----------t5-----------t6----------------------t7----------->
        |                                                   |
        v                                                   v
    Mandar a sonar                             Esperar al dictamen del sonar
        |                                                       | ^
        | Proporciona:                                          v |
        |   credenciales                                     +------------------------------+
        |   url del sonar                                    |                        BBDD  |
        v                                 Jenkins (API) ---> | OK o NOK = PROYECTO ID       |
      maven (plugin de sonar)                  ^    ^        +------------------------------+
        |                                      |    |
        |  Comunicación                        |    +----< Se instala mediante el plugin de Sonar
        |  asíncrona                           |
        |                                      | < Credenciales
        v                                      |
    SonarQube (API REST) ............... QualityGate -> OK
                                                        NOK
                                            Webhook (al ocurrir un evento notifica a una URL)




## Pruebas en pararelo

                                             Informe de Cobertura de código (JACOCO)
                        Pruebas unitarias--> Informe de Pruebas unitarias
                           ^                    (Si cada prueba va bien o no +log +tiempo)
                           |
    ---t1-------------+----t2---------^----------------------------------------------->
        |             |               |
        v             +----t3----t4---+
    Compilación            |
                           v
                        Pruebas de calidad de código SONAR?
                            - Smell codes (código que no sigue buenas prácticas)
                            - Defectos (bugs) (no has inicializado una variable)
                            - Cobertura de pruebas (revisa que tengas suficientes pruebas unitarias, no si éstas van bien o mal)
                                Para determinar ésto, es necesario ejecutar las pruebas... para ver por qué lineas de código se está pasando.
                                Pero sonar NO EJECUTA CODIGO.
                                El código lo ejecuto cuando lanzo las pruebas unitarias
                                    Y al lanzarlas puedo solicitar, mediante un plugin que monte en maven
                                    un informe JACOCO (Cobertura de codigo)
                                Ese informe se lo he de pasar a Sonar.



## Pruebas secuenciales

                                          --> Informe Jacoco de cobertura
                                                Has ejecutado las lineas de la 70-170 de tu código
                                                Sonar calcula: Si en total hay 200 lineas -> 50%
                        Pruebas unitarias --> Informe de Pruebas unitarias
                           ^                    (Si cada prueba va bien o no +log +tiempo)
                           |
    ---t1------------------t2---------t3-------------------------t4------------------------------------------>
        |                             |                          |
        v                             v                          v
    Compilación         Pruebas de calidad de código      Esperar dictamen de Sonar
                            * Necesita el informe JACOCO

* Y si las pruebas unitarias fallan? Tengo informe de calidad ded código? NO

stage   Descarga de git
stage   Compilación
stage   Pruebas
        Lanzo las pruebas unitarias
        Siempre -> Guarde el informe de pruebas unitarias
Siempre ->  Lanzar el sonar
            Esperar al sonar

Pero... mi código empieza a quedar RARO / perder estructura.. y esto va en detrimento de:
- Mantenibilidad del código
- Cuando veo en la pantalla del Jenkins la ejecución... si fallan las pruebas... qué ha fallado?
  - A buscar logs

---
# Qué me jode en este caso?

Qué es lo que me está limitando? La sintaxis Declarativa del jenkins
Que dentro de un post/always no puedo meter un stage

Opciones: 
1- Sintaxis scripted... donde no tengo ese problema
2- Dividir el pipeline en 2


                                          --> Informe Jacoco de cobertura
                                                Has ejecutado las lineas de la 70-170 de tu código
                                                Sonar calcula: Si en total hay 200 lineas -> 50%
                        Pruebas unitarias --> Informe de Pruebas unitarias
                           ^                    (Si cada prueba va bien o no +log +tiempo)
                           |
 P1 ---t1------------------t2----ACABA---> guardando ARTEFACTOS EN JENKINS (informe JACOCO)
        |                           
        v                           
    Compilación


 P2 >t0-----t3-------------------------t4--------------ACABA
            |                          |
            v                          v
    Pruebas de calidad de código    Esperar dictamen de Sonar
      * Necesita el informe JACOCO

    t0: Traerte el informe JACOCO del P1

    En qué casos puede fallar el segundo pipeline?
        - Que sonar esté caído                  ----> ALARMA !!
        - Si el informe de sonar es negativo    ----> No deben sonar las alarmas

    * Si soy el desarrollador qué me importa? Si el sonar dice que mi proyecto está bien o no
    * Si soy el administrador del Jenkins / devops? Me importa que sonar esté operativo y que los pipeline se estén ejecutando correctamente
        Desde mi punto de vista, que el sonar diga que el proyecto no pasa... significa que el pipeline ha fallado? NO



Cuando se debe ejecutar el P2? Siempre que acabe el primero... aunque falle!

    P1 -> P2

Cómo definimos el flujo de PIPELINES?
- Configurar en P1 que cuando acabe llame a P2
- Configurar que P2 se ejecute DESPUÉS de P1

Pienso que estamos montando un sistema con componentes ACOPLADOS, tanto en un escenario como en otro.
Hoy en día nos hemos dado cuenta en el mundo del desarrollo del software que esas arquitecturas no nos llevan a un buen puerto.
- Montar un nuevo Pipeline de orquestación

    P3
    v   v
    P1  P2

    P1 no falle si la compilación se da por fallida
    P1 no falle si las pruebas se dan por fallidas

    Aunque la compilación falle o las pruebas fallen... el pipeline se ha ejecuta correctamente
    Otra cosa es que lo que el código del desarrollado sea una castaña!

    P2 no falle si el sonar dice que el proyecto es una RUINA !
    Aunque el sonar diga que el proyecto es una ruina, el pipeline se ejecuta bien!

    Y el DEVOPS (/Administrador de Jenkins) va a mirar cómo acabn P1 y P2...
        Y tendrán una pantallita en Jenkins (VISTA), donde está monitorizando estos Pipelines: P1 y P2

    Y haré que P3 falle si hay un problema de compilación o de ejecución de pruebas o de QualityGate
        Y tendré una pantallita en Jenkins (VISTA), donde los desarrolles este monitorizando estos pipelines P3

    Y esto son solo DECISIONES ! (buenas prácticas)

---
Git soporta el trabajo con SUBMODULOS
Y yo puedo tener un repo de git que dentro apunte a otro repo de git

Repo con el software    Repo con Jenkins (a clonar este repo, puedo pedir que se me clonen los submodules que tenga)
/src/                   pipelines/
    ...                     compilacion
/pom.xml                    calidad
                            integracion continua
                        -> Referencia al repo del software
---

Tengo un script de Terraform que crea :
- Una red en Amazon AWS
- Crea unos servidores que pincha en esa RED
  - De forma que los servidores ganan varias direcciones IP
        - Interfaz de administración
        - Interfaz de servicio
  
    Desde terraform PODÍA llamar a Chef, Habitat, Puppet, and Salt 

Playbook de ANSIBLE/Puppet/chef/Salt que se encarga de terminar de configurar cada máquina:
    - Para hacer esa configuración, lo primero que necesita es entrar a la máquina
    - Y para ello lo primero es saber la IP
  
  En Ansible, se definen por separado 2 cosas:
    - Playbook (Script de tareas)
    - Inventario  (IP, fqdn)

300 scripts Terraform > 200 playbooks de Puppet
                        1-> reescribir todos los scripts de Puppet en Ansible

2-> Tengo que cambiar todos los scripts de terraform para que ahora llamen a Ansible y no a Puppet

Me acabo de meter en un PROBLEMON !!!!!
Y el problema es que de esto no me doy cuenta hasta dentro de 4 años!


Pipeline de Jenkins (Orquestador de tareas)         ... más cositas
 v ^                            v ^
300 scripts Terraform        200 playbooks de Ansible

---

Tal petición (que detallo) al sistema instalado en TAL entorno, y estando sometido el sistema a TAL carga
debe tener un Percentil 95 inferior a 100ms en contestar, cuando la ejecuto TANTAS VECES.

Esta prueba en qué entorno debería de realizarla? Producción es el que tiene el entorno real
Aunque... me da un poco miedo hacer la prueba en PRODUCCION... joder ... NO DEJA DE SER PRODUCCION
Decisiones:
    - PRODUCCION -> Planificación... Cuidado con la basura que genero... que habrá que borrarla...
    - PRE-PROPRODUCCION -> Aquí no hay problema de esas cosas... pero hay otro: PASTA !!!!
        Claro... si puedo crear el entorno fácilmente y desmantelarlo cuando quiera...
        Y me sale barato... y tardo poco en hacerlo, me interesa.
            Cloud: [Terraform + Ansible] (PRODUCCION) -> PRE-PRODUCCION

Sea como fuere.. me parece que eso se escapa del objetivo de este pipeline

Ahora... quizás hay otra prueba interesante que puedo hacer ahora:
- Tal petición (que detallo) al sistema instalado en TAL entorno, y SIN ESTAR SOMETIDA A NINGUNA CARGA DE TRABAJO: SOLO 1 petición cada vex debe tener un Percentil 95 inferior a 100ms en contestar, cuando la ejecuto TANTAS VECES.
  - Qué me permite establecer esto? Una linea base con la que trabajar de referencia
        ---> 800ms... Me planteo hacer las otras pruebas en producción? NO

            Esto no pasa a producción

    Pregunta... en el entorno de producción, la app irá más rápida o más lenta? 
        - En el entorno de producción es un requisito (o algo que tengamos en cuanta)
            el que vaya rápida? No mucho... me la pela.. algunas cositas si... pero pocas.
        - Cuales son mis requisitos... en que me fijo? qué necesito conseguir?
            - Alta disponibilidad
                El servicio tiene que funcionar SI o SI... y ésto tiene un coste... GRANDE !
                - Necesito balanceadores de carga
                - Cuando guardo un dato en el ORACLE (archiveLog, redoLog)-> 
                    Esto sirve para que si hay un problema con la BBDD pueda hacer una restauración del sistema... Depende lo crítico de los datos lo haré o no... Backups
                        Implica: CADA OPERACION que se hace contra la BBDD se guarda en fichero, antes de dar la confirmación
                - Weblogic -> cluster
                                                                                        CACHE DISTRIBUIDA
                        Federico --->                     Balanceador ->  Weblogic 1            Redis
                                                                                                Coherence
                                                                          Weblogic 2            Redis
---

# Y aún más dificil. Mortal hacia atras con tres tirabuzones:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>curso</groupId>
  <artifactId>proyecto1</artifactId>
  <version>1.0-SNAPSHOT</version>
  ...
```

Qué pasa con la versión?
  <version>1.2.3</version>

Maven, cuando genera el artefacto: miaplicativo-v1.2.3.jar ... o cuando yo quiera subir ese artefacto a un repo de artefactos le tendré que asociar la versión

Quién escribe ese valor en el fichero? El desarrollador... NO... eso quiero que se haga de forma AUTOMATIZADA!
Entre otras cosas... porque ese no es el único sitio donde está la versión.
    En el código también.... y también ahí pondré que se cargue automáticamente
En qué otro sitio está? Y DEBE DE ESTAR !!!!!
    ES MAS... queremos tener el mismo dato en varios sitios escrito? En general eso... en la vida es bueno? NO
        Por qué? porque se me puede quedar des-sincronizado = FOLLÓN
                 + más trabajo de mnto.
    SOLO HAY UN SITIO DONDE TENGO QUE TENER LA VERSION... y esto aplica a : un software, un playbook de ansible, un script de terraform, una prueba de jmeter
    CUAL ES? El garante de las versiones. El que controla las versiones. El único e inimitable: GIT 

Cómo controla GIT las versiones? ETIQUETAS: TAGS

Y quiero que sea git quien las controle:
* O configuro HOOKs de git, para que cuando las cosas se pasen de una rama a otra se cambien las versiones


            Cuándo haya un commit nuevo en esta rama, qué va a pasar? 
                Que tendré un proyecto/pipeline en Jenkins que hará un despliegue en producción

    hotfix     1.1.1-hotfix1    > para bajar a producción... Quiero otro pipeline de jenkins
                ^
    main       1.1.0        2.0.0
        ^                       Quiero a alguna persona del mundo haciéndolo? La que me lian es parda
        ^                           Solo Jenkins lo debe hacer... cuando se hayan superado todas las pruebas de paso a producción de un sistema
    release    1.1.0-rc2
        ^
    desarrollo 1.1.0-des6


Pero voy a hacer que Jenkins sea quien cambie los commits de unas ramas a otras

---

