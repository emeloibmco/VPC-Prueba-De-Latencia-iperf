# VPC - Prueba de Latencia con iperf ☁💻📈
El termino *latencia* corresponde al tiempo que se tarda la transmisión de información a través de una red. Por otro lado, el *ancho de banda* corresponde a la cantidad de datos que se pueden transmitir por segundo (medido en este caso en Mbits/sec). Para realizar esta medición se utiliza el comando ```iperf```, que es una herramienta de la línea de comandos usada en el diagnóstico de problemas de velocidad de red. Este comando mide la capacidad máxima de procesamiento de red que puede manejar un servidor. Es particularmente útil cuando se experimentan problemas de velocidad en la red, debido a que se puede utilizar para determinar cuál servidor es incapaz de llegar al rendimiento máximo. 

Para este ejercicio se implementan 2 VSI ubicadas en Dallas y Londres y posteriormente se realiza la respectiva prueba para medir el ancho de banda entre ambos servidores mediante el comando ```iperf```, tal y como se presenta en los siguientes pasos.

<br />

## Índice  📰
1. [Pre-Requisitos](#Pre-Requisitos-bookmark_tabs)
2. [Tabla sobre latencia de red](#Tabla-sobre-latencia-de-red-clipboard)
3. [Crear VPC con subred y VSI en Dallas y Londres](#Crear-VPC-con-subred-y-VSI-en-Dallas-y-Londres-cloud)
4. [Configurar archivos y acceder a VSI Dallas y VSI Londres](#Configurar-archivos-y-acceder-a-VSI-Dallas-y-VSI-Londres-card_file_box-computer)
5. [Instalar comando iperf3 en cada VSI](#Instalar-comando-iperf3-en-cada-VSI-inbox_tray)
6. [Configurar reglas en VSI](#Configurar-reglas-en-VSI-hammer_and_wrench)
7. [Realizar test de latencia con iperf3](#Realizar-test-de-latencia-con-iperf3-chart_with_downwards_trend-memo)
8. [Referencias](#Referencias-mag)
9. [Autores](#Autores-black_nib)
<br />

## Pre-Requisitos :bookmark_tabs:
* Contar con una cuenta en <a href="https://cloud.ibm.com/"> IBM Cloud </a>.
* Contar con un grupo de recursos específico para la implementación de los recursos.
<br />

## Tabla sobre latencia de red :clipboard: 
*IBM Cloud* cuenta con la herramienta <a href="http://lg.softlayer.com/">lg.softlayer.com</a>, que permite visualizar datos sobre latencia de red entre servidores en distintas ubicaciones. En la siguiente imagen se presenta la tabla que contiene los datos. 
<br />
<p align="center"><img width="700" src="https://github.com/emeloibmco/VPC-Despliegue-VSI-Acceso-SSH/blob/main/Imagenes/latency.png"></p>
<br />
Para este caso, si se analizan los servidores en ubicaciones como Dallas y Londres se obtiene que la latencia de red tiene un valor de ```108 ms```. Es importante aclarar que los datos presentados en la tabla son estáticos y no una representación en tiempo real.
<br />

## Crear VPC con subred y VSI en Dallas y Londres :cloud:
https://github.com/emeloibmco/VPC-Despliegue-VSI-Acceso-SSH/blob/main/README.md#Crear-VPC-cloud
https://github.com/emeloibmco/VPC-Despliegue-VSI-Acceso-SSH/blob/main/README.md#Desplegar-VSI-en-VPC-computer
https://github.com/emeloibmco/VPC-Despliegue-VSI-Acceso-SSH/blob/main/README.md#Configurar-claves-SSH-closed_lock_with_key
Para realizar el test, en primero lugar debe implementar:
* Una *VPC* en Dallas y una *VPC* en Londres. Tomar como guía el paso <a href="https://github.com/emeloibmco/VPC-Despliegue-VSI-Acceso-SSH/blob/main/README.md#Crear-VPC-cloud">Crear  VPC</a>.
* Una subred en cada *VPC* (Dallas y Londres). Tomar como guía el paso <a href="https://github.com/emeloibmco/VPC-Despliegue-VSI-Acceso-SSH/blob/main/README.md#Desplegar-VSI-en-VPC-computer">Crear subred</a>.
* Una *VSI* con SO CentOS en Dallas.

<p align="center"><img width="700" src="https://github.com/emeloibmco/VPC-Despliegue-VSI-Acceso-SSH/blob/main/Imagenes/vsi_dallas.gif"></p>
<br />

* Una *VSI* con SO CentOS en Londres. 

<p align="center"><img width="700" src="https://github.com/emeloibmco/VPC-Despliegue-VSI-Acceso-SSH/blob/main/Imagenes/Crear%20VSI%20Londres.gif"></p>
<br />


Recuerde el paso sobre como <a href="https://github.com/emeloibmco/VPC-Despliegue-VSI-Acceso-SSH/blob/main/README.md#Configurar-claves-SSH-closed_lock_with_key">Configurar claves SSH</a> de forma previa para crear sus instancias. 
<br />

## Configurar archivos y acceder a VSI Dallas y VSI Londres :card_file_box: :computer:
Como se comento en <a href="https://github.com/emeloibmco/VPC-Despliegue-VSI-Acceso-SSH/blob/main/README.md#Configuraci%C3%B3n-adicional-para-acceder-a-la-VSI-por-medio-de-SSH-hammer">Configuración adicional para acceder a la VSI por medio de SSH</a>, para acceder por medio de SSH a las *VSI* de Londres y Dallas, es necesario realizar una configuración previa en las *VSI*. Debido a que las máquinas aprovisionadas tienen sistema operativo *CentOS*, debe previamente instalar el comando ```nano``` ejecutando el siguiente comando:

```
yum install nano
```
Los demás comandos se aplican de la misma forma que en la máquina Ubuntu.


<p align="center"><img width="700" src="https://github.com/emeloibmco/VPC-Despliegue-VSI-Acceso-SSH/blob/main/Imagenes/configssh.gif"></p>

Una vez se realice la configuración podrá conectarse por medio de *SSH* únicamente ingresando el ```password``` configurado.

<p align="center"><img width="700" src="https://github.com/emeloibmco/VPC-Despliegue-VSI-Acceso-SSH/blob/main/Imagenes/ingreso.PNG"></p>
<br />

## Instalar comando iperf3 en cada VSI :inbox_tray:

Actualmente hay dos ramas independientes de ```iPerf``` que se desarrollan en paralelo: ```iPerf2``` y ```iPerf3```. La funcionalidad de estas herramientas es en su mayoría compatible, pero utilizan diferentes puertos de red de forma predeterminada. En ```iPerf1/2``` es 5001, en ```iPerf3``` es 5201.

Las diferencias no son tan significativas, por lo que no es necesario utilizar una versión específica de ```iPerf```, en este ejercicio usaremos ```iperf3```.

Para instalar ```iperf3``` en las VSI *CentOS*, ejecute:

```
yum update
dnf install iperf3
```

<p align="center"><img width="1000" src="https://github.com/emeloibmco/VPC-Despliegue-VSI-Acceso-SSH/blob/main/Imagenes/iperf.PNG"></p>

Una vez termine de instalar, podrá utilizar el comando ```iperf3``` en las *VSI*.
<br />

## Configurar reglas en VSI :hammer_and_wrench:
<br />

## Realizar test de latencia con iperf3 :chart_with_downwards_trend: :memo:
<br />

## Referencias :mag:
<br />

## Autores :black_nib:
<br />


