---
title: Optimización del rendimiento de TCP/IP para máquinas virtuales de Azure | Microsoft Docs
description: Obtenga información sobre diversas técnicas de optimización del rendimiento de TCP/IP comunes y su relación con máquinas virtuales de Azure.
services: virtual-network
documentationcenter: na
author: rimayber
manager: paragk
editor: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/02/2019
ms.author: rimayber
ms.reviewer: dgoddard, stegag, steveesp, minale, btalb, prachank
ms.openlocfilehash: 67b635f09cb9407279e89b5f7b8526dab3c08946
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87068516"
---
# <a name="tcpip-performance-tuning-for-azure-vms"></a>Optimización del rendimiento de TCP/IP para máquinas virtuales de Azure

En este artículo se describen técnicas de optimización del rendimiento de TCP/IP comunes y algunos aspectos a tener en cuenta cuando se usan para máquinas virtuales que se ejecutan en Azure. Se proporciona una introducción básica de las técnicas y se explora cómo se pueden optimizar.

## <a name="common-tcpip-tuning-techniques"></a>Técnicas de optimización de TCP/IP comunes

### <a name="mtu-fragmentation-and-large-send-offload"></a>MTU, fragmentación y descarga de envíos grandes

#### <a name="mtu"></a>MTU

La unidad de transmisión máxima (MTU) es el tamaño de marco más grande (paquete), especificado en bytes, que se puede enviar mediante una interfaz de red. La MTU es un valor configurable. El valor predeterminado de MTU utilizado en las máquinas virtuales de Azure y la configuración predeterminada de la mayoría de los dispositivos de red a nivel global es de 1 500 bytes.

#### <a name="fragmentation"></a>Fragmentación

La fragmentación se produce cuando se envía un paquete que supera la MTU de una interfaz de red. La pila TCP/IP dividirá el paquete en partes más pequeñas (fragmentos) que se ajusten a la MTU de la interfaz. La fragmentación se produce en la capa de IP y es independiente del protocolo subyacente (por ejemplo, TCP). Cuando se envía un paquete de 2 000 bytes a través de una interfaz de red con una MTU de 1 500, el paquete se divide en un paquete de 1 500 bytes y un paquete de 500 bytes.

Los dispositivos de red de la ruta entre un origen y un destino pueden eliminar los paquetes que superan la MTU o fragmentar el paquete en partes más pequeñas.

#### <a name="the-dont-fragment-bit-in-an-ip-packet"></a>El bit No Fragmentar de un paquete IP

El bit No Fragmentar (DF) es una marca en el encabezado del protocolo IP. El bit DF indica que los dispositivos de red de la ruta entre el remitente y el receptor no deben fragmentar el paquete. Este bit se puede establecer por diversos motivos. (Consulte la sección "Detección de la MTU de la ruta" de este artículo para obtener un ejemplo). Cuando un dispositivo de red recibe un paquete con el bit No Fragmentar establecido y ese paquete supera la MTU de la interfaz del dispositivo, el comportamiento estándar del dispositivo es eliminar el paquete. El dispositivo envía de vuelta un mensaje de fragmentación necesaria de ICMP al origen del paquete.

#### <a name="performance-implications-of-fragmentation"></a>Implicaciones de rendimiento de la fragmentación

La fragmentación puede afectar al rendimiento de forma negativa. Una de las razones principales para el efecto en el rendimiento es el impacto de la fragmentación y el reensamblado de paquetes en la CPU y la memoria. Cuando un dispositivo de red tiene que fragmentar un paquete, tendrá que asignar recursos de CPU y memoria para llevar a cabo la fragmentación.

Lo mismo sucede cuando se vuelve a ensamblar el paquete. El dispositivo de red tiene que almacenar todos los fragmentos hasta que se reciben para poder reensamblarlos en el paquete original. Este proceso de fragmentación y reensamblado también puede provocar latencia.

La otra posible implicación negativa de la fragmentación en el rendimiento es que los paquetes fragmentados pueden llegar desordenados. Cuando se reciben paquetes desordenados, algunos tipos de dispositivos de red pueden eliminarlos. Cuando esto ocurre, se tiene que retransmitir el paquete completo.

Normalmente, los dispositivos de seguridad, como los firewalls de red o cuando los búferes de recepción de un dispositivo de red se agotan, eliminan los fragmentos. Cuando se agotan los búferes de recepción de un dispositivo de red, este intenta volver a ensamblar el paquete fragmentado pero no dispone de recursos para almacenar y volver a ensamblar el paquete.

La fragmentación se puede ver como una operación negativa, pero la compatibilidad con la fragmentación es necesaria cuando se conectan redes diversas a través de Internet.

#### <a name="benefits-and-consequences-of-modifying-the-mtu"></a>Ventajas y consecuencias de modificar la MTU

Por lo general, puede crear una red más eficaz si aumenta la MTU. Todos los paquetes que se transmiten tienen una información de encabezado que se agrega al paquete original. Cuando la fragmentación crea más paquetes, hay más sobrecarga de encabezados y esto hace que la red sea menos eficiente.

