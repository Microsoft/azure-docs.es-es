---
title: Prueba del rendimiento de la red de una máquina virtual de Azure
titlesuffix: Azure Virtual Network
description: Use NTTTCP para establecer como destino la red para las pruebas y minimizar el uso de otros recursos que podrían afectar al rendimiento.
services: virtual-network
documentationcenter: na
author: steveesp
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/21/2017
ms.author: steveesp
ms.openlocfilehash: e5aa2c1c51fccddc3fb62d7ebdbadee19a2b093e
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87265184"
---
# <a name="bandwidththroughput-testing-ntttcp"></a>Pruebas de ancho de banda y rendimiento (NTTTCP)

Al probar el rendimiento de la red de Azure, se recomienda usar una herramienta que tenga como destino de la red para realizar pruebas y minimizar el uso de otros recursos que podrían afectar al rendimiento. Se recomienda NTTTCP.

Copie la herramienta en dos máquinas virtuales de Azure del mismo tamaño. Una máquina virtual funciona como remitente y la otra como receptora.

#### <a name="deploying-vms-for-testing"></a>Implementación de máquinas virtuales para pruebas
Para los fines de esta prueba, las dos máquinas virtuales deben tener el mismo servicio en la nube o el mismo conjunto de disponibilidad para que podamos usar sus direcciones IP internas y excluir los equilibradores de carga de la prueba. Es posible realizar pruebas con la dirección VIP, pero este tipo de pruebas está fuera del ámbito de este documento.

Tome nota de la dirección IP del DESTINATARIO. Vamos a llamar a esa dirección IP "a.b.c.r".

Tome nota del número de núcleos de la máquina virtual. Llamémoslo "\#num\_cores".

Ejecute la prueba NTTTCP durante 300 segundos (o 5 minutos) en la máquina virtual del remitente y en la del receptor.

Sugerencia: Al configurar esta prueba por primera vez, puede probar un período de prueba más corto para obtener antes comentarios. Una vez que la herramienta funciona según lo esperado, amplíe el período de prueba en 300 segundos para obtener resultados más precisos.

> [!NOTE]
> El remitente **y** el receptor deben especificar **el mismo** parámetro de duración de prueba (-t).

Para probar un único flujo TCP durante 10 segundos:

Parámetros de receptor: ntttcp -r -t 10 -P 1

Parámetros de remitente: ntttcp -s10.27.33.7 -t 10 -n 1 -P 1

> [!NOTE]
> El ejemplo anterior solo debe emplearse para confirmar la configuración. Más adelante en este documento se ofrecen ejemplos válidos de pruebas.

## <a name="testing-vms-running-windows"></a>Pruebas de máquinas virtuales que ejecutan WINDOWS:

#### <a name="get-ntttcp-onto-the-vms"></a>Obtenga NTTTCP en las máquinas virtuales.

Descargue la versión más reciente: <https://gallery.technet.microsoft.com/NTttcp-Version-528-Now-f8b12769>

O búsquela si se ha migrado: <https://www.bing.com/search?q=ntttcp+download>\< (debe ser el primer resultado)

Considere la colocar NTTTCP en una carpeta independiente, como c:\\tools.

#### <a name="allow-ntttcp-through-the-windows-firewall"></a>Procedimiento para permitir NTTTCP a través del Firewall de Windows
En el RECEPTOR, cree una regla Permitir en el Firewall de Windows para permitir la recepción de tráfico NTTTCP. Es más fácil permitir todo el programa NTTTCP por su nombre en lugar de determinados puertos TCP de entrada.

Permita NTTTCP a través del Firewall de Windows de la siguiente forma:

netsh advfirewall firewall add rule program=\<PATH\>\\ntttcp.exe name="ntttcp" protocol=any dir=in action=allow enable=yes profile=ANY

Por ejemplo, si ha copiado ntttcp.exe a la carpeta "c:\\tools", este sería el comando: 

