Jenkins viene de otra herramienta llamada HUDSON (2004-2005)
Hudson en un momento dado tuvo la desgracia de caer en manos del destructor de software por excelencia: Oracle
- MySQL      -> MariaDB
- OpenOffice -> LibreOffice
- Hudson     -> Jenkins

# Problemas proyectos estilo libre

Solo un bloque Stage con sus pasos y sus post.
  En muchos casos esto no es suficiente.

No tenemos gestión de versiones... y no podemos dar marcha atrás... PROBLEMON !

No podíamos llevar un Pipeline de un Jenkins a Otro fácilmente.

Algunas cosas se tenían que implementar de forma muy artificiosa:
- Para poder usar tas tareas que ofrecen los plugins como "post-tareas" en el bloque de "tareas" (pasos)
  había que montar un plugin.. configurarlo... crear un condicional build step.. que se ejecute siempre... RUINA !

Al final... no es tan cómodo como parece... acabaos con una pantalla que no sabemos ni donde estamos.

Lento el definir un pipeline.

# Proyectos pipeline

El script (la definición de pasos) se hace mediante un fichero de texto = GENIAL !

Ese fichero (Jenkinsfile) lo puedo meter en un repo de un sistema de control de versiones.

Puedo tener todos los steps que quiera... con sus bloques de post-tareas (me permite una gestión de errores avanzada)

Me quito de artificios... puedo usar las tareas como post-tareas y viceversa.

Tenemos 2 sintaxis:

- Declarativa (propia de Jenkins) = Más sencilla que la de abajo <- Me vale para el 95% de los casos
- Scripted (basada en groovy)     = Más potente que la de arriba

## Sintaxis declarativa

Nos da una sintaxis propia para estructurar el pipeline.
Pero, una vez creada la estructura, acabamos ejecutando pasos (tareas)

```groovy

pipeline {
    agent any

    tools {
        maven "MiMaven"
    }

    stages {
        stage('Descarga del código') {
            steps {
                git 'https://github.com/IvanciniGT/curso-JenkinsCabot'   # TAREA
            }
        }        
        stage('Compilación') {
            steps {
                sh 'mvn -f proyectos/proyectomaven1/pom.xml compile'       # TAREA
                sh 'mvn -f proyectos/proyectomaven1/pom.xml test-compile'  # TAREA
            }
        }
    }
}
```

La sintaxis de esas tareas es propia de cada plugin. 
Para el 90% de los plugins, voy a encontrar un formulario que me genera su sintaxis... mediante ese GENERADOR DE SNIPPETS
Algunos plugins (contados) no aparecen en el generador de snippets. Ahí me toca partirme la cabeza (documentación del plugin... google)

En definitiva, cuando trabajamos con esta sintaxis (DECLARATIVA), lo único que necesito es acordarme de la ESTRUCTURA
La propia estructura también podemos generarla mediante un GENERADOR dentro de jenkins....
pero esta herramienta va generando trocitos de la estructura

La ESTRUCTURA necesitamos tenerla clara.


# Nodos de Jenkins

Puedo tenerlos precreados (forma tradicional de trabajar.. y caída en desuso)

Puedo configurar un cloud en Jenkins.
En Jenkins un cloud ... no es lo lo que normalmente entendemos por un Cloud (o si... más o menos)

En Jenkins, llamamos CLOUD a un servicio externo capaz de provisionar agentes.

## "Cloud" fuera de Jenkins

Llamamos cloud al conjunto de SERVICIOS que una empresa IT ofrece a través de internet:
- de forma desatendida (sin intervención humana)
- mediante un modelo de pago por uso

Distintos tipos de servicios:
- IaaS: Infraestructura: Almacenamiento, Máquinas virtuales (o físicas), redes
- PaaS: Plataforma:      Kubernetes, MariaDB, Jenkins
- SaaS: Software:        Office 365

# Aplicación Web

2 partes:
- Frontal, desarrollado en JS -> Un repo de git
- Backend, JAVA               -> Otro repo de git
  - BBDD: MariaDB

# Integración continua

Pipeline 1 - Backend
    - Compilación, pruebas iniciales (Unitarias, Calidad de código) NO QUIERO UN MARIADB
    - Pruebas de Integración -> Entorno de integración SI QUIERO UN MARIADB
Pipeline 2 - Frontal
    - Compilación, pruebas iniciales (Unitarias, Calidad de código) NO QUIERO UN BACKEND
    - Pruebas de Integración -> Entorno de integración SI QUIERO UN BACKEND
      - parámetro para ver la exhaustividad de las pruebas
        - Pruebo con el Chrome desde un ordenador
        - Probar mi app desde Chrome, Edge, Safari, Teléfono movil, Tablet, Portatil

Son cosas independientes

## Tengo un repo de git para el backend

COMMITs -> Foto completa de mi proyecto en un momento del tiempo = Backup completo
Vamos guardando una SECUENCIA TEMPORAL de commits... backups... eso lo hace cualquier sistema de BACKUP

RAMAS (BRANCHES) -> Tener distintas SECUENCIA TEMPORAL de commits en paralelo

Cuando trabajamos con GIT, es habitual en los proyecto tener un mínimo de ramas que usamos -> GIT FLOW

Git Flow tiene que ver con las ramas que uso en mi proyecto y con cómo muevo cosas de unas a otras <- POLÍTICA que debe cumplirse de forma estricta.