Este es un ejemplo. El tamaño del encabezado de Ethernet es de 14 bytes más una secuencia de comprobación del marco de 4 bytes para garantizar la coherencia del marco. Si se envía un paquete de 2 000 bytes, se agregan los 18 bytes de sobrecarga de Ethernet a la red. Si el paquete se fragmenta en un paquete de 1 500 bytes y un paquete de 500 bytes, cada paquete tendrá 18 bytes del encabezado de Ethernet, un total de 36 bytes.

Tenga en cuenta que aumentar la MTU no crea necesariamente una red más eficaz. Si una aplicación envía solo paquetes de 500 bytes, existirá la misma sobrecarga de encabezado si la MTU es de 1 500 bytes o de 9 000 bytes. La red será más eficaz únicamente si utiliza mayores tamaños de paquete que se ven afectados por la MTU.

#### <a name="azure-and-vm-mtu"></a>Azure y la MTU de la máquina virtual

La MTU predeterminada de las máquinas virtuales de Azure es de 1 500 bytes. La pila de la red virtual de Azure intentará fragmentar un paquete de 1 400 bytes.

Tenga en cuenta que la pila de la red virtual no es intrínsecamente ineficaz porque fragmenta los paquetes de 1 400 bytes, aunque las máquinas virtuales tengan una MTU de 1 500. Un porcentaje elevado de paquetes de red son mucho menores de 1 400 o 1 500 bytes.

#### <a name="azure-and-fragmentation"></a>Azure y la fragmentación

La pila de la red virtual está configurada para eliminar los "fragmentos desordenados", es decir, los paquetes fragmentados que no llegan en el orden de fragmentación original. Estos paquetes se eliminan principalmente debido a una vulnerabilidad de la seguridad de la red que se anunció en noviembre de 2018 llamada FragmentSmack.

FragmentSmack es un defecto de la manera en que el kernel de Linux administra el reensamblado de paquetes IPv4 e IPv6 fragmentados. Un atacante remoto podría utilizar este defecto para desencadenar operaciones de reensamblado de fragmentos costosas, lo que puede provocar un aumento del uso de la CPU y una denegación de servicio en el sistema de destino.

#### <a name="tune-the-mtu"></a>Ajuste de la MTU

Puede configurar un valor de MTU de la máquina virtual de Azure, al igual que es posible en cualquier otro sistema operativo. Pero se debe tener en cuenta la fragmentación que se produce en Azure, que se ha descrito anteriormente, cuando se configura una MTU.

No recomendamos a los clientes el aumento de la MTU de la máquina virtual. Esta explicación intenta explicar los detalles de cómo implementa Azure la MTU y cómo realiza la fragmentación.

> [!IMPORTANT]
>No está demostrado que el aumento de la MTU mejore el rendimiento y podría tener un impacto negativo en el rendimiento de la aplicación.
>
>

#### <a name="large-send-offload"></a>Descarga de envíos grandes

