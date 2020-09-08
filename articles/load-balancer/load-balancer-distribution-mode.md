---
title: Configuración del modo de distribución de Azure Load Balancer
titleSuffix: Azure Load Balancer
description: En este artículo, empezará a configurar el modo de distribución de Azure Load Balancer para admitir la afinidad de IP de origen.
services: load-balancer
documentationcenter: na
author: asudbring
ms.service: load-balancer
ms.devlang: na
ms.topic: how-to
ms.custom: seodec18
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/19/2019
ms.author: allensu
ms.openlocfilehash: d6995073ba7d283a30f7580456cd61573752f7cc
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89051396"
---
# <a name="configure-the-distribution-mode-for-azure-load-balancer"></a>Configuración del modo de distribución de Azure Load Balancer

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="hash-based-distribution-mode"></a>Distribución basada en hash

El modo de distribución predeterminado para Azure Load Balancer es un hash de tupla de cinco elementos. 

La tupla consta de:
* **IP de origen**
* **Puerto de origen**
* **IP de destino**
* **Puerto de destino**
* **Tipo de protocolo**

El hash se utiliza para asignar el tráfico a los servidores disponibles. El algoritmo solo proporciona adherencia dentro de una sesión de transporte. Los paquetes que se encuentran en la misma sesión se dirigen a la misma dirección IP del centro de datos tras el punto de conexión con equilibrio de carga. Cuando el cliente inicia una nueva sesión desde la misma IP de origen, el puerto de origen cambia y provoca que el tráfico vaya hacia otro punto de conexión del centro de datos.

![Modo de distribución basada en un hash de tupla de cinco elementos](./media/load-balancer-distribution-mode/load-balancer-distribution.png)

## <a name="source-ip-affinity-mode"></a>Modo de afinidad de IP de origen

El equilibrador de carga también se puede configurar mediante el modo de distribución de afinidad de IP de origen. Este modo de distribución también se conoce como afinidad de la sesión o afinidad de IP del cliente. El modo utiliza un hash de tupla de dos elementos (IP de origen e IP de destino) o de tres elementos (IP de origen, IP de destino y tipo de protocolo) para asignar el tráfico a los servidores disponibles. Mediante el uso de la afinidad de la IP de origen, las conexiones que se han iniciado desde el mismo equipo cliente van al mismo punto de conexión del centro de datos.

En la figura siguiente, se ilustra la configuración de tupla de dos elementos. Observe cómo la tupla de dos elementos se ejecuta a través del equilibrador de carga en la máquina virtual 1 (VM1). A continuación, VM2 y VM3 realizan una copia de seguridad de VM1.

![Modo de distribución de afinidad de la sesión de una tupla de dos elementos](./media/load-balancer-distribution-mode/load-balancer-session-affinity.png)

El modo de afinidad de IP de origen resuelve una incompatibilidad entre Azure Load Balancer y la Puerta de enlace de Escritorio remoto (RD Gateway). Al utilizar este modo, puede crear una granja de servidores de Puerta de enlace de Escritorio remoto en un solo servicio en la nube.

Otro escenario de caso de uso es la carga de elementos multimedia. La carga de datos tiene lugar a través de UDP, pero el plano de control se consigue mediante TCP:

* Un cliente inicia una sesión TCP en la dirección pública de carga equilibrada y se le dirige a una DIP específica. El canal se mantiene activo para supervisar el estado de conexión.
* Se inicia una nueva sesión UDP desde el mismo equipo cliente al mismo punto de conexión público de carga equilibrada. La conexión se dirige al mismo punto de conexión DIP que la conexión TCP anterior. La carga de elementos multimedia se puede ejecutar a alto rendimiento al tiempo que mantiene un canal de control a través de TCP.

> [!NOTE]
> Cuando un conjunto de carga equilibrada cambia al quitar o agregar una máquina virtual, la distribución de las solicitudes de cliente se vuelve a calcular. No puede depender de nuevas conexiones desde clientes existentes para terminar en el mismo servidor. Además, el uso del modo de distribución de afinidad de IP de origen puede ocasionar una distribución desigual del tráfico. Los clientes que se ejecutan detrás de servidores proxy pueden considerarse como una sola aplicación cliente.

## <a name="configure-source-ip-affinity-settings"></a>Configuración de la afinidad de IP de origen

### <a name="azure-portal"></a>Portal de Azure

Para cambiar la configuración del modo de distribución, modifique la regla de equilibrio de carga en el portal.