netsh advfirewall firewall add rule program=c:\\tools\\ntttcp.exe name="ntttcp" protocol=any dir=in action=allow enable=yes profile=ANY

#### <a name="running-ntttcp-tests"></a>Ejecución de pruebas de NTTTCP

Inicie NTTTCP en el RECEPTOR (**se ejecuta desde CMD**, no desde PowerShell):

ntttcp -r –m [2\*\#num\_cores],\*,a.b.c.r -t 300

Si la máquina virtual tiene cuatro núcleos y una dirección IP 10.0.0.4, sería similar al siguiente:

ntttcp -r –m 8,\*,10.0.0.4 -t 300


Inicie NTTTCP en el REMITENTE (**se ejecuta desde CMD**, no desde PowerShell)::

ntttcp -s –m 8,\*,10.0.0.4 -t 300 

Espere a que se muestren los resultados.


## <a name="testing-vms-running-linux"></a>Pruebas de máquinas virtuales que ejecutan LINUX:

Use nttcp-for-linux. Está disponible en <https://github.com/Microsoft/ntttcp-for-linux>.

En las máquinas virtuales de Linux (REMITENTE y RECEPTOR), ejecute estos comandos para preparar ntttcp-for-linux en las máquinas virtuales:

CentOS: instalación de Git:
``` bash
  yum install gcc -y  
  yum install git -y
```
Ubuntu: instalación de Git:
``` bash
 apt-get -y install build-essential  
 apt-get -y install git
```
Cree e instale en ambas:
``` bash
 git clone https://github.com/Microsoft/ntttcp-for-linux
 cd ntttcp-for-linux/src
 make && make install
```

Como en el ejemplo de Windows, supongamos que la IP del RECEPTOR Linux es 10.0.0.4.

Inicie ntttcp-for-linux en el RECEPTOR:

``` bash
ntttcp -r -t 300
```

Y en el REMITENTE, ejecute:

``` bash
ntttcp -s10.0.0.4 -t 300
```
 
Los valores predeterminados de duración de prueba están establecidos en 60 segundos si no hay ningún parámetro de tiempo.

## <a name="testing-between-vms-running-windows-and-linux"></a>Pruebas entre máquinas virtuales que ejecutan Windows y Linux:

En estos escenarios, se debería habilitar el modo sin sincronización para que la prueba pueda ejecutarse. Para ello, use **-N flag** para Linux y **-ns flag** para Windows.

#### <a name="from-linux-to-windows"></a>De Linux a Windows:

Receptor\<Windows>:

``` bash
ntttcp -r -m <2 x nr cores>,*,<Windows server IP>
```

Remitente\<Linux> :

``` bash
ntttcp -s -m <2 x nr cores>,*,<Windows server IP> -N -t 300
```

#### <a name="from-windows-to-linux"></a>De Windows a Linux:

Receptor\<Linux>:

``` bash
ntttcp -r -m <2 x nr cores>,*,<Linux server IP>
```

Remitente\<Windows>:

``` bash
ntttcp -s -m <2 x nr cores>,*,<Linux  server IP> -ns -t 300
```
## <a name="testing-cloud-service-instances"></a>Prueba de instancias de servicio en la nube:
Debe agregar la siguiente sección en ServiceDefinition.csdef.
```xml
<Endpoints>
  <InternalEndpoint name="Endpoint3" protocol="any" />
</Endpoints> 
```

## <a name="next-steps"></a>Pasos siguientes
* Dependiendo de los resultados, puede que haya espacio para [optimizar máquinas de rendimiento de red](virtual-network-optimize-network-bandwidth.md) para su escenario.
* Obtenga información acerca de cómo [se asigna el ancho de banda a las máquinas virtuales](virtual-machine-network-throughput.md)
* Obtenga más información con las [Preguntas más frecuentes (P+F) acerca de Azure Virtual Network](virtual-networks-faq.md)
