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

<p align="center"><img width="700" src="https://github.com/emeloibmco/VPC-Prueba-De-Latencia-iperf/blob/main/Imagenes/latency.png"></p>
<br />

Para este caso, si se analizan los servidores en ubicaciones como Dallas y Londres se obtiene que la latencia de red tiene un valor de ```108 ms```. Es importante aclarar que los datos presentados en la tabla son estáticos y no una representación en tiempo real.

<br />

## Crear VPC con subred y VSI en Dallas y Londres :cloud:
Para realizar el test, en primero lugar debe implementar:
<br />

* Una *VPC* en Dallas y una *VPC* en Londres. Tomar como guía el paso <a href="https://github.com/emeloibmco/VPC-Despliegue-VSI-Acceso-SSH/blob/main/README.md#Crear-VPC-cloud">Crear  VPC</a>.
* Una subred en cada *VPC* (Dallas y Londres). Tomar como guía el paso <a href="https://github.com/emeloibmco/VPC-Despliegue-VSI-Acceso-SSH/blob/main/README.md#Desplegar-VSI-en-VPC-computer">Crear subred</a>.
* Una *VSI* con SO CentOS en Dallas.
<br />

<p align="center"><img width="700" src="https://github.com/emeloibmco/VPC-Prueba-De-Latencia-iperf/blob/main/Imagenes/vsi_dallas.gif"></p>
<br />

* Una *VSI* con SO CentOS en Londres. 

<p align="center"><img width="700" src="https://github.com/emeloibmco/VPC-Prueba-De-Latencia-iperf/blob/main/Imagenes/Crear%20VSI%20Londres.gif"></p>
<br />


Recuerde el paso sobre como <a href="https://github.com/emeloibmco/VPC-Despliegue-VSI-Acceso-SSH/blob/main/README.md#Configurar-claves-SSH-closed_lock_with_key">Configurar claves SSH</a> de forma previa para crear sus instancias. 
<br />

## Configurar archivos y acceder a VSI Dallas y VSI Londres :card_file_box: :computer:
Como se comento en <a href="https://github.com/emeloibmco/VPC-Despliegue-VSI-Acceso-SSH/blob/main/README.md#Configuraci%C3%B3n-adicional-para-acceder-a-la-VSI-por-medio-de-SSH-hammer">Configuración adicional para acceder a la VSI por medio de SSH</a>, para acceder por medio de SSH a las *VSI* de Londres y Dallas, es necesario realizar una configuración previa en las *VSI*. Debido a que las máquinas aprovisionadas tienen sistema operativo *CentOS*, debe previamente instalar el comando ```nano``` ejecutando el siguiente comando:

```
yum install nano
```
Los demás comandos se aplican de la misma forma que en la máquina Ubuntu.
<br />

<p align="center"><img width="700" src="https://github.com/emeloibmco/VPC-Prueba-De-Latencia-iperf/blob/main/Imagenes/configssh.gif"></p>
<br />

Una vez se realice la configuración podrá conectarse por medio de *SSH* únicamente ingresando el ```password``` configurado.
<br />

<p align="center"><img width="700" src="https://github.com/emeloibmco/VPC-Prueba-De-Latencia-iperf/blob/main/Imagenes/ingreso.PNG"></p>
<br />

## Instalar comando iperf3 en cada VSI :inbox_tray:

Actualmente hay dos ramas independientes de ```iPerf``` que se desarrollan en paralelo: ```iPerf2``` y ```iPerf3```. La funcionalidad de estas herramientas es en su mayoría compatible, pero utilizan diferentes puertos de red de forma predeterminada. En ```iPerf1/2``` es 5001, en ```iPerf3``` es 5201.

Las diferencias no son tan significativas, por lo que no es necesario utilizar una versión específica de ```iPerf```, en este ejercicio usaremos ```iperf3```.

Para instalar ```iperf3``` en las VSI *CentOS*, ejecute:

```
yum update
dnf install iperf3
```
<br />

<p align="center"><img width="1000" src="https://github.com/emeloibmco/VPC-Prueba-De-Latencia-iperf/blob/main/Imagenes/iperf.PNG"></p>

Una vez termine de instalar, podrá utilizar el comando ```iperf3``` en las *VSI*.

<br />

## Configurar reglas en VSI :hammer_and_wrench:
Antes de realizar la prueba de funcionamiento para medir el ancho de banda entre un cliente y un servidor *TCP*, debe configurar las reglas de entrada en el grupo de seguridad de la *VPC* cuya *VSI* funcionará como servidor (para este caso se configura en la *VPC* Londres). Para ello realice:

<br />

1.	En ```Infraestructura VPC/VPC Infrastructure``` ubique la sección ```Red/Net``` y de click en la opción ```Grupos de seguridad/Security groups```.
<br />

2.	Seleccione la región en donde se encuentra la *VPC* y visualice el grupo se seguridad. El nombre que aparece se crea de forma automática.
<br />

3.	De click sobre el grupo de seguridad y seleccione la pestaña ```Reglas/Rules```.
<br />

4.	Posteriormente, de click en el botón ```Crear``` ➡ seleccione el protocolo ```TPC``` y en el rango de puertos coloque el puerto ```5201``` que corresponde al puerto de escucha del servidor (definido por defecto con ```iperf3```).  Cuando la configuración esté lista de click en el botón ```Guardar```.
<br />

5.	Espero unos segundos mientras se implementa la regla y verifique que se encuentre correctamente.
<br />

<p align="center"><img width="700" src="https://github.com/emeloibmco/VPC-Prueba-De-Latencia-iperf/blob/main/Imagenes/Configurar%20Reglas.gif"></p>
<br />

> NOTA: puede realizar la configuración de las reglas en ambas *VSI*, en caso de que desee realizar la prueba intercambiando los roles de servidor y de cliente.
<br />

## Realizar test de latencia con iperf3 :chart_with_downwards_trend: :memo:
El último paso consiste en realizar la prueba para medir el ancho de banda entre ambos servidores. Para ello, realice lo siguiente:
<br />

1.	Determine cual *VSI* funcionará como servidor y cual funcionará como cliente. Para este caso de ejemplo se considera:

* ```VSI 1 – Londres```: servidor.
* ```VSI 2 – Dallas```: cliente.
<br />

2.	Acceda a cada una de las *VSI* con la respectiva IP flotante y contraseña. 
<br />

3.	En la *VSI* que funcionará como servidor (en este caso la *VSI* 1) coloque el comando:
```
iperf3 -s
```
<br />

4.	En la *VSI* que funcionará como cliente (en este caso la *VSI* 2) coloque el comando:
```
iperf3 -c <ip_servidor>
```
<br />

Una vez ejecute el comando va a obtener como respuesta una serie de medidas, entre ellas la velocidad de transmisión en Mbits/sec.

<br />

<p align="center"><img width="700" src="https://github.com/emeloibmco/VPC-Prueba-De-Latencia-iperf/blob/main/Imagenes/TestFinalIperf.gif"></p>
<br />

## Referencias :mag:
* <a href="http://lg.softlayer.com/">SoftLayer IP Backbone - Latencia de red</a>.
* <a href="https://www.ibm.com/docs/es/elm/7.0.1?topic=architecture-testing-cloud-data-center-latency">Prueba de latencia del centro de datos de IBM Cloud</a>.
* <a href="https://www.ibm.com/support/pages/how-do-you-determine-traffic-rate-your-hosts-can-manage">How do you determine the traffic rate that your hosts can manage?</a>.
<br />

## Autores :black_nib:
<br />