La descarga de envíos grandes (LSO) puede mejorar el rendimiento de la red mediante la descarga de la segmentación de los paquetes al adaptador Ethernet. Cuando se habilita LSO, la pila TCP/IP crea un paquete TCP grande y lo envía al adaptador Ethernet para su segmentación antes de reenviarlo. La ventaja de LSO es que puede liberar a la CPU de la segmentación de los paquetes en tamaños que se ajusten a la MTU y descarga ese procesamiento a la interfaz Ethernet, donde se realiza en hardware. Para más información acerca de las ventajas de LSO, consulte [Supporting large send offload](https://docs.microsoft.com/windows-hardware/drivers/network/performance-in-network-adapters#supporting-large-send-offload-lso) (Compatibilidad con la descarga de envíos grandes).

Cuando LSO está habilitado, los clientes de Azure podrían ver tamaños de marco grandes al realizar capturas de paquetes. Estos tamaños de marco grandes podrían llevar a algunos clientes a pensar que se está produciendo la fragmentación o que se está usando una MTU grande cuando no es así. Con LSO, el adaptador Ethernet puede anunciar un tamaño de segmento máximo (MSS) mayor a la pila TCP/IP para crear un paquete TCP mayor. Este marco no segmentado completo se reenvía al adaptador Ethernet y sería visible en una captura de paquetes realizada en la máquina virtual. Pero el adaptador Ethernet dividirá el paquete en varios marcos más pequeños, según la MTU del adaptador Ethernet.

### <a name="tcp-mss-window-scaling-and-pmtud"></a>Escalado de la ventana de MSS de TCP y PMTUD

#### <a name="tcp-maximum-segment-size"></a>Tamaño de segmento máximo de TCP

El tamaño de segmento máximo de TCP (MSS) es una configuración que limita el tamaño de los segmentos TCP, lo que evita la fragmentación de los paquetes TCP. Los sistemas operativos normalmente utilizan esta fórmula para establecer el MSS:

`MSS = MTU - (IP header size + TCP header size)`

El encabezado IP y el encabezado TCP son de 20 bytes cada uno o de 40 bytes en total. Por lo tanto, una interfaz con una MTU de 1 500 tendrá un MSS de 1 460. Pero el MSS es configurable.

Esta configuración es aceptada en el protocolo de enlace de tres vías de TCP cuando se configura una sesión TCP entre un origen y un destino. Ambos lados envían un valor de MSS y el menor de los dos se usa para la conexión TCP.

Tenga en cuenta que las MTU del origen y el destino no son los únicos factores que determinan el valor de MSS. Los dispositivos de red intermedios, como las puertas de enlace de VPN, incluido Azure VPN Gateway, pueden ajustar la MTU independientemente del origen y el destino para garantizar un rendimiento óptimo de la red.

#### <a name="path-mtu-discovery"></a>Detección de la MTU de la ruta

El MSS se negocia, pero podría no indicar el MSS real que se puede usar. Esto es porque otros dispositivos de red de la ruta entre el origen y el destino podrían tener un valor de MTU menor que el origen y el destino. En este caso, el dispositivo cuyo MTU sea menor que el paquete eliminará el paquete. El dispositivo enviará de vuelta un mensaje de fragmentación necesaria de ICMP (Tipo 3, Código 4) que contiene su MTU. Este mensaje de ICMP permite al host de origen reducir su MTU de la ruta adecuadamente. El proceso se denomina detección de la MTU de la ruta (PMTUD).

El proceso PMTUD es ineficaz y afecta al rendimiento de la red. Cuando se envían paquetes que superan la MTU de la ruta de red, los paquetes se deben retransmitir con un MSS inferior. Si el remitente no recibe el mensaje de fragmentación necesaria de ICMP, quizás debido a un firewall de red en la ruta (conocido comúnmente como un *agujero negro de PMTUD*), el remitente no sabe que necesita reducir el MSS y retransmitirá continuamente el paquete. Esta es la razón por la que no se recomienda aumentar la MTU de la máquina virtual de Azure.

#### <a name="vpn-and-mtu"></a>VPN y MTU

Si usa máquinas virtuales que realizan encapsulación (por ejemplo, las VPN de IPsec), hay algunas consideraciones adicionales sobre el tamaño de paquete y la MTU. Las VPN agregan más encabezados a los paquetes, lo que aumenta el tamaño del paquete y requiere un MSS menor.

Para Azure, se recomienda que establezca el bloqueo de MSS de TCP en 1 350 bytes y la MTU de la interfaz del túnel en 1 400. Para más información, consulte la [página de dispositivos VPN y parámetros de IPSec/IKE](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpn-devices).

### <a name="latency-round-trip-time-and-tcp-window-scaling"></a>Latencia, tiempo de ida y vuelta y escalado de la ventana de TCP

#### <a name="latency-and-round-trip-time"></a>Latencia y tiempo de ida y vuelta

La latencia de red se rige por la velocidad de la luz en una red de fibra óptica. El rendimiento de red de TCP también se rige de forma eficaz por el tiempo de ida y vuelta (RTT) entre dos dispositivos de red.

| Enrutar | Distancia | Tiempo unidireccional | RTT |
| ----- | -------- | ------------ | --- |
|De Nueva York a San Francisco|4 148 km|21 ms|42 ms|
|De Nueva York a Londres|5 585 km|28 ms|56 ms|
|De Nueva York a Sydney|15 993 km|80 ms|160 ms|

Esta tabla muestra la distancia en línea recta entre dos ubicaciones. En las redes, la distancia es normalmente mayor que la distancia en línea recta. Esta es una fórmula sencilla para calcular el RTT mínimo en función de la velocidad de la luz:

`minimum RTT = 2 * (Distance in kilometers / Speed of propagation)`

Puede usar 200 para la velocidad de propagación. Se trata de la distancia, en kilómetros, que recorre la luz en 1 milisegundo.

Tomemos de Nueva York a San Francisco como ejemplo. La distancia en línea recta es de 4 148 km. Sustituyendo ese valor en la ecuación, se obtiene lo siguiente:

`Minimum RTT = 2 * (4,148 / 200)`

El resultado de la ecuación se expresa en milisegundos.

Si desea obtener el mejor rendimiento de red, la opción lógica consiste en seleccionar destinos con la distancia más corta entre ellos. También debe diseñar la red virtual para optimizar la ruta del tráfico y reducir la latencia. Para más información, consulte la sección "Consideraciones de diseño de la red" de este artículo.

#### <a name="latency-and-round-trip-time-effects-on-tcp"></a>Latencia y efectos del tiempo de ida y vuelta en TCP

El tiempo de ida y vuelta tiene un efecto directo en el rendimiento máximo de TCP. En el protocolo TCP, el *tamaño de ventana* es la cantidad máxima de tráfico que se puede enviar a través de una conexión TCP antes de que el remitente deba recibir confirmación por parte del receptor. Si el MSS de TCP está establecido en 1 460 y el tamaño de ventana de TCP está establecido en 65 535, el remitente puede enviar 45 paquetes antes de que tenga que recibir confirmación del receptor. Si el remitente no recibe la confirmación, retransmitirá los datos. Esta es la fórmula:

`TCP window size / TCP MSS = packets sent`

En este ejemplo, 65 535 / 1 460 se redondea a 45.

Este estado de "en espera de confirmación", un mecanismo para garantizar la entrega confiable de los datos, es lo que hace que el RTT afecte al rendimiento de TCP. Cuanto más tiempo esté el remitente en espera de confirmación, más tiempo debe esperar antes de enviar más datos.

Esta es la fórmula para calcular el rendimiento máximo de una única conexión TCP:

`Window size / (RTT latency in milliseconds / 1,000) = maximum bytes/second`

Esta tabla muestra el rendimiento máximo en megabytes por segundo de una única conexión TCP. (Para mejorar la legibilidad, se usan megabytes para la unidad de medida).

| Tamaño de la ventana de TCP (bytes) | Latencia de RTT (ms) | Rendimiento máximo en megabytes/segundo | Rendimiento máximo en megabits/segundo |
| ----------------------- | ---------------- | ---------------------------------- | --------------------------------- |
|65 535|1|65,54|524,29|
|65 535|30|2,18|17,48|
|65 535|60|1,09|8,74|
|65 535|90|0,73|5,83|
|65 535|120|0,55|4.37|

Si se pierden paquetes, se reducirá el rendimiento máximo de una conexión TCP mientras el remitente retransmite datos que ya ha enviado.

#### <a name="tcp-window-scaling"></a>Escalado de la ventana de TCP

El escalado de la ventana de TCP es una técnica que aumenta dinámicamente el tamaño de la ventana de TCP para permitir que se envíen más datos antes de solicitar una confirmación. En el ejemplo anterior, se enviarían 45 paquetes antes de solicitar una confirmación. Si aumenta el número de paquetes que se pueden enviar antes de necesitar una confirmación, se reduce el número de veces que un remitente está en espera de confirmación, lo que aumenta el rendimiento máximo de TCP.

Esta tabla muestra esas relaciones:

| Tamaño de la ventana de TCP (bytes) | Latencia de RTT (ms) | Rendimiento máximo en megabytes/segundo | Rendimiento máximo en megabits/segundo |
| ----------------------- | ---------------- | ---------------------------------- | --------------------------------- |
|65 535|30|2,18|17,48|
|131 070|30|4.37|34,95|
|262 140|30|8,74|69,91|
|524 280|30|17,48|139,81|

Pero el valor del encabezado TCP para el tamaño de la ventana de TCP tiene solo 2 bytes, lo que significa que el valor máximo de una ventana de recepción es de 65 535. Para aumentar el tamaño máximo de la ventana, se introdujo un factor de escala de la ventana de TCP.

El factor de escala también es un valor que se puede configurar en un sistema operativo. Esta es la fórmula para calcular el tamaño de la ventana de TCP mediante el uso de factores de escala:

`TCP window size = TCP window size in bytes \* (2^scale factor)`

Este es el cálculo para un factor de escala de la ventana de 3 y un tamaño de la ventana de 65 535:

`65,535 \* (2^3) = 262,140 bytes`

Un factor de escala de 14 da como resultado un tamaño de la ventana de TCP de 14 (el desplazamiento máximo permitido). El tamaño de la ventana de TCP será de 1 073 725 440 bytes (8,5 gigabits).

#### <a name="support-for-tcp-window-scaling"></a>Compatibilidad con el escalado de la ventana de TCP

Windows puede establecer diferentes factores de escala para diferentes tipos de conexión. (Las clases de conexiones incluyen centros de datos, Internet, etc). Puede usar el comando `Get-NetTCPConnection` de PowerShell para ver el tipo de conexión del escalado de la ventana:

```powershell
Get-NetTCPConnection
```

Puede usar el comando `Get-NetTCPSetting` de PowerShell para ver los valores de cada clase:

```powershell
Get-NetTCPSetting
```

Puede establecer el tamaño inicial de la ventana de TCP y el factor de escalado de TCP en Windows mediante el comando `Set-NetTCPSetting` de PowerShell. Para más información, consulte [Set-NetTCPSetting](https://docs.microsoft.com/powershell/module/nettcpip/set-nettcpsetting?view=win10-ps).

```powershell
Set-NetTCPSetting
```

Esta es la configuración de TCP eficaz para `AutoTuningLevel`:

| AutoTuningLevel | Factor de escalado | Multiplicador de escalado | Fórmula para<br/>calcular el tamaño máximo de la ventana |
| --------------- | -------------- | ------------------ | -------------------------------------------- |
|Disabled|None|None|Tamaño de la ventana|
|Restringido|4|2^4|Tamaño de la ventana * (2^4)|
|Muy restringido|2|2^2|Tamaño de la ventana * (2^2)|
|Normal|8|2^8|Tamaño de la ventana * (2 ^ 8)|
|Experimental|14|2^14|Tamaño de la ventana * (2^14)|

Esta configuración es la que más puede afectar al rendimiento de TCP, pero tenga en cuenta que también pueden afectar al rendimiento de TCP muchos otros factores de Internet, fuera del control de Azure.

#### <a name="increase-mtu-size"></a>Aumento del tamaño de la MTU

Dado que una MTU mayor implica un mayor MSS, tal vez se pregunte si aumentar la MTU puede aumentar el rendimiento de TCP. Probablemente no. Existen ventajas y desventajas relativas al tamaño de paquete más allá del tráfico TCP. Como se explicó anteriormente, los factores más importantes que afectan al rendimiento de TCP son el tamaño de la ventana de TCP, la pérdida de paquetes y el RTT.

> [!IMPORTANT]
> No se recomienda que los clientes de Azure cambia el valor de la MTU predeterminada de las máquinas virtuales.
>
>

### <a name="accelerated-networking-and-receive-side-scaling"></a>Redes aceleradas y escalado en la recepción

#### <a name="accelerated-networking"></a>Redes aceleradas

Las funciones de red de las máquinas virtuales históricamente han sido de alto uso de CPU tanto en la máquina virtual invitada como en el hipervisor o host. La CPU del host procesa por software todos los paquetes que transitan a través del host, incluida toda la encapsulación y desencapsulación de la red virtual. Por tanto, cuanto más tráfico pase por el host, mayor será la carga de CPU. Y si la CPU del host está ocupada con otras operaciones, esto también afectará al rendimiento de la red y la latencia. Azure resuelve este problema con las redes aceleradas.

Las redes aceleradas proporcionan una latencia de red muy baja y coherente mediante el uso de hardware programable interno de Azure y tecnologías como SR-IOV. Las redes aceleradas trasladan gran parte de la pila de redes definida por software de Azure desde las CPU a tarjetas SmartNIC basadas en FPGA. Este cambio permite que las aplicaciones de usuario final puedan reclamar ciclos de proceso, lo que supone una carga menor en la máquina virtual y reduce la inestabilidad y las incoherencias en la latencia. En otras palabras, el rendimiento puede ser más determinista.

Las redes aceleradas mejoran el rendimiento al permitir que la máquina virtual invitada pueda omitir el host y establecer una ruta de datos directamente con una tarjeta SmartNIC del host. Estas son algunas de las ventajas de las redes aceleradas:

- **Menor latencia / Más paquetes por segundo (pps)** : al eliminarse el conmutador virtual de la ruta de datos, se elimina el tiempo que los paquetes pasan en el host para el procesamiento de las directivas y se aumenta el número de paquetes que se pueden procesar en la máquina virtual.

- **Inestabilidad reducida**: el procesamiento del conmutador virtual depende de la cantidad de directivas que deben aplicarse y la carga de trabajo de la CPU que se encarga del procesamiento. Al descargarse la aplicación de directivas en el hardware se elimina esa variabilidad, ya que los paquetes se entregan directamente a la máquina virtual y se elimina la comunicación del host a la máquina virtual, así como todas las interrupciones de software y los cambios de contexto.

- **Disminución de la utilización de la CPU**: el pasar por alto el conmutador virtual en el host conlleva una disminución de la utilización de la CPU para procesar el tráfico de red.

Para usar redes aceleradas, las debe habilitar explícitamente en cada máquina virtual aplicable. Consulte [Creación de una máquina virtual Linux con redes aceleradas](https://docs.microsoft.com/azure/virtual-network/create-vm-accelerated-networking-cli) para obtener instrucciones.

#### <a name="receive-side-scaling"></a>Escalado en la recepción

El escalado en la recepción (RSS) es una tecnología del controlador de red que distribuye la recepción de tráfico de red de forma más eficaz mediante la distribución del procesamiento de recepción en varias CPU en un sistema multiprocesador. En términos sencillos, RSS permite a un sistema procesar más tráfico recibido porque usa todas las CPU disponibles en lugar de una sola. Para obtener una explicación más técnica sobre RSS, consulte [Introduction to Receive Side Scaling](https://docs.microsoft.com/windows-hardware/drivers/network/introduction-to-receive-side-scaling) (Introducción al escalado en la recepción).

Para obtener el mejor rendimiento cuando se habilitan las redes aceleradas en una máquina virtual, deberá habilitar RSS. RSS también puede proporcionar ventajas en máquinas virtuales que no usan redes aceleradas. Para obtener información general sobre cómo determinar si RSS está habilitado y cómo habilitarlo, consulte [Optimización del rendimiento de red para máquinas virtuales de Azure](https://aka.ms/FastVM).

### <a name="tcp-time_wait-and-time_wait-assassination"></a>TCP TIME_WAIT y TIME_WAIT assassination

TCP TIME_WAIT es otra configuración común que afecta al rendimiento de la red y la aplicación. En máquinas virtuales ocupadas que abren y cierran muchos sockets, sea como clientes o como servidores (Dirección IP de origen:Puerto de origen + Dirección IP de destino:Puerto de destino) durante el funcionamiento normal de TCP, un socket determinado puede acabar en un estado TIME_WAIT durante mucho tiempo. El estado TIME_WAIT está pensado para permitir que los datos adicionales se entreguen en un socket antes de cerrarlo. Por lo que las pilas TCP/IP generalmente impiden la reutilización de un socket mediante la eliminación del paquete TCP SYN del cliente de forma silenciosa.

La cantidad de tiempo que un socket está en estado TIME_WAIT es configurable. Puede variar entre 30 segundos y 240 segundos. Los sockets son un recurso finito y el número de sockets que se pueden usar en un momento dado es configurable. (El número de sockets disponibles normalmente es de aproximadamente 30 000). Si se consumen los sockets disponibles o si clientes y servidores tienen una configuración de TIME_WAIT no coincidente y una máquina virtual intenta volver a usar un socket en un estado de TIME_WAIT, se producirá un error en las nuevas conexiones, ya que los paquetes TCP SYN se descartan silenciosamente.

El valor del intervalo de puertos para los sockets de salida normalmente es configurable en la pila TCP/IP de un sistema operativo. Lo mismo ocurre con la configuración de TCP TIME_WAIT y la reutilización de sockets. La modificación de estos números puede mejorar potencialmente la escalabilidad. Sin embargo, según la situación, estos cambios podrían causar problemas de interoperabilidad. Debe tener cuidado si cambia estos valores.

Puede usar TIME_WAIT assassination para abordar esta limitación de escalado. TIME_WAIT assassination permite que un socket se reutilice en determinadas situaciones, como cuando el número de secuencia del paquete IP de la nueva conexión supera el número de secuencia del último paquete de la conexión anterior. En este caso, el sistema operativo permitirá que se establezca la nueva conexión (aceptará el nuevo SYN/ACK) y forzará el cierre la conexión anterior que estaba en un estado de TIME_WAIT. Esta funcionalidad se admite en las máquinas virtuales Windows de Azure. Para obtener información sobre la compatibilidad con otras máquinas virtuales, póngase en contacto con el proveedor del sistema operativo.

Para obtener información acerca de cómo configurar los valores de TCP TIME_WAIT y el intervalo de puertos de origen, consulte [Configuración que puede modificarse para mejorar el rendimiento de red](https://docs.microsoft.com/biztalk/technical-guides/settings-that-can-be-modified-to-improve-network-performance).

## <a name="virtual-network-factors-that-can-affect-performance"></a>Factores de red virtual que pueden afectar al rendimiento

### <a name="vm-maximum-outbound-throughput"></a>Rendimiento máximo de salida de la máquina virtual

Azure proporciona una variedad de tamaños y tipos de máquinas virtuales, cada uno con una combinación diferente de funcionalidades de rendimiento. Una de estas funcionalidades es el rendimiento de red (o ancho de banda) medido en megabits por segundo (Mbps). Dado que las máquinas virtuales se hospedan en hardware compartido, la capacidad de red debe compartirse equitativamente entre las máquinas virtuales que usan el mismo hardware. A las máquinas virtuales más grandes se les asigna más ancho de banda que a las más pequeñas.

El ancho de banda de red asignado a cada máquina virtual se mide en el tráfico de salida de la máquina virtual. Todo el tráfico de red que deja la máquina virtual se cuenta para el límite asignado, independientemente del destino. Por ejemplo, si una máquina virtual tiene un límite de 1 000 Mbps, ese límite se aplica tanto si el tráfico saliente está destinado a otra máquina virtual en la misma red virtual como a una fuera de Azure.

La entrada no se mide o no está directamente limitada. Sin embargo, hay otros factores, como los límites de la CPU y de almacenamiento, que pueden afectar a la capacidad de una máquina virtual de procesar los datos entrantes.

Las redes aceleradas están diseñadas para mejorar el rendimiento de red, incluidas la utilización de la CPU, el rendimiento y la latencia. Las redes aceleradas pueden mejorar el rendimiento de una máquina virtual, pero solo pueden hacerlo hasta el ancho de banda asignado de la máquina virtual.

Las máquinas virtuales de Azure tienen al menos una interfaz de red conectada. Pueden tener varias. El ancho de banda asignado a una máquina virtual es la suma de todo el tráfico saliente de todas las interfaces de red conectadas a la máquina. En otras palabras, el ancho de banda se asigna por máquina virtual, independientemente de la cantidad de interfaces de red conectadas a la máquina.

El rendimiento de salida esperado y el número de interfaces de red admitidas en cada tamaño de máquina virtual se detallan en [Tamaños de las máquinas virtuales Windows en Azure](https://docs.microsoft.com/azure/virtual-machines/windows/sizes?toc=%2fazure%2fvirtual-network%2ftoc.json). Para ver el rendimiento máximo, seleccione un tipo, como **De uso general** y, a continuación, busque la sección acerca de los tamaños y las series en la página resultante (por ejemplo, "Serie Dv2"). Para cada serie, hay una tabla que proporciona las especificaciones de redes en la última columna, que se titula "Nº máx. de NIC/ancho de banda de red esperado (Mbps)".

El límite de rendimiento se aplica a la máquina virtual. El rendimiento no se ve afectado por estos factores:

- **Número de interfaces de red**: el límite de ancho de banda se aplica a la suma de todo el tráfico de salida de la máquina virtual.

- **Redes aceleradas**: aunque esta característica puede ser útil para alcanzar el límite publicado, no cambia dicho límite.

- **Destino del tráfico**: todos los destinos cuentan para el límite de salida.

- **Protocolo**: todo el tráfico saliente a través de todos los protocolos cuenta para el límite.

Para más información, consulte [Ancho de banda de red de las máquinas virtuales](https://aka.ms/AzureBandwidth).

### <a name="internet-performance-considerations"></a>Consideraciones sobre el rendimiento de Internet

Como se describe en este artículo, hay factores en Internet y fuera del control de Azure que pueden afectar al rendimiento de la red. Estos son algunos de esos factores:

- **Latencia**: el tiempo de ida y vuelta entre dos destinos puede verse afectado por incidencias en redes intermedias, por tráfico que no toma la ruta de distancia "más corta" y por rutas de emparejamiento poco óptimas.

- **Pérdida de paquetes**: La pérdida de paquetes se puede deber a la congestión de la red, a incidencias de la ruta física y al déficit de rendimiento de los dispositivos de red.

- **Tamaño de MTU y fragmentación**: la fragmentación a lo largo de la ruta puede dar lugar a retrasos en la llegada de los datos o en a paquetes que llegan desordenados, lo que pueden afectar a la entrega de los paquetes.

Traceroute es una buena herramienta para medir las características de rendimiento de red (como la pérdida de paquetes o la latencia) a lo largo de cada ruta de red entre un dispositivo de origen y un dispositivo de destino.

### <a name="network-design-considerations"></a>Consideraciones de diseño de la red

Junto con los aspectos descritos anteriormente en este artículo, la topología de una red virtual puede afectar al rendimiento de la red. Por ejemplo, un diseño de concentrador y radios que retorna globalmente el tráfico a una red virtual de un solo concentrador provocará latencia de red, lo que afectará al rendimiento general de la red.

El número de dispositivos de red que atraviesa el tráfico de red también puede afectar a la latencia total. Por ejemplo, en un diseño de concentrador y radios, si el tráfico pasa a través de una aplicación virtual de red del radio y una aplicación virtual del concentrador antes del tránsito a Internet, las aplicaciones virtuales de red pueden introducir latencia.

### <a name="azure-regions-virtual-networks-and-latency"></a>Regiones de Azure, redes virtuales y latencia

Las regiones de Azure se componen de varios centros de datos dentro de un área geográfica general. Estos centros de datos podrían no estar físicamente juntos. En algunos casos les separan hasta 10 kilómetros. La red virtual es una superposición lógica sobre la red de centros de datos físicos de Azure. Una red virtual no implica una topología de red específica en el centro de datos.

Por ejemplo, dos máquinas virtuales que se encuentran en la misma red virtual y subred podrían estar en diferentes bastidores, filas o incluso centros de datos. Pueden estar separadas por metros de cable de fibra óptica o por kilómetros de cable de fibra óptica. Esta variación puede introducir una latencia variable (una diferencia de unos pocos milisegundos) entre diferentes máquinas virtuales.

La ubicación geográfica de las máquinas virtuales y la posible latencia resultante entre dos máquinas virtuales pueden verse influenciadas por la configuración de los conjuntos de disponibilidad y las zonas de disponibilidad. Pero la distancia entre los centros de datos de una región es específica de dicha región y viene influida principalmente por la topología de los centros de datos de la región.

### <a name="source-nat-port-exhaustion"></a>Agotamiento de puertos NAT de origen

Una implementación de Azure puede comunicarse con puntos de conexión externos a Azure en el espacio de direcciones IP públicas o en Internet público. Cuando una instancia inicia una conexión de salida, Azure asigna dinámicamente la dirección IP privada a una dirección IP pública. Una vez que Azure crea esta asignación, el tráfico de retorno del flujo originado de salida también puede comunicarse con la dirección IP privada donde se originó el flujo.

Para todas las conexiones salientes, Azure Load Balancer debe mantener esta asignación durante un período de tiempo. Dada la naturaleza multiinquilino de Azure, mantener esta asignación para cada flujo de salida de cada máquina virtual puede consumir muchos recursos. Por lo que se establecen límites que se basan en la configuración de Azure Virtual Network. O bien, para expresarlo con mayor precisión, una máquina virtual de Azure solo puede realizar un cierto número de conexiones salientes en un momento dado. Cuando se alcanzan estos límites, la máquina virtual no podrá realizar más conexiones salientes.

No obstante, este comportamiento es configurable. Para más información acerca de SNAT y el agotamiento de puertos SNAT, consulte [este artículo](https://docs.microsoft.com/azure/load-balancer/load-balancer-outbound-connections).

## <a name="measure-network-performance-on-azure"></a>Medición del rendimiento de red en Azure

Varios de los valores máximos de rendimiento de este artículo están relacionados con la latencia de red y el tiempo de ida y vuelta (RTT) entre dos máquinas virtuales. Esta sección proporciona algunas sugerencias para la prueba de latencia y RTT y sobre cómo probar el rendimiento de TCP y rendimiento de la red de la máquina virtual. Puede ajustar y probar el rendimiento de los valores de TCP/IP y de la red mencionados anteriormente mediante el uso de las técnicas descritas en esta sección. Puede insertar los valores de latencia, MTU, MSS y tamaño de la ventana en los cálculos que se proporcionaron anteriormente y comparar los valores máximos teóricos con los valores reales que se observan durante las pruebas.

### <a name="measure-round-trip-time-and-packet-loss"></a>Medición del tiempo de ida y vuelta y la pérdida de paquetes

El rendimiento de TCP depende en gran medida del RTT y la pérdida de paquetes. La utilidad PING disponible en Windows y Linux proporciona la manera más fácil de medir el RTT y la pérdida de paquetes. La salida de PING muestra la latencia mínima, máxima y promedio entre un origen y destino. También muestra la pérdida de paquetes. PING usa el protocolo ICMP de forma predeterminada. Puede usar PsPing para probar el RTT de TCP. Para más información, consulte [PsPing](https://docs.microsoft.com/sysinternals/downloads/psping).

### <a name="measure-actual-throughput-of-a-tcp-connection"></a>Medición del rendimiento real de una conexión TCP

NTttcp es una herramienta para probar el rendimiento de TCP de una máquina virtual Windows o Linux. Puede cambiar varios valores de TCP y, a continuación, probar las ventajas con NTttcp. Para obtener más información, vea estos recursos:

- [Pruebas de ancho de banda y rendimiento (NTttcp)](https://aka.ms/TestNetworkThroughput)

- [Utilidad NTttcp](https://gallery.technet.microsoft.com/NTttcp-Version-528-Now-f8b12769)

### <a name="measure-actual-bandwidth-of-a-virtual-machine"></a>Medición del ancho de banda real de una máquina virtual

Puede probar el rendimiento de diferentes tipos de máquinas virtuales, redes aceleradas y otros elementos mediante una herramienta llamada iPerf. iPerf también está disponible en Linux y Windows. iPerf puede usar TCP o UDP para probar el rendimiento general de la red. Las pruebas de rendimiento de TCP de iPerf se ven influenciadas por los factores explicados en este artículo (como la latencia y el RTT). Por tanto, UDP podría ofrecer mejores resultados si solo desea probar el rendimiento máximo.

Para obtener más información, consulte estos artículos:

- [Solución de problemas de rendimiento de red de ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-troubleshooting-network-performance)

- [Validación del rendimiento de la VPN en una red virtual](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-validate-throughput-to-vnet)

### <a name="detect-inefficient-tcp-behaviors"></a>Detección de comportamientos no eficaces de TCP

En las capturas de paquetes, los clientes de Azure podrían ver paquetes de TCP con marcas de TCP (SACK, DUP ACK, RETRANSMIT y FAST RETRANSMIT) que podrían indicar problemas de rendimiento de la red. Esos paquetes indican específicamente problemas de eficacia de la red como resultado de la pérdida de paquetes. Pero la pérdida de paquetes no es provocada necesariamente por problemas de rendimiento de Azure. Los problemas de rendimiento podrían ser el resultado de problemas de la aplicación, problemas del sistema operativo u otros problemas que podrían no estar directamente relacionados con la plataforma de Azure.

Además, tenga en cuenta que algunos elementos ACK de retransmisión o duplicados son normales en una red. Los protocolos TCP se diseñaron para ser confiables. La evidencia de estos paquetes TCP en una captura de paquetes no indica necesariamente un problema sistémico de la red a menos que sean excesivos.

Aún así, estos tipos de paquetes son indicaciones de que el rendimiento de TCP no alcanza su rendimiento máximo por las razones descritas en otras secciones de este artículo.

## <a name="next-steps"></a>Pasos siguientes

Ahora que tiene información sobre el ajuste del rendimiento de TCP/IP de las máquinas virtuales de Azure, tal vez quiera consultar otras consideraciones sobre el [planeamiento de redes virtuales](https://docs.microsoft.com/azure/virtual-network/virtual-network-vnet-plan-design-arm) u [obtener información sobre la conexión y configuración de redes virtuales](https://docs.microsoft.com/azure/virtual-network/).