-> master | main:   Los commits (fotos) que hay en esta rama son VERSIONES LISTAS PARA PRODUCCIÓN
                    NUNCA JAMAS EN LA VIDA se hace un commit en esta rama. Solo pongo en ella commits de otras ramas
                                    ^
                                Haber superado pruebas de alta disponibilidad / carga / rendimiento ...
                                    |
-> release          Corresponde con un entorno de preproducción                                    
                                    ^
                                Haber superado pruebas de integración
                                    |
-> desarrollo:      Para que los desarrolladores vaya INTEGRANDO sus cambios / desarrollo 
                                    ^
                                Para poder pasar cosas de una a otra? Pruebas unitarias
                                    |
-> Ramas por cada cambio        Para que los desarrolladores vayan dejando los cambios / desarrollos que hacen concretos
   Ramas por persona

---

Administrador de sistema es un PROGRAMADOR
Tester                   es un PROGRAMADOR

Antiguamente la responsabilidad de un sysadmin era : Administrar sistemas
Pero hoy en día la responsabilidad de un sysadmin era : construir programas que administren sistemas
    Ansible, Puppet, Chef, Kubernetes, Terraform, Vagrant, Script shell
        ^                                ^
        ------- REPO DE GIT --------------

Antiguamente la responsabilidad de un tester era : Probar sistemas
Pero hoy en día la responsabilidad de un tester era : construir programas que prueben sistemas
    Selenium, JMeter, ReadyAPI
        ^                   ^
        --- REPO DE GIT -----

Hay muchos tipos de software:
- Sistemas operativos
- Aplicaciones
- Drivers
- Scripts*
- Comandos
- Demonios

---

Quiero hacer el provisionamiento de un infraestructura en AWS para desplegar mi aplicación para un nuevo cliente.

Te voy a crear tu propio entorno (maquinitas) donde te voy a montar mi sistema para que lo uses.

Pipeline de Jenkins
Crear infra                Planchar las máquinas                Despliegue de la app
Script de terraform     -> Playbook de Ansible              ->  Ejecutar Scripts BBDD  (Ansible, Sripts sh, jython wlst)
- Servidores (IP)           - Carpetas                          Preconfigurar una BBDD
- Almacenamiento            - Usuarios                          Backups
- Redes /subnet             - Paquetería                        Despliegue
- Balanceadores             - Configuración kernel SO Linux
- Proxy reverso             - SELINUX / AppArmor
- DNS                       - Configurar agentes de monitorización
- Certificados
- Firewall
- Egress
- Ingress

Escalar la infra
 3 servidores en cluster -> 14 servidores en cluster... y lo quiero para dentro de YA !

# Contenedores

Entorno aislado en que el que ejecutar procesos (dentro de un so con kernel Linux).
Los contenedores se crean desde una Imagen de contenedor

# Imagen de contenedor

Un archivo comprimido que lleva dentro:
- Un software ya preinstalado por alguien
- Otros programas / dependencias / librerías / configuración que puedan ser de utilidad

De dónde saco yo una imagen de contenedor...? Registro de repositorios de imagenes de contenedor: DOCKER HUB

IMAGEN DE CONTENEDOR: 3.8.6-openjdk-11
Qué hay dentro? MAVEN v3.8.6 OpenJDK v11

IMAGEN DE CONTENEDOR: 3.8-openjdk-11
Qué hay dentro? MAVEN La última v3.8.X OpenJDK v11

IMAGEN DE CONTENEDOR: 3-openjdk-11
Qué hay dentro? MAVEN La última v3.X.X OpenJDK v11

IMAGEN DE CONTENEDOR: openjdk-11
Qué hay dentro? MAVEN La última versión de maven OpenJDK v11

Me han dicho los desarrolladores me dicen que necesita una determina versión de maven (por incluir una determinada funcionalidad que ellos necesitan) Nosotros ahora estamos trabajando con la v3.8.6-openjdk-11

Qué imagen de esas cojo yo?
IMAGEN DE CONTENEDOR: 3.8-openjdk-11
    Me asegura que tiene la funcionalidad que necesito

                        Cuando suben esos numeritos?
3 <- MAJOR              Breaking changes. Cuando cambio algo que rompe RETROCOMPATIBILIDAD
8 <- MINOR              Nueva funcionalidad
                        Algo se marca como DEPRECATED (OBSOLETO)
6 <- PATCH/MICRO        Cuando se arregla un error(bug)


Docker es un gestor de contenedores que me permite crear y gestionar contenedores en una máquina
(eso es un poco mentira... SWARM que permite trabajar con clusters... de juguete)

Kubernetes es una herramienta que me permite gestionar gestores de contenedores en un cluster de máquinas

Docker trabaja con el concepto de CONTENEDOR
Pero Kubernetes, Openshift (Distro de kub de Redhat), incluso algunos gestores de contenedores un poco más avanzados que docker, como PODMAN no trabajan con el concepto de contenedor, sino con el concepto de: PODS

Qué es un POD?
Un POD es un conjunto de contenedores (puede tener solo 1), que:
- Se despliegan en el mismo host
- Comparten configuración de RED <----- Pueden hablar entre si mediante la palabra: localhost
  - POD A
    - nginx - php
              - php.conf -> URL de la BBDD :       localhost:3306
    - mariadb

Contenedor mariaDB
                ^localhos:3306
Contenedor microserviocios
                ^localhost:8080
Contenedor node -> Web JS
            ^localhost: 80
Contenedor chrome
            ^ Protocolo que soportan hoy en día todos los navegadores WEBDRIVER: 
Contenedor selenium
            ^ localhost:4444
Contenedor donde ejecutar el script de selenium
