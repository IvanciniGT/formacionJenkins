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

