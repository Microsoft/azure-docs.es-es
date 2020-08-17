---
title: Conexiones salientes en Azure
titleSuffix: Azure Load Balancer
description: En este artículo se explica cómo Azure permite que las máquinas virtuales puedan comunicarse con servicios de la red Internet pública.
services: load-balancer
documentationcenter: na
author: asudbring
ms.service: load-balancer
ms.custom: seodec18
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/24/2020
ms.author: allensu
ms.openlocfilehash: a2292dc789938b8bde709728f5bbffe661529cc2
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87072629"
---
# <a name="outbound-connections-in-azure"></a>Conexiones salientes en Azure

Azure Load Balancer proporciona conectividad saliente mediante varios mecanismos. En este artículo se describen los escenarios y cómo administrarlos. Si tiene problemas con la conectividad saliente a través de Azure Load Balancer, vea la [guía de solución de problemas para conexiones salientes](../load-balancer/troubleshoot-outbound-connection.md).

>[!NOTE]
>En este artículo se describen las implementaciones de Resource Manager. Microsoft recomienda Resource Manager para las cargas de trabajo de producción.

## <a name="terminology"></a>Terminología

| Término | Protocolos aplicables | Detalles|
| ---------|---------| -------|
| Traducción de direcciones de red de origen (SNAT) | TCP, UDP | En una implementación de Azure es posible comunicarse con puntos de conexión externos a Azure en el espacio de direcciones IP públicas. Cuando una instancia inicia un flujo de salida a un destino del espacio de direcciones IP públicas, Azure asigna dinámicamente la dirección IP privada a una dirección IP pública. Una vez creada esta asignación, el tráfico de retorno de este flujo originado de salida también puede comunicarse con la dirección IP privada donde se originó el flujo. Azure usa la **traducción de direcciones de red de origen (SNAT)** para realizar esta función.|
| Arquitectura de redes de sistemas de enmascaramiento de puertos (PAT)| TCP, UDP |  Cuando se enmascaran varias direcciones IP privadas detrás de una única dirección IP pública, Azure usa la **traducción de direcciones de puerto (PAT)** para ocultar o enmascarar las direcciones IP privadas. En PAT se usan puertos efímeros y se [asignan previamente](#preallocatedports) en función del tamaño del grupo. Cuando un recurso de Load Balancer público está asociado con instancias de máquina virtual, que no tienen direcciones IP públicas dedicadas, se reescribe cada origen de conexión saliente. El origen se reescribe del espacio de direcciones IP privadas de la red virtual a la dirección IP pública de servidor front-end del equilibrador de carga. En el espacio de direcciones IP públicas, la tupla de cinco elementos del flujo (dirección IP de origen, puerto de origen, protocolo de transporte IP, dirección IP de destino, puerto de destino) debe ser única. SNAT de enmascaramiento de puertos se puede usar con los protocolos IP UDP o TCP. Para conseguir esto, se usan puertos efímeros (puertos SNAT) después de volver a escribir la dirección IP de origen privada, dado que varios flujos se originan desde una única dirección IP pública. El algoritmo SNAT de enmascaramiento de puertos asigna los puertos SNAT de forma diferente para UDP que para TCP.|
| Puertos SNAT| TCP | Los puertos SNAT son puertos efímeros disponibles para una determinada dirección IP de origen pública. Se consume un puerto SNAT por cada flujo a una combinación única de dirección IP y puerto de destino. En el caso de varios flujos TCP a la misma dirección IP, puerto y protocolo de destino, cada flujo TCP consume un solo puerto SNAT. Esto garantiza que los flujos son únicos si se han originado desde la misma dirección IP pública y se dirigen a la misma dirección IP, puerto y protocolo de destino. Si hay varios flujos, cada uno de ellos dirigido a una dirección IP, un puerto y un protocolo de destino diferentes, se comparte un solo puerto SNAT. La dirección IP, el puerto y el protocolo de destino hacen que los flujos sean únicos, sin necesidad de usar puertos de origen adicionales para distinguirlos en el espacio de direcciones IP públicas.|
|Puertos SNAT | UDP | Los puertos UDP SNAT se administran mediante un algoritmo diferente que los puertos TCP SNAT.  Load Balancer utiliza un algoritmo que se conoce como "NAT de cono restringido de puertos" para UDP.  Se consume un puerto SNAT por cada flujo, con independencia de la combinación de dirección IP y puerto de destino.|
| Agotamiento | - | Cuando se agotan los recursos de los puertos SNAT, los flujos de salida generan errores hasta que los flujos ya existentes liberan puertos SNAT. Load Balancer reclama puertos SNAT cuando el flujo se cierra y usa un [tiempo de espera de inactividad de 4 minutos](../load-balancer/troubleshoot-outbound-connection.md#idletimeout) para reclamar puertos SNAT de los flujos inactivos. Los puertos UDP SNAT suelen agotarse mucho más rápidamente que los puertos TCP SNAT debido a la diferencia en el algoritmo utilizado. Debe tener presente esta diferencia al realizar pruebas de diseño y escalado.|
| Comportamiento de liberación de puertos SNAT | TCP | Si tanto el cliente como el servidor envían FIN/ACK, el puerto SNAT se liberará después de 240 segundos. Si se ve un RST, el puerto SNAT se liberará después de 15 segundos. Si se agotó el tiempo de expiración, el puerto se libera.|
| Comportamiento de liberación de puertos SNAT | UDP |Si se agotó el tiempo de expiración, el puerto se libera.|
| Reutilización del puerto SNAT | TCP, UDP | Una vez que se ha liberado un puerto, está disponible para reutilizarse según sea necesario.  Puede pensar en los puertos SNAT como una secuencia del menor al mayor disponibles para un escenario determinado. El primer puerto SNAT disponible se usa para las nuevas conexiones.|

### <a name="port-allocation-algorithm"></a><a name="preallocatedports"></a>Algoritmo de asignación de puertos

Azure usa un algoritmo para determinar el número de puertos SNAT asignados previamente disponibles en función del tamaño del grupo de back-end cuando se usa PAT. Para cada dirección IP pública asociada a un equilibrador de carga, hay 64 000 puertos disponibles como puertos SNAT para cada protocolo de transporte IP. Está preasignado el mismo número de puertos SNAT para UDP y TCP respectivamente, y se consumen independiente para cada protocolo de transporte de IP.  Aunque la utilización del puerto SNAT varía en función de que el flujo sea UDP o TCP. Cuando se crean flujos de salida, estos puertos se consumen de forma dinámica (hasta el límite asignado previamente) y se liberan cuando el flujo se cierra o se producen [tiempos de espera de inactividad](../load-balancer/troubleshoot-outbound-connection.md#idletimeout). Solo se consumen puertos si es necesario que los flujos sean únicos.

#### <a name="default-snat-ports-allocated"></a><a name="snatporttable"></a> Puertos SNAT predeterminados asignados

En la tabla siguiente se muestran las asignaciones previas de puertos SNAT para los niveles de tamaño de grupo de servidores back-end:

| Tamaño del grupo (instancias de máquina virtual) | Puertos SNAT asignados previamente por configuración IP |
| --- | --- |
| 1-50 | 1024 |
| 51-100 | 512 |
| 101-200 | 256 |
| 201-400 | 128 |
| 401-800 | 64 |
| 801-1000 | 32 |

El cambio del tamaño del grupo de back-end puede afectar a algunos de los flujos establecidos:

- Si el tamaño del grupo de servidores back-end aumenta y pasa al siguiente nivel, la mitad de los puertos SNAT previamente asignados se reclaman durante la transición al siguiente nivel más grande del grupo de servidores back-end. Los flujos que están asociados a un puerto SNAT reclamado agotan el tiempo de espera y deben restablecerse. Si se intenta un nuevo flujo, el flujo se inicia inmediatamente siempre y cuando los puertos asignados previamente estén disponibles.
- Si el tamaño del grupo de servidores back-end se reduce y pasa a un nivel inferior, aumenta el número de puertos SNAT disponibles. En este caso, los puertos SNAT asignados existentes y sus flujos respectivos no se ven afectados.

## <a name="outbound-connections-scenario-overview"></a><a name="scenarios"></a>Información general sobre el escenario de conexiones salientes

| Escenario | Método | Protocolos IP | Descripción |
|  --- | --- | --- | --- |
|  1. Máquina virtual con un dirección IP pública (con o sin Azure Load Balancer) | SNAT, no se usa el enmascaramiento de puertos | TCP, UDP, ICMP, ESP | Azure usa la dirección IP pública asignada a la configuración IP de la NIC de la instancia para todos los flujos de salida. La instancia tiene disponibles todos los puertos efímeros. Es irrelevante si la máquina virtual es de carga equilibrada o no. Este escenario tiene prioridad sobre las demás. Una dirección IP pública asignada a una máquina virtual es una relación 1:1 (en lugar de 1:muchos) y se implementa como NAT de 1:1 sin estado. |
| 2. Load Balancer público asociado a una máquina virtual (ninguna dirección IP pública en la instancia o máquina virtual) | SNAT con enmascaramiento de puertos (PAT) mediante los servidores front-end de Load Balancer | TCP, UDP | En este escenario, el recurso Load Balancer se debe configurar con una regla de equilibrador de carga que cree un vínculo entre el front-end de dirección IP pública y el grupo de back-end. Si no configura esta regla, el comportamiento es el que se describe en el escenario 3. No es necesario que la regla tenga un agente de escucha en funcionamiento en el grupo de servidores back-end o que el sondeo de mantenimiento sea correcto. Cuando la máquina virtual crea un flujo de salida, Azure traduce la dirección IP de origen privada del flujo de salida en la dirección IP pública del front-end público de Load Balancer a través de SNAT. Para distinguir los flujos individuales que se originan en la máquina virtual, se usan puertos efímeros de la dirección IP pública de front-end del equilibrador de carga. Cuando se crean flujos de salida, SNAT usa dinámicamente los [puertos efímeros asignados previamente](#preallocatedports). En este contexto, los puertos efímeros usados para SNAT se conocen como puertos SNAT. Los puertos SNAT se asignan previamente, como se describe en la [tabla de puertos SNAT asignados predeterminados](#snatporttable). |
| 3. Máquina virtual (sin Load Balancer, sin dirección IP pública) o máquina virtual asociada con Load Balancer interno básico | SNAT con enmascaramiento de puertos (PAT) | TCP, UDP | Cuando la máquina virtual crea un flujo de salida, Azure traduce la dirección IP de origen privado del flujo de salida en una dirección IP de origen público. Esta dirección IP pública **no es configurable**, no se puede reservar y no cuenta para el límite de recursos de IP pública de la suscripción. Si vuelve a implementar la máquina virtual, el conjunto de disponibilidad o el conjunto de escalado de máquinas virtuales, esta dirección IP pública se publicará y se solicitará una nueva dirección IP pública. No use este escenario para direcciones IP permitidas. En su lugar, use el escenario 1 o 2 donde se declara explícitamente el comportamiento de salida. Los puertos SNAT se asignan previamente, como se describe en la [tabla de puertos SNAT asignados predeterminados](#snatporttable).

## <a name="outbound-rules"></a><a name="outboundrules"></a>Reglas de salida

 Las reglas de salida simplifican la configuración de la traducción de direcciones de red públicas de salida de [Standard Load Balancer](load-balancer-standard-overview.md).  Tiene control declarativo completo sobre la conectividad de salida para escalar y ajustar esta conectividad según sus necesidades específicas. En esta sección se expande el escenario 2 (B) descrito anteriormente.

![Reglas de salida de Load Balancer](media/load-balancer-outbound-rules-overview/load-balancer-outbound-rules.png)

Con las reglas de salida, puede usar Load Balancer para definir la NAT de salida desde cero. También puede escalar y ajustar el comportamiento de la NAT de salida existente.

Las reglas de salida permiten controlar:

- qué máquinas virtuales se deben traducir para qué direcciones IP públicas.
- cómo se deben asignar los puertos de SNAT de salida.
- para qué protocolos se proporciona traducción de salida.
- qué duración se debe usar para el tiempo de espera de inactividad de conexión (de 4 a 120 minutos).
- si quiere enviar un restablecimiento de TCP en caso de tiempo de espera de inactividad.
- protocolos de transporte TCP y UDP con una sola regla

### <a name="outbound-rule-definition"></a>Definición de la regla de salida

Como sucede con el resto de reglas de Load Balancer, las de salida siguen la misma sintaxis conocida que las reglas de NAT de entrada y de equilibrio de carga: **front-end** + **parámetros** + **grupo de back-end**. Una regla de salida configura una NAT de salida para _todas las máquinas virtuales identificadas por el grupo de back-end_ que se deben traducir para el _front-end_.  Los _parámetros_ proporcionan un control más preciso sobre el algoritmo de NAT de salida.

### <a name="scale-outbound-nat-with-multiple-ip-addresses"></a><a name="scale"></a> NAT de salida de escala con varias direcciones IP

Cada dirección IP adicional que proporciona un front-end ofrece 64 000 puertos efímeros adicionales que Load Balancer puede usar como puertos SNAT. Puede usar varias direcciones IP para planear escenarios de gran escala y puede usar reglas de salida para mitigar los patrones con tendencia a [agotamiento de SNAT](troubleshoot-outbound-connection.md#snatexhaust).  

También puede usar un [prefijo de IP pública](https://aka.ms/lbpublicipprefix) directamente con una regla de salida.  El uso de un prefijo de dirección IP pública proporciona un escalado más fácil y una creación simplificada de listas de permitidos para los flujos que se originan en la implementación de Azure. Puede configurar una configuración de IP de front-end en el recurso de Load Balancer para hacer referencia directamente a un prefijo de dirección IP pública.  Esto permite el control exclusivo de Load Balancer sobre el prefijo IP público y la regla de salida usará automáticamente todas las direcciones IP públicas que contiene el prefijo de IP pública para las conexiones de salida.  Cada una de las direcciones IP dentro del prefijo de IP pública proporciona 64 000 puertos efímeros adicionales por dirección IP que Load Balancer puede usar como puertos SNAT.

### <a name="outbound-flow-idle-timeout-and-tcp-reset"></a><a name="idletimeout"></a> Tiempo de espera de inactividad de flujo de salida y restablecimiento de TCP

Las reglas de salida proporcionan un parámetro de configuración para controlar el tiempo de espera de inactividad del flujo de salida y que se corresponda con las necesidades de su aplicación. El tiempo de espera de inactividad de salida predeterminado es de 4 minutos. Puede aprender a [configurar tiempos de espera de inactividad](load-balancer-tcp-idle-timeout.md#tcp-idle-timeout). De forma predeterminada, Load Balancer anula el flujo en silencio cuando se alcanza el tiempo de espera de inactividad de salida.  Con el parámetro `enableTCPReset`, puede habilitar un comportamiento de la aplicación más predecible y controlar si se debe enviar un restablecimiento de TCP bidireccional (TCP RST) al agotarse del tiempo de espera de inactividad de salida. Revise [Restablecimiento de TCP al agotarse el tiempo de espera de inactividad](https://aka.ms/lbtcpreset) para obtener detalles como la disponibilidad por regiones.

### <a name="preventing-outbound-connectivity"></a><a name="preventoutbound"></a>Impedir la conectividad saliente

Las reglas de equilibrio de carga proporcionan programación automática de NAT de salida. Sin embargo, algunos escenarios requieren que deshabilite la programación automática de NAT de salida por parte de la regla de equilibrio de carga para permitirle controlar o refinar el comportamiento, o bien mejoran al hacerlo.  
Puede usar este parámetro de dos maneras:

1. Supresión opcional del uso de la dirección IP de entrada para SNAT de salida mediante la deshabilitación de SNAT de salida para una regla de equilibrio de carga
  
2. Ajuste los parámetros de SNAT de salida de una dirección IP que se usa para el tráfico de entrada y salida al mismo tiempo.  La programación de NAT de salida automática debe deshabilitarse para permitir que una regla de salida tome el control.  Por ejemplo, para cambiar la asignación de puertos SNAT de una dirección que también se usa para la entrada, el parámetro `disableOutboundSnat` se debe establecer en true.  Si intenta usar una regla de salida para volver a definir los parámetros de una dirección IP que también se usa para la entrada y no ha liberado la programación de NAT saliente de la regla de equilibrio de carga, se producirá un error en la operación para configurar una regla de salida.

>[!IMPORTANT]
> La máquina virtual no tendrá conectividad de salida si este parámetro se establece en true y no tiene una regla de salida para definir la conectividad de salida.  Algunas operaciones de la VM o la aplicación pueden depender de la conectividad de salida disponible. Asegúrese de comprender las dependencias del escenario y de haber considerado el impacto de hacer este cambio.

En ocasiones, no es aconsejable permitir que una máquina virtual cree un flujo de salida. O bien, puede que exista un requisito para administrar a qué destinos se puede llegar con los flujos de salida o qué destinos pueden comenzar los flujos de entrada. En este caso, puede usar los [grupos de seguridad de red](../virtual-network/security-overview.md) para administrar los destinos a los que puede llegar la máquina virtual. También puede usar los NSG para administrar qué destino público puede iniciar los flujos de entrada.

Cuando aplique un grupo de seguridad de red a una máquina virtual de carga equilibrada, preste atención a las [etiquetas de servicio](../virtual-network/security-overview.md#service-tags) y a las [reglas de seguridad predeterminadas](../virtual-network/security-overview.md#default-security-rules). Debe asegurarse de que la máquina virtual puede recibir solicitudes de sondeo de mantenimiento desde Azure Load Balancer.

Si un grupo de seguridad de red bloquea las solicitudes de sondeo de mantenimiento de la etiqueta predeterminada AZURE_LOADBALANCER, se producirá un error en el sondeo de mantenimiento de la máquina virtual y esta se marca como inactiva. Load Balancer dejará de enviar nuevos flujos a esa máquina virtual.

## <a name="scenarios-with-outbound-rules"></a>Escenarios con reglas de salida

| # | Escenario| Detalles|
|---|---|---|
| I | Limpieza de las conexiones de salida en un conjunto específico de direcciones IP públicas| Puede usar una regla de salida para limpiar las conexiones de salida para que parezca que se originan en un conjunto concreto de direcciones IP públicas a fin de facilitar los escenarios de creación de listas de permitidos.  Esta dirección IP pública de origen puede ser la misma que usa una regla de equilibrio de carga u otro conjunto de direcciones IP públicas distinto del que usa la regla de equilibrio de carga.  1. Cree un [prefijo IP público](https://aka.ms/lbpublicipprefix) (o direcciones IP públicas a partir de un prefijo de IP pública). 2. Cree una instancia pública de Standard Load Balancer. 3. Cree servidores front-end que hagan referencia al prefijo de IP pública (o direcciones IP públicas) que quiera usar. 4. Reutilice un grupo de back-end o cree uno, y coloque las VM en un grupo de back-end de la instancia pública de Load Balancer. 5. Configure una regla de salida en la instancia pública de Load Balancer para programar una NAT de salid para estas VM mediante los front-end. Si no quiere usar la regla de equilibrio de carga para la salida, debe deshabilitar la SNAT de salida en la regla de equilibrio de carga.
| II | Modificación de la asignación de puertos SNAT| Puede utilizar reglas de salida para ajustar la [asignación de puertos de SNAT automática basada en el tamaño del grupo de back-end](load-balancer-outbound-connections.md#preallocatedports). Por ejemplo, si tiene dos máquinas virtuales que comparten una única dirección IP pública de NAT de salida, puede aumentar el número de puertos SNAT asignados a de los 1024 puertos predeterminados si experimenta agotamiento de SNAT. Cada dirección IP pública puede contribuir hasta a 64 000 puertos efímeros.  Si configura una regla de salida con un único front-end de dirección IP pública, puede distribuir un total de 64 000 puertos SNAT a VM del grupo de back-end.  Para dos VM, se puede asignar un máximo de 32 000 puertos SNAT con una regla de salida (2 x 32 000 = 64 000). Puede usar reglas de salida para ajustar los puertos SNAT asignados de forma predeterminada. Puede asignar más o menos puertos SNAT de los que proporciona la asignación predeterminada. Cada dirección IP pública de todos los front-end de una regla de salida aporta hasta 64 000 puertos efímeros para usarlos como puertos SNAT.  Load Balancer asigna puertos SNAT en múltiplos de 8. Si proporciona un valor que no se puede dividir por 8, se rechaza la operación de configuración.  Si intenta asignar más puertos SNAT de los disponibles en función del número de direcciones IP públicas, se rechaza la operación de configuración.  Por ejemplo, si asigna 10 000 puertos por VM y 7 VM de un grupo de back-end tienen que compartir una única dirección IP pública, la configuración se rechaza (7 x 10 000 puertos SNAT > 64 000 puertos SNAT).  Puede agregar más direcciones IP públicas al front-end de la regla de salida para habilitar el escenario. Puede volver a la [asignación de puertos SNAT predeterminada basada en el tamaño del grupo de back-end](load-balancer-outbound-connections.md#preallocatedports) si especifica 0 como el número de puertos. En ese caso, las primeras 50 instancias de máquina virtual obtendrán 1024 puertos, las instancias de máquina virtual de la 51 a la 100 obtendrán 512 y así sucesivamente según la [tabla](#snatporttable).|
| III| Habilitación de solo la salida | Puede usar una instancia pública de Standard Load Balancer para proporcionar NAT de salida para un grupo de VM. En este escenario, puede usar una sola regla de salida, sin necesidad de reglas adicionales.|
| IV | NAT de salida solo para VM (no de entrada) | Defina una instancia pública de Standard Load Balancer, coloque las VM en el grupo de back-end y configure una regla de salida para programar la NAT de salida y limpiar las conexiones de salida para que se originen desde una dirección IP pública específica. También puede usar un prefijo IP público para simplificar la inclusión del origen de las conexiones de salida en una lista blanca. 1. Cree una instancia pública de Standard Load Balancer. 2. Cree un grupo de back-end y coloque las VM en un grupo de back-end de la instancia pública de Load Balancer. 3. Configure una regla de salida en la instancia pública de Load Balancer para programar una NAT de salid para estas VM.
| V| NAT de salida para escenarios internos de Standard Load Balancer| Cuando se usa una instancia de Standard Load Balancer, la NAT de salida no está disponible hasta que la conectividad de salida se haya declarado explícitamente. Puede definir la conectividad de salida utilizando una regla de salida para crear conectividad de salida para las máquinas virtuales detrás de una instancia interna de Standard Load Balancer con los pasos siguientes: 1. Cree una instancia pública de Standard Load Balancer. 2. Cree un grupo de back-end y coloque las máquinas virtuales en un grupo de back-end de la instancia pública de Load Balancer además de la instancia interna de Load Balancer. 3. Configure una regla de salida en la instancia pública de Load Balancer para programar una NAT de salid para estas VM.|
| VI | Habilitar los protocolos TCP y UDP para NAT de salida con una instancia pública de Standard Load Balancer | Cuando se usa una instancia pública de Standard Load Balancer, la programación de NAT de salida automática proporcionada coincide con el protocolo de transporte de la regla de equilibrio de carga. 1. Deshabilite la SNAT de salida en la regla de equilibrio de carga. 2. Configure una regla de salida en la misma instancia de Load Balancer. 3. Vuelva a usar el grupo de back-end que usó para sus VM. 4. Especifique "protocol": "All" como parte de la regla de salida. Si solo se usan reglas de NAT entrantes, no se proporciona ninguna NAT de salida. 1. Coloque las VM en un grupo de back-end. 2. Defina una o varias configuraciones de IP de front-end con direcciones IP públicas o un prefijo de IP pública. 3. Configure una regla de salida en la misma instancia de Load Balancer. 4. Especifique "protocol": "All" como parte de la regla de salida. |


## <a name="limitations"></a>Limitaciones

- El número máximo de puertos efímeros posibles por dirección IP de front-end es 64 000.
- El intervalo de tiempo de espera de inactividad de salida que puede configurar oscila entre 4 y 120 minutos (de 240 a 7200 segundos).
- Load Balancer no es compatible con ICMP para NAT de salida.
- Las reglas de salida solo se pueden aplicar a la configuración de IP principal de una NIC.  Se admiten varias NIC.
- Solo se puede acceder a los roles de trabajo web sin una red virtual y otros servicios de plataforma de Microsoft si se usa un equilibrador de carga estándar debido a un efecto secundario del funcionamiento de los servicios previos a la red virtual y otros servicios de plataforma. No dependa de este efecto secundario, porque el propio servicio o la plataforma subyacente pueden cambiar sin previo aviso. Siempre debe pensar que necesita crear conectividad de salida de manera explícita si lo desea al usar solo un equilibrador de carga estándar interno. El escenario 3 que se describe en este artículo no está disponible.

## <a name="next-steps"></a>Pasos siguientes

- Más información acerca de [Load Balancer Estándar](load-balancer-standard-overview.md).
- Vea las [preguntas más frecuentes sobre Azure Load Balancer](load-balancer-faqs.md).
- Obtenga más información sobre las [reglas de salida](load-balancer-outbound-rules-overview.md) para la instancia pública de Load Balancer estándar.
- Más información acerca de [Load Balancer](load-balancer-overview.md).
- Más información sobre los [grupos de seguridad de red](../virtual-network/security-overview.md).
- Aprenda sobre las demás [funcionalidades de red](../networking/networking-overview.md) clave en Azure.
