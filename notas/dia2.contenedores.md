# Contenedor

Es un entorno aislado dentro de un SO Linux donde ejecutar procesos:
- Sus propias variables de entorno
- Su propio sistema de archivos
- Su propia configuración de red -> Su propia IP (172.17.0.2/16)
- Limitación de acceso al hierro (CPU, RAM...)


# Linux

Kernel de SO

GNU/Linux es un SO que usa Linux como kernel -> Redhat, debian, ubuntu, suse
Android es otro SO que usa Linux como kernel

# Windows es un SO

DOS                 MSDOS, Windows 3, windows 95, 98, millenium
Tiene kernel? NT    Windows NT, XP, Server, 7, 8, 10, 11

# Instalación de un software en un Servidor: MongoDB / MariaDB / SQL Server / DB2

        App1 | App2 | App3              Si app2 tiene un bug... que pone la CPU pa' freir huevos... calentita
    -------------------------               App2 pasa a estado OFFLINE
        Sistema Operativo                   App1 pasa a estado OFFLINE
    -------------------------               App3 pasa a estado OFFLINE
             Hierro

# Instalación mediante maquinas virtuales

Las máquinas virtuales me ofrecen ENTORNOS AISLADOS donde ejecutar procesos

        App1 | App2 + App3 
    -------------------------
        SO1 |      SO2
    -------------------------
        MV1 |      MV2
    -------------------------
        Hipervisor: VMWare, Citrix, HyperV    
    -------------------------
        Sistema Operativo    
    -------------------------
             Hierro

# Despliegue mediante contenedores

        App1 | App2 + App3 
    -------------------------
         C1  |      C2
    -------------------------
      Gestor de contenedores
      docker + containerd + crio + podman
    -------------------------
        Sistema Operativo    
    -------------------------
             Hierro

    Los contenedores se crean desde imagenes de contenedor.
    
## Instalación de una BBDD

Quiero montar un MySQL en mi portatil de forma tradicional:

0 - Preparar en la máquina todas las cosas (configuraciones, librerias, depenendencias) que necesite el programa que instalo
1 - Descargar un instalador
2 - Ejecuto el instalador ( aquí hay que echarle valor y sudor!)
3 - ejecuto y sonrio

C:\Archivos de Programa\MySQL -> zip -> email
                                  v
                                 IMAGEN DE CONTENEDOR
                                 
## Imagen de contenedor

Un archivo comprimido (tar) que tiene un software YA INSTALADO DENTRO.

Esas imágenes de contenedor se publican en REGISTROS DE REPOSITORIOS DE IMAGENES DE CONTENEDOR para que la gente las pueda descargar
El más famoso se llama Docker hub

Los desarrolladores hoy en día lo que entregan / generan es una IMAGEN DE CONTENEDOR

# Kubernetes

Me permite trabajar con contenedores también... pero ofreciendo:
- Alta disponibilidad:
    Tratar de garantizar que el sistema está operativo un determinado tiempo pactado contractualmente
    Tratar de garantizar la no perdida de información
- Escalabilidad:
    Ajustar la infra a las necesidades de cada momento (arriba o abajo) escalado vertical/horizontal


docker image pull nginx:latest
docker container create --name minginx nginx:latest
docker start minginx


docker image pull httpd:latest
docker container create --name mihttpd httpd:latest
docker start mihttpd
docker stop mihttpd
docker restart mihttpd


Kubernetes < google para su entorno de producción
cluster
    Maquina 1
        crio (equivalente a docker)
    Maquina 2
        crio
    Maquina 3
        crio

Balanceadores de carga / Servicio
Proxy reverso          / Ingress controller

Wordpress
- Apache        --- En el mismo contenedor
- Mysql         --/

Escalo. el wordpress: 2 contenedores
    4 Apache -> 1 MySQL

Maquina Virtual MySQL 5.7.1
                    v
                    actualizar el software
                    v
                MySQL 5.7.2
¿Qué hago?
Actualizo el software

Con contenedores:
    Borro el contenedor creado con la imagen 5.7.1
    Creo un contenedor con la imagen 5.7.2
        Y le pongo el mismo "VOLUMEN" con los datos que tenía el anterior.
        LISTO
        