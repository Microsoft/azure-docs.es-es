---
title: Configuración del nombre de red distribuida (DNN)
description: Aprenda a configurar un nombre de red distribuida (DNN) para enrutar el tráfico al servidor de SQL Server en la instancia del clúster de conmutación por error de la máquina virtual de Azure.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
manager: jroth
tags: azure-resource-manager
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 06/02/2020
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 8eb9caf466148e43266c4be9cf1308da15fb67f2
ms.sourcegitcommit: c293217e2d829b752771dab52b96529a5442a190
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/15/2020
ms.locfileid: "88245543"
---
# <a name="configure-a-distributed-network-name-for-an-fci"></a>Configuración de un nombre de red distribuida para una FCI 
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

En Azure Virtual Machines, se usa el nombre de red distribuida (DNN) para enrutar el tráfico al recurso en el clúster adecuado. Proporciona una manera más fácil de conectarse a la instancia del clúster de conmutación por error de SQL Server que el nombre de red virtual (VNN), sin necesidad de Azure Load Balancer. Esta característica se encuentra actualmente en versión preliminar y solo está disponible para SQL Server 2019 CU2 y versiones posteriores, y Windows Server 2016 y versiones posteriores. 

En este artículo aprenderá a configurar un nombre de red distribuida para enrutar el tráfico a sus FCI con SQL Server en máquinas virtuales de Azure para lograr una alta disponibilidad y recuperación ante desastres (HADR). 

## <a name="prerequisites"></a>Requisitos previos

Antes de completar los pasos de este artículo, ya debe tener:

- Decidió que el nombre de red distribuida es la [opción de conectividad adecuada para su solución HADR](hadr-cluster-best-practices.md#connectivity).
- Configuró las [instancias de clúster de conmutación por error](failover-cluster-instance-overview.md). 
- Instaló la versión más reciente de [Azure PowerShell](/powershell/azure/install-az-ps). 

## <a name="create-dnn-resource"></a>Creación de un recurso DNN 

El recurso DNN se crea en el mismo grupo de clúster que la FCI de SQL Server. Use PowerShell para crear el recurso DNN en el grupo de clúster de la FCI. 

El siguiente comando de PowerShell agrega un recurso DNN al grupo de clústeres de la instancia de clúster de conmutación por error (FCI) de SQL Server con el nombre de recurso `<dnnResourceName>`. El nombre del recurso se utiliza para identificarlo de forma única. Use uno que le resulte útil y que sea único en todo el clúster. El tipo de recurso debe ser `Distributed Network Name`. 

El valor `-Group` debe ser el nombre del grupo de clústeres que corresponda a la FCI de SQL Server en el que desee agregar el nombre de red distribuida. En el caso de una instancia predeterminada, el formato típico es `SQL Server (MSSQLSERVER)`. 


```powershell
Add-ClusterResource -Name <dnnResourceName> `
-ResourceType "Distributed Network Name" -Group "<WSFC role of SQL server instance>"
```

Por ejemplo, para crear el recurso DNN `dnn-demo` para una FCI de SQL Server predeterminado, use el siguiente comando de PowerShell:

```powershell
Add-ClusterResource -Name dnn-demo `
-ResourceType "Distributed Network Name" -Group "SQL Server (MSSQLSERVER)"

```

## <a name="set-cluster-dnn-dns-name"></a>Establecimiento del nombre DNS del nombre de red distribuida del clúster

Establezca el nombre DNS del recurso DNN en el clúster. Después, el clúster usa este valor para enrutar el tráfico al nodo que hospeda actualmente la FCI de SQL Server. 
 
Los clientes utilizan el nombre DNS para conectarse a la FCI de SQL Server. Puede elegir un valor único. O bien, si ya tiene una FCI existente y no desea actualizar las cadenas de conexión de cliente, puede configurar el nombre de red distribuida para que use el nombre de la red virtual actual que los clientes ya usan. Para ello, debe [cambiar el nombre de red virtual](#rename-the-vnn) antes de establecer el nombre de red distribuida en DNS.


Use este comando para establecer el nombre DNS para el nombre de red distribuida: 

```powershell
Get-ClusterResource -Name <dnnResourceName> | `
Set-ClusterParameter -Name DnsName -Value <DNSName>
```

El valor `DNSName` es lo que los clientes usan para conectarse a la FCI de SQL Server. Por ejemplo, para que los clientes se conecten a `FCIDNN`, use el siguiente comando de PowerShell:

```powershell
Get-ClusterResource -Name dnn-demo | `
Set-ClusterParameter -Name DnsName -Value FCIDNN
```

Los clientes ahora escribirán `FCIDNN` en su cadena de conexión al conectarse a la FCI de SQL Server. 

   > [!WARNING]
   > No elimine el nombre de red virtual (VNN) actual, ya que es un componente necesario de la infraestructura de FCI. 


### <a name="rename-the-vnn"></a>Cambio del nombre de red virtual 

Si tiene un nombre de red virtual y desea que los clientes sigan usando este valor para conectarse a la FCI de SQL Server, debe cambiar el nombre de red virtual actual por un valor de marcador de posición. Después de cambiar el nombre de red virtual actual, puede establecer ese nombre como nombre DNS para el nombre de red distribuida. 

Al cambiar el nombre de red virtual, se aplican algunas restricciones. Para más información, consulte [Cambio del nombre de una FCI](/sql/sql-server/failover-clusters/install/rename-a-sql-server-failover-cluster-instance).

Si en su empresa no es necesario usar el nombre de red virtual actual, omita esta sección. Después de cambiar el nombre de red virtual, [establezca el nombre DNS del nombre de red distribuida del clúster](#set-cluster-dnn-dns-name). 

   
## <a name="set-dnn-resource-online"></a>Establecimiento del recurso DNN en línea

Una vez que el recurso DNN tenga el nombre apropiado y haya establecido el valor de nombre DNS en el clúster, use PowerShell para establecer el recurso DNN en línea en el clúster: 

```powershell
Start-ClusterResource -Name <dnnResourceName>
```

Por ejemplo, para iniciar el recurso DNN `dnn-demo`, use el siguiente comando de PowerShell: 

```powershell
Start-ClusterResource -Name dnn-demo
```

## <a name="configure-possible-owners"></a>Configuración de propietarios posibles

De forma predeterminada, el clúster enlaza el nombre DNS del nombre de red distribuida con todos los nodos del clúster. Sin embargo, los nodos del clúster que no forman parte de la FCI de SQL Server se deben excluir de la lista de propietarios posibles de DNN. 

Para actualizar los propietarios posibles, siga estos pasos:

1. Vaya al recurso DNN en el Administrador de clústeres de conmutación por error. 
1. Haga clic con el botón derecho en el recurso DNN y seleccione **Propiedades**. 
   :::image type="content" source="media/hadr-distributed-network-name-dnn-configure/fci-dnn-properties.png" alt-text="Menú contextual del recurso DNN, con el comando Propiedades resaltado.":::
1. Desactive la casilla de los nodos que no participen en la instancia del clúster de conmutación por error. La lista de propietarios posibles del recurso DNN debe coincidir con la del recurso de la instancia de SQL Server. Por ejemplo, suponiendo que Data3 no participe en la FCI, la imagen siguiente es un ejemplo de cómo quitar Data3 de la lista de propietarios posibles del recurso DNN: 

   :::image type="content" source="media/hadr-distributed-network-name-dnn-configure/clear-check-for-nodes-not-in-fci.png" alt-text="Desactivación de la casilla situada junto a los nodos que no participen en la FCI para los propietarios posibles del recurso DNN":::

1. Seleccione **Aceptar** para guardar la configuración. 


## <a name="restart-sql-server-instance"></a>Reinicio de la instancia de SQL Server 

Use el Administrador de clústeres de conmutación por error para reiniciar la instancia de SQL Server. Siga estos pasos:

1. Vaya a su recurso de SQL Server en el Administrador de clústeres de conmutación por error.
1. Haga clic con el botón derecho en el recurso de SQL Server y desconéctelo. 
1. Una vez que todos los recursos asociados estén desconectados, haga clic con el botón derecho en el recurso de SQL Server y vuelva a ponerlo en línea. 

## <a name="update-connection-string"></a>Actualización de la cadena de conexión

Para garantizar una conectividad rápida tras la conmutación por error, agregue `MultiSubnetFailover=True` a la cadena de conexión si la versión del cliente SQL es anterior a 4.6.1. 

Además, si el nombre de red distribuida no usa el nombre de red virtual original, los clientes SQL que se conecten a la FCI de SQL Server deberán actualizar su cadena de conexión con el nombre DNS del nombre de red distribuida. Para evitar este requisito, puede actualizar el valor del nombre DNS para que sea el nombre de red virtual. Sin embargo, primero tendrá que [reemplazar el nombre de red virtual existente con un marcador de posición](#rename-the-vnn). 

## <a name="test-failover"></a>Conmutación por error de prueba

Pruebe la conmutación por error del recurso en clúster para validar la funcionalidad del clúster. 

Para probar la conmutación por error, siga estos pasos: 

1. Conéctese a uno de los nodos de clúster de SQL Server con RDP.
1. Abra el **Administrador de clústeres de conmutación por error**. Seleccione **Roles**. Observe qué nodo posee el rol de FCI de SQL Server.
1. Haga clic con el botón derecho en el rol de FCI de SQL Server. 
1. Seleccione **Mover** y, después, seleccione **Mejor nodo posible**.

El **Administrador de clústeres de conmutación por error** muestra el rol y sus recursos pierden su conexión. Después, los recursos se mueven y vuelven a conectarse en el otro nodo.

## <a name="test-connectivity"></a>Comprobación de la conectividad

Para probar la conectividad, inicie sesión en otra máquina virtual de la misma red virtual. Abra **SQL Server Management Studio** y conéctese al nombre de la FCI de SQL Server con el nombre DNS de DNN.

Si es necesario, puede [descargar SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms).

## <a name="avoid-ip-conflict"></a>Evitar conflictos de IP

Este es un paso opcional para evitar que se asigne a otro recurso de Azure un duplicado de la dirección IP virtual (VIP) que usa el recurso FCI. 

Aunque los clientes ahora usan la DNN para conectarse a la FCI de SQL Server, el nombre de red virtual (VNN) y la dirección IP virtual no se pueden eliminar ya que son componentes necesarios de la infraestructura de FCI. Sin embargo, dado que ya no hay un equilibrador de carga que reserva la dirección IP virtual en Azure, existe el riesgo de que otro recurso de la red virtual tenga asignada la misma dirección IP que la que usa la FCI. Como consecuencia, podría producirse un problema de conflicto de IP duplicada. 

Para reservar la dirección IP, configure una dirección APIPA o un adaptador de red dedicado. 

### <a name="apipa-address"></a>Dirección APIPA

Para evitar el uso de direcciones IP duplicadas, configure una dirección APIPA (también conocida como dirección local de vínculo). Para ello, ejecute el siguiente comando:

```powershell
Get-ClusterResource "virtual IP address" | Set-ClusterParameter 
    –Multiple @{"Address”=”169.254.1.1”;”SubnetMask”=”255.255.0.0”;"OverrideAddressMatch"=1;”EnableDhcp”=0}
```

En este comando, "dirección IP virtual" es el nombre del recurso de dirección VIP en clúster y "169.254.1.1" es la dirección APIPA elegida para la dirección VIP. Elija la dirección que mejor se adapte a su negocio. Establezca `OverrideAddressMatch=1` para permitir que la dirección IP esté en cualquier red, incluido el espacio de direcciones APIPA. 

### <a name="dedicated-network-adapter"></a>Adaptador de red dedicado

También puede configurar un adaptador de red en Azure para reservar la dirección IP usada por el recurso de dirección IP virtual. Sin embargo, esta solución consume la dirección del espacio de direcciones de la subred y existe la sobrecarga adicional de garantizar que el adaptador de red no se use para ningún otro propósito.

## <a name="limitations"></a>Limitaciones

- Actualmente, el nombre de red distribuida se admite únicamente para SQL Server 2019 CU2 y versiones posteriores en Windows Server 2016. 
- Actualmente, el nombre de red distribuida solo se admite para las instancias de clúster de conmutación por error con SQL Server en máquinas virtuales de Azure. Use el nombre de red virtual con Azure Load Balancer para las escuchas del grupo de disponibilidad.
- Puede haber más consideraciones al trabajar con otras características de SQL Server y una FCI con un nombre de red distribuida. Para más información, consulte [Interoperabilidad de FCI con DNN](failover-cluster-instance-dnn-interoperability.md). 

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de las características HADR de SQL Server en Azure, consulte [Grupos de disponibilidad](availability-group-overview.md) e [Instancia de clúster de conmutación por error](failover-cluster-instance-overview.md). También puede consultar los [procedimientos recomendados](hadr-cluster-best-practices.md) para configurar el entorno para una alta disponibilidad y recuperación ante desastres. 

Puede haber requisitos de configuración adicionales para algunas características de SQL Server específicas cuando se usan con el nombre de red distribuida y la instancia de clúster de conmutación por error. Consulte [Interoperabilidad de FCI con DNN](failover-cluster-instance-dnn-interoperability.md) para más información. 