1. Inicie sesión en Azure Portal y busque el grupo de recursos que contiene el equilibrador de carga que desea cambiar. Para ello, debe hacer clic en **Grupos de recursos**.
2. En la pantalla de información general del equilibrador de carga, haga clic en **Reglas de equilibrio de carga**  en **Configuración**.
3. En la pantalla de reglas de equilibrio de carga, haga clic en la regla de equilibrio de carga para la que desee cambiar el modo de distribución.
4. En la regla, para cambiar el modo de distribución hay que cambiar el cuadro desplegable **Persistencia de la sesión**.  Están disponibles las siguientes opciones:
    
    * **Ninguno (basado en hash)** : especifica que cualquier máquina virtual puede controlar las solicitudes sucesivas del mismo cliente.
    * **IP de cliente (dos tuplas de afinidad de IP de origen)** : especifica que la misma máquina virtual controlará las solicitudes sucesivas de la misma dirección IP de cliente.
    * **IP de cliente y protocolo (tres tuplas de afinidad de IP de origen)** : especifica que la misma máquina virtual controlará las solicitudes sucesivas de la combinación de la misma dirección IP de cliente y protocolo.

5. Elija el modo de distribución y haga clic en **Guardar**.

### <a name="azure-powershell"></a>Azure PowerShell

En máquinas virtuales implementadas con Resource Manager, use PowerShell para cambiar la configuración de distribución del equilibrador de carga en una regla de equilibrio de carga existente. El siguiente comando actualiza el modo de distribución: 

```azurepowershell-interactive
$lb = Get-AzLoadBalancer -Name MyLb -ResourceGroupName MyLbRg
$lb.LoadBalancingRules[0].LoadDistribution = 'sourceIp'
Set-AzLoadBalancer -LoadBalancer $lb
```

En máquinas virtuales clásicas, use Azure PowerShell para cambiar la configuración de distribución. Agregue un punto de conexión de Azure a una máquina virtual y configure el modo de distribución del equilibrador de carga:

```azurepowershell-interactive
Get-AzureVM -ServiceName mySvc -Name MyVM1 | Add-AzureEndpoint -Name HttpIn -Protocol TCP -PublicPort 80 -LocalPort 8080 –LoadBalancerDistribution sourceIP | Update-AzureVM
```

Establezca el valor del elemento `LoadBalancerDistribution` para la cantidad necesaria de equilibrio de carga. Especifique sourceIP para el equilibrio de carga con tupla de dos elementos (IP de origen e IP de destino). Especifique sourceIPProtocol para el equilibrio de carga con tupla de tres elementos (IP de origen, IP de destino y tipo de protocolo). Especifique none para el comportamiento predeterminado de equilibrio de carga con tupla de cinco elementos.

Recupere una configuración de modo de distribución del equilibrador de carga de punto de extremo mediante estos valores:

```azurepowershell
PS C:\> Get-AzureVM –ServiceName MyService –Name MyVM | Get-AzureEndpoint

VERBOSE: 6:43:50 PM - Completed Operation: Get Deployment
LBSetName : MyLoadBalancedSet
LocalPort : 80
Name : HTTP
Port : 80
Protocol : tcp
Vip : 65.52.xxx.xxx
ProbePath :
ProbePort : 80
ProbeProtocol : tcp
ProbeIntervalInSeconds : 15
ProbeTimeoutInSeconds : 31
EnableDirectServerReturn : False
Acl : {}
InternalLoadBalancerName :
IdleTimeoutInMinutes : 15
LoadBalancerDistribution : sourceIP
```

Cuando el elemento `LoadBalancerDistribution` no está presente, Azure Load Balancer usa el algoritmo de tupla de cinco elementos predeterminado.

### <a name="configure-distribution-mode-on-load-balanced-endpoint-set"></a>Configuración del modo de distribución en un conjunto de puntos de conexión de carga equilibrada

Cuando los puntos de conexión forman parte de un conjunto de puntos de conexión de carga equilibrada, el modo de distribución debe configurarse en el conjunto de puntos de conexión de carga equilibrada:

```azurepowershell-interactive
Set-AzureLoadBalancedEndpoint -ServiceName MyService -LBSetName LBSet1 -Protocol TCP -LocalPort 80 -ProbeProtocolTCP -ProbePort 8080 –LoadBalancerDistribution sourceIP
```

### <a name="configure-distribution-mode-for-cloud-services-endpoints"></a>Configuración del modo de distribución para puntos de conexión de Cloud Services

