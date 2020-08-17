---
title: Validación del rendimiento de la VPN en una red de Microsoft Azure Virtual Network
description: El objetivo de este documento es ayudar a un usuario a validar el rendimiento de red de sus recursos locales en una máquina virtual de Azure.
titleSuffix: Azure VPN Gateway
services: vpn-gateway
author: cherylmc
manager: dcscontentpm
ms.service: vpn-gateway
ms.topic: troubleshooting
ms.date: 05/29/2019
ms.author: radwiv
ms.reviewer: chadmat;genli
ms.openlocfilehash: 7d19cc4a474324ff3e88ec0c5353a47c157ec715
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "86998483"
---
# <a name="how-to-validate-vpn-throughput-to-a-virtual-network"></a>Validación del rendimiento de la VPN en una red virtual

Una conexión a la puerta de enlace de la VPN le permite establecer una conectividad segura y entre entornos locales entre su red virtual dentro de Azure y la infraestructura local de TI.

Este artículo muestra cómo validar el rendimiento de red de los recursos locales en una máquina virtual de Azure.

> [!NOTE]
> Este artículo se ha creado para ayudar a diagnosticar problemas comunes y solucionarlos. Si no puede resolver el problema mediante el uso de la siguiente información, [póngase en contacto con soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

## <a name="overview"></a>Información general

La conexión a la puerta de enlace de la VPN afecta a los siguientes componentes:

* Dispositivo VPN local (vea una lista de [dispositivos VPN validados)](vpn-gateway-about-vpn-devices.md#devicetable).
* Internet público
* Azure VPN Gateway
* Azure VM

El siguiente diagrama muestra la conectividad lógica de una red local en una red virtual de Azure a través de VPN.

![Conectividad lógica de la red del cliente en la red MSFT mediante VPN](./media/vpn-gateway-validate-throughput-to-vnet/VPNPerf.png)

## <a name="calculate-the-maximum-expected-ingressegress"></a>Cálculo de la entrada/salida máxima esperada

1. Determine los requisitos de rendimiento de la línea de base de la aplicación.
1. Establezca los límites de rendimiento de la puerta de enlace de VPN de Azure. Para obtener ayuda, consulte la sección "SKU de puerta de enlace" de [Acerca de VPN Gateway](vpn-gateway-about-vpngateways.md#gwsku).
1. Determine la [Guía de rendimiento de la máquina virtual de Azure](../virtual-machines/sizes.md) para el tamaño de la máquina virtual.
1. Establezca el ancho de banda del proveedor de servicios de Internet (ISP).
1. Calcule el rendimiento esperado tomando el menor ancho de banda de la máquina virtual, VPN Gateway o ISP, que se mide en megabits por segundo (/) dividido por ocho (8).

Si el rendimiento calculado no cumple con los requisitos de rendimiento de línea de base de la aplicación, debe aumentar el ancho de banda del recurso que identificó como el cuello de botella. Para cambiar el tamaño de una instancia de Azure VPN Gateway, vea [Cambio de una SKU de puerta de enlace](vpn-gateway-about-vpn-gateway-settings.md#gwsku). Para cambiar el tamaño de una máquina virtual, vea [Cambio del tamaño de una VM](../virtual-machines/windows/resize-vm.md). Si no obtiene el ancho de banda de Internet previsto, también puede ponerse en contacto con el ISP.

> [!NOTE]
> El rendimiento de VPN Gateway es un agregado de todas las conexiones de sitio a sitio, de red virtual a red virtual o de punto a sitio.

## <a name="validate-network-throughput-by-using-performance-tools"></a>Validación del rendimiento de red mediante el uso de herramientas de rendimiento

Esta validación debe realizarse durante las horas de poca actividad, ya que la saturación de rendimiento del túnel VPN durante las pruebas provoca que no se produzcan resultados precisos.

La herramienta que se usa para esta prueba es iPerf, que funciona en Windows y Linux, y tiene los modos de cliente y servidor. Está limitada a 3 GB/s para máquinas virtuales Windows.

Esta herramienta no lleva a cabo operaciones de lectura/escritura en el disco. Solo se produce el tráfico TCP generado automáticamente de un extremo a otro. Genera estadísticas basadas en la experimentación que mide el ancho de banda disponible entre los nodos de cliente y servidor. Cuando se realiza la prueba entre dos nodos, un nodo actúa como servidor y el otro nodo como cliente. Una vez completada esta prueba, se recomienda que invierta los roles de los nodos para probar el rendimiento de carga y descarga en ambos nodos.

### <a name="download-iperf"></a>Descarga de iPerf

Descargue [iPerf](https://iperf.fr/download/iperf_3.1/iperf-3.1.2-win64.zip). Para más información, vea la [documentación de Ambari](https://iperf.fr/iperf-doc.php).

 > [!NOTE]
 > Los productos de terceros que se tratan en este artículo están fabricados por compañías independientes de Microsoft. Microsoft no otorga ninguna garantía, implícita o de otro tipo, sobre el rendimiento o la confiabilidad de estos productos.

### <a name="run-iperf-iperf3exe"></a>Ejecución de iPerf (iperf3.exe)

1. Habilite una regla NSG/ACL que permita el tráfico (para la prueba de la dirección IP pública en la máquina virtual de Azure).

1. En ambos nodos, habilite una excepción de firewall para el puerto 5001.

   **Windows:** Ejecute el comando siguiente como administrador:

   ```CMD
   netsh advfirewall firewall add rule name="Open Port 5001" dir=in action=allow protocol=TCP localport=5001
   ```

   Para quitar la regla cuando la prueba esté completa, ejecute este comando:

   ```CMD
   netsh advfirewall firewall delete rule name="Open Port 5001" protocol=TCP localport=5001
   ```

   **Azure Linux:** las imágenes de Azure Linux tienen firewalls permisivos. Si hay una aplicación que realiza la escucha en un puerto, se permite el tráfico. Las imágenes personalizadas que se protegen pueden necesitar puertos abiertos de forma explícita. Los firewalls de nivel de sistema operativo Linux comunes incluyen `iptables`, `ufw`, o `firewalld`.

1. En el nodo de servidor, cambie al directorio donde se extrae iperf3.exe. A continuación, ejecute iPerf en el modo de servidor y configúrela para que escuche el puerto 5001 como se indica a continuación:

   ```CMD
   cd c:\iperf-3.1.2-win65

   iperf3.exe -s -p 5001
   ```

   > [!Note]
   > El puerto 5001 se puede personalizar para tener en cuenta las restricciones particulares del firewall de su entorno.

1. En el nodo de cliente, cambie al directorio donde se extrae la herramienta iperf y luego ejecute el siguiente comando:

   ```CMD
   iperf3.exe -c <IP of the iperf Server> -t 30 -p 5001 -P 32
   ```

   El cliente dirige treinta segundos de tráfico en el puerto 5001 al servidor. La marca "-P" indica que estamos realizando 32 conexiones simultáneas en el nodo de servidor.

   La siguiente pantalla muestra el resultado de este ejemplo:

   ![Output](./media/vpn-gateway-validate-throughput-to-vnet/06theoutput.png)

1. (OPCIONAL) Para conservar los resultados de pruebas, ejecute este comando:

   ```CMD
   iperf3.exe -c IPofTheServerToReach -t 30 -p 5001 -P 32  >> output.txt
   ```

1. Después de completar los pasos anteriores, ejecute los mismos pasos con los roles invertidos, de manera que el nodo servidor será ahora el nodo cliente y viceversa.

> [!Note]
> Iperf no es la única herramienta. [NTTTCP es una solución alternativa para la realización de pruebas](https://docs.microsoft.com/azure/virtual-network/virtual-network-bandwidth-testing).

## <a name="test-vms-running-windows"></a>Prueba de máquinas virtuales que ejecutan Windows

### <a name="load-latteexe-onto-the-vms"></a>Cargue Latte.exe en las máquinas virtuales.

Descargue la versión más reciente de [Latte.exe](https://gallery.technet.microsoft.com/Latte-The-Windows-tool-for-ac33093b).

Considere la posibilidad de colocar Latte.exe en una carpeta independiente, como `c:\tools`.

### <a name="allow-latteexe-through-the-windows-firewall"></a>Permitir Latte.exe mediante el firewall de Windows

En el receptor, cree una regla Permitir en el Firewall de Windows para permitir la recepción de tráfico de Latte.exe. Es más fácil permitir todo el programa Latte.exe por su nombre en lugar de determinados puertos TCP de entrada.

### <a name="allow-latteexe-through-the-windows-firewall-like-this"></a>Permita Latte.exe a través del Firewall de Windows de la siguiente forma:

`netsh advfirewall firewall add rule program=<PATH>\latte.exe name="Latte" protocol=any dir=in action=allow enable=yes profile=ANY`

Por ejemplo, si ha copiado Latte.exe a la carpeta "c:\tools", este sería el comando

`netsh advfirewall firewall add rule program=c:\tools\latte.exe name="Latte" protocol=any dir=in action=allow enable=yes profile=ANY`

### <a name="run-latency-tests"></a>Ejecución de pruebas de latencia

Inicie Latte.exe en el receptor (se ejecuta desde CMD, no desde PowerShell):

`latte -a <Receiver IP address>:<port> -i <iterations>`

Es suficiente alrededor de 65 000 iteraciones para obtener unos resultados representativos.

Cualquier número de puerto disponible es válido.

Si la máquina virtual tiene la dirección IP 10.0.0.4, sería similar al siguiente:

`latte -c -a 10.0.0.4:5005 -i 65100`

Inicie Latte.exe en el emisor (se ejecuta desde CMD, no desde PowerShell).

`latte -c -a <Receiver IP address>:<port> -i <iterations>`

El comando resultante es el mismo que en el receptor, excepto con la adición de "-c" para indicar que se trata del "cliente" o del emisor.

`latte -c -a 10.0.0.4:5005 -i 65100`

Espere a que se muestren los resultados. Dependiendo de la distancia que haya entre las máquinas virtuales, podría llevar unos minutos completarlos. Considere comenzar con menos iteraciones para probar si se ha realizado correctamente antes de ejecutar pruebas más largas.

## <a name="test-vms-running-linux"></a>Prueba de máquinas virtuales que ejecutan Linux

Use [SockPerf](https://github.com/mellanox/sockperf) para probar las máquinas virtuales.

### <a name="install-sockperf-on-the-vms"></a>Instalación de SockPerf en las máquinas virtuales

En las máquinas virtuales Linux (tanto en el emisor como en el receptor), ejecute estos comandos para preparar SockPerf en las máquinas virtuales:

#### <a name="centos--rhel---install-git-and-other-helpful-tools"></a>CentOS y RHEL: instalación de GIT y otras herramientas útiles

`sudo yum install gcc -y -q`
`sudo yum install git -y -q`
`sudo yum install gcc-c++ -y`
`sudo yum install ncurses-devel -y`
`sudo yum install -y automake`

#### <a name="ubuntu---install-git-and-other-helpful-tools"></a>Ubuntu: instalación de GIT y otras herramientas útiles

`sudo apt-get install build-essential -y`
`sudo apt-get install git -y -q`
`sudo apt-get install -y autotools-dev`
`sudo apt-get install -y automake`

#### <a name="bash---all"></a>Bash: todo

Desde la línea de comandos de bash (asume que git está instalado)

`git clone https://github.com/mellanox/sockperf`
`cd sockperf/`
`./autogen.sh`
`./configure --prefix=`

"Make" es más lento, puede tardar varios minutos

`make`

"Make install" es rápido

`sudo make install`

### <a name="run-sockperf-on-the-vms"></a>Ejecución de SockPerf en las máquinas virtuales

#### <a name="sample-commands-after-installation-serverreceiver---assumes-servers-ip-is-10004"></a>Comandos de ejemplo después de la instalación. Servidor o receptor: asume que la dirección IP del servidor es 10.0.0.4

`sudo sockperf sr --tcp -i 10.0.0.4 -p 12345 --full-rtt`

#### <a name="client---assumes-servers-ip-is-10004"></a>Cliente: asume que la dirección IP del servidor es 10.0.0.4

`sockperf ping-pong -i 10.0.0.4 --tcp -m 1400 -t 101 -p 12345  --full-rtt`

> [!Note]
> Asegúrese de que no haya saltos intermedios (por ejemplo, una aplicación virtual) durante la prueba de rendimiento entre la máquina virtual y la puerta de enlace.
> Si los resultados de las pruebas de iPERF/NTTTCP anteriores son deficientes (en términos de rendimiento total), consulte el siguiente artículo para comprender los factores clave que subyacen a las posibles causas principales del problema: https://docs.microsoft.com/azure/virtual-network/virtual-network-tcpip-performance-tuning.

En particular, el análisis del seguimiento de la captura de paquetes (Wireshark/Monitor de red) recopilado en paralelo del cliente y del servidor durante estas pruebas ayudará a evaluar el rendimiento deficiente. Estos seguimientos pueden incluir la pérdida de paquetes, la latencia alta, el tamaño de MTU, la fragmentación, la ventana TCP 0, fragmentos desordenados, etc.

## <a name="address-slow-file-copy-issues"></a>Solución de problemas de copia de archivos lenta

Incluso si el rendimiento global evaluado con los pasos anteriores (iPERF/NTTTCP/etc.) era correcto, es posible que experimente una copia lenta de los archivos al usar el Explorador de Windows o al arrastrar y soltar en una sesión de RDP. Este problema suele ser debido a uno o ambos de los siguientes factores:

* Las aplicaciones de copia de archivos, como el Explorador de Windows y RDP, no utilizan varios subprocesos al copiar archivos. Para mejorar el rendimiento, utilice una aplicación de copia de archivos de multiproceso como [Richcopy](https://technet.microsoft.com/magazine/2009.04.utilityspotlight.aspx) para copiar archivos mediante 16 o 32 subprocesos. Para cambiar el número de subprocesos de copia de archivos en Richcopy, haga clic en **Acción** > **Opciones de copia** > **Copia de archivos**.

   ![Problemas de copia lenta de archivos](./media/vpn-gateway-validate-throughput-to-vnet/Richcopy.png)<br>

   > [!Note]
   > No todas las aplicaciones funcionan de la misma manera y no todas las aplicaciones o procesos utilizan todos los subprocesos. Si ejecuta la prueba, puede ver que algunos subprocesos están vacíos y no proporcionarán resultados precisos de rendimiento.
   > Para comprobar el rendimiento de la transferencia de archivos de la aplicación, utilice varios subprocesos aumentando el número de subprocesos sucesivamente o disminuyéndolo para encontrar el rendimiento óptimo de la aplicación o de la transferencia de archivos.

* Velocidad de lectura/escritura del disco de VM insuficiente. Para más información, vea [Solución de problemas de Azure Storage](../storage/common/storage-e2e-troubleshooting.md).

## <a name="on-premises-device-external-facing-interface"></a>Interfaz con orientación externa del dispositivo local

Se han mencionado las subredes de los intervalos locales a las que le gustaría que Azure llegara a través de VPN en la puerta de enlace de la red local. A la vez, defina el espacio de direcciones de la red virtual de Azure en el dispositivo local.

* **Puerta de enlace basada en rutas**: La directiva o el selector de tráfico para las VPN basadas en enrutamiento se configura como conectividad de tipo cualquiera a cualquier (o caracteres comodín).

* **Puerta de enlace basada en directivas**: Las VPN basadas en directivas cifran y dirigen los paquetes a través de túneles de IPsec basados en las combinaciones de prefijos de dirección entre su red local y la red virtual de Azure. Normalmente, la directiva (o selector de tráfico) se define como una lista de acceso en la configuración de la VPN.

* Conexiones **UsePolicyBasedTrafficSelectors**: (si se establece "UsePolicyBasedTrafficSelectors" en $True en una conexión, configurará la puerta de enlace de VPN de Azure de modo que se conecte al firewall VPN basado en directivas de forma local. Si habilita PolicyBasedTrafficSelectors, debe asegurarse de que el dispositivo VPN tiene los selectores de tráfico coincidentes definidos con todas las combinaciones de sus prefijos de red local (puerta de enlace de red local) hacia y desde los prefijos de red virtual de Azure, en lugar de cualquiera a cualquiera.

Una configuración inadecuada puede dar lugar a desconexiones frecuentes dentro del túnel, caídas de paquetes, rendimiento deficiente y latencia.

## <a name="check-latency"></a>Comprobación de la latencia

Puede comprobar la latencia con las siguientes herramientas:

* WinMTR
* TCPTraceroute
* `ping` y `psping` (estas herramientas pueden proporcionar una buena estimación de RTT, pero no se pueden usar en todos los casos).

![Comprobación de latencia](./media/vpn-gateway-validate-throughput-to-vnet/08checkinglatency.png)

Si observa un pico de latencia alta en cualquiera de los saltos antes de entrar en la red troncal de Microsoft, es posible que desee realizar más investigaciones con su proveedor de servicios de Internet.

Si se detecta un pico de latencia grande e inusual en los saltos en "msn.net", póngase en contacto con el soporte técnico de Microsoft para que lo investiguen.

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte el vínculo siguiente:

* [Ayuda y soporte técnico de Microsoft](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)