Utilice el SDK de Azure para .NET 2.5 para actualizar el servicio en la nube. Los valores del punto de conexión para Cloud Services se establecen en el archivo .csdef. Para actualizar el modo de distribución del equilibrador de carga para una implementación de Cloud Services, se requiere una actualización de la implementación.

Este es un ejemplo de los cambios de .csdef para la configuración de extremo:

```xml
<WorkerRole name="worker-role-name" vmsize="worker-role-size" enableNativeCodeExecution="[true|false]">
    <Endpoints>
    <InputEndpoint name="input-endpoint-name" protocol="[http|https|tcp|udp]" localPort="local-port-number" port="port-number" certificate="certificate-name" loadBalancerProbe="load-balancer-probe-name" loadBalancerDistribution="sourceIP" />
    </Endpoints>
</WorkerRole>
<NetworkConfiguration>
    <VirtualNetworkSite name="VNet"/>
    <AddressAssignments>
<InstanceAddress roleName="VMRolePersisted">
    <PublicIPs>
    <PublicIP name="public-ip-name" idleTimeoutInMinutes="timeout-in-minutes"/>
    </PublicIPs>
</InstanceAddress>
    </AddressAssignments>
</NetworkConfiguration>
```

## <a name="api-example"></a>Ejemplo de API

En el ejemplo siguiente se muestra cómo volver a configurar el modo de distribución del equilibrador de carga para un conjunto específico de carga equilibrada en una implementación. 

### <a name="change-distribution-mode-for-deployed-load-balanced-set"></a>Cambiar el modo de distribución para un conjunto de carga equilibrada implementado

Utilice el modelo de implementación clásica de Azure para cambiar una configuración de implementación existente. Agregue el `x-ms-version` encabezado y establezca el valor en la versión 2014-09-01 o posterior.

#### <a name="request"></a>Solicitud

```http
POST https://management.core.windows.net/<subscription-id>/services/hostedservices/<cloudservice-name>/deployments/<deployment-name>?comp=UpdateLbSet   x-ms-version: 2014-09-01
Content-Type: application/xml
```

```xml
<LoadBalancedEndpointList xmlns="http://schemas.microsoft.com/windowsazure" xmlns:i="https://www.w3.org/2001/XMLSchema-instance">
    <InputEndpoint>
        <LoadBalancedEndpointSetName> endpoint-set-name </LoadBalancedEndpointSetName>
        <LocalPort> local-port-number </LocalPort>
        <Port> external-port-number </Port>
        <LoadBalancerProbe>
            <Port> port-assigned-to-probe </Port>
            <Protocol> probe-protocol </Protocol>
            <IntervalInSeconds> interval-of-probe </IntervalInSeconds>
            <TimeoutInSeconds> timeout-for-probe </TimeoutInSeconds>
        </LoadBalancerProbe>
        <Protocol> endpoint-protocol </Protocol>
        <EnableDirectServerReturn> enable-direct-server-return </EnableDirectServerReturn>
        <IdleTimeoutInMinutes>idle-time-out</IdleTimeoutInMinutes>
        <LoadBalancerDistribution>sourceIP</LoadBalancerDistribution>
    </InputEndpoint>
</LoadBalancedEndpointList>
```

Como se ha descrito anteriormente, establezca el elemento `LoadBalancerDistribution` en sourceIP para la afinidad con tupla de dos elementos, sourceIPProtocol para la afinidad con tupla de tres elementos o none para los casos sin afinidad (afinidad con tupla de cinco elementos).

#### <a name="response"></a>Response

```http
HTTP/1.1 202 Accepted
Cache-Control: no-cache
Content-Length: 0
Server: 1.0.6198.146 (rd_rdfe_stable.141015-1306) Microsoft-HTTPAPI/2.0
x-ms-servedbyregion: ussouth2
x-ms-request-id: 9c7bda3e67c621a6b57096323069f7af
Date: Thu, 16 Oct 2014 22:49:21 GMT
```

## <a name="next-steps"></a>Pasos siguientes

* [Información general sobre el equilibrador de carga interno de Azure](load-balancer-internal-overview.md)
* [Introducción a la configuración de un equilibrador de carga accesible desde Internet](quickstart-load-balancer-standard-public-powershell.md)
* [Configuración de opciones de tiempo de espera de inactividad de TCP para el equilibrador de carga](load-balancer-tcp-idle-timeout.md)
