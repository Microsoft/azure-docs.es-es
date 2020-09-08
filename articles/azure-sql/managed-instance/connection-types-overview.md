---
title: Tipos de conexión
titleSuffix: Azure SQL Managed Instance
description: Información acerca de los tipos de conexión de Instancia administrada de Azure SQL
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.topic: conceptual
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: vanto
ms.date: 10/07/2019
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: e46c6d1c14d226522a1d534418b91076efeaaccf
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89070724"
---
# <a name="azure-sql-managed-instance-connection-types"></a>Tipos de conexión de Instancia administrada de Azure SQL
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

En este artículo se explica cómo se conectan los clientes a Instancia administrada de Azure SQL en función del tipo de conexión. A continuación se proporcionan ejemplos de script para cambiar los tipos de conexión y consideraciones sobre la modificación de la configuración predeterminada de la conectividad.

## <a name="connection-types"></a>Tipos de conexión

Instancia administrada de Azure SQL admite los dos tipos de conexión siguientes:

- **Redirigir (recomendado):** los clientes establecen conexiones directamente con el nodo que hospeda la base de datos. Para habilitar la conectividad mediante el redireccionamiento, debe abrir los firewalls y los grupos de seguridad de red (NSG) para permitir el acceso a los puertos 1433 y 11000-11999. Los paquetes van directamente a la base de datos y, por lo tanto, hay mejoras de rendimiento de latencia y capacidad de proceso mediante el redireccionamiento en proxy.
- **Proxy (predeterminado)** : en este modo, todas las conexiones utilizan un componente de puerta de enlace de proxy. Para habilitar la conectividad, solo es necesario abrir el puerto 1433 para las redes privadas y el puerto 3342 para la conexión pública. La elección de este modo puede producir una latencia mayor y un rendimiento inferior, según la naturaleza de la carga de trabajo. Es muy recomendable utilizar la directiva de conexión de redireccionamiento para obtener la menor latencia y el mayor rendimiento.

## <a name="redirect-connection-type"></a>Tipo de conexión: redireccionamiento

En el tipo de conexión de redireccionamiento, una vez establecida la sesión TCP en el motor de SQL, la sesión de cliente obtiene la dirección IP virtual de destino del nodo de clúster virtual del equilibrador de carga. Los paquetes posteriores fluyen directamente al nodo de clúster virtual y omiten la puerta de enlace. En el siguiente diagrama, se ilustra este flujo de tráfico.

![redirect.png](./media/connection-types-overview/redirect.png)

> [!IMPORTANT]
> El tipo de conexión de redireccionamiento solo funciona actualmente para los puntos de conexión privados. Independientemente de la configuración del tipo de conexión, las conexiones que llegan por el punto de conexión público se realizarían mediante proxy.

## <a name="proxy-connection-type"></a>Tipo de conexión: proxy

En el tipo de conexión de proxy, la sesión TCP se establece con la puerta de enlace y todos los paquetes posteriores fluyen por esta. En el siguiente diagrama, se ilustra este flujo de tráfico.

![proxy.png](./media/connection-types-overview/proxy.png)

## <a name="changing-connection-type"></a>Cambio del tipo de conexión

- **Mediante el portal:** Para cambiar el tipo de conexión mediante Azure Portal, abra la página Red virtual y use el valor **Tipo de conexión** para cambiar el tipo de conexión y guardar los cambios.

- **Script para cambiar la configuración del tipo de conexión mediante PowerShell:**

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

El siguiente script de PowerShell muestra cómo cambiar el tipo de conexión a `Redirect` para una instancia administrada.

```powershell
Install-Module -Name Az
Import-Module Az.Accounts
Import-Module Az.Sql

Connect-AzAccount
# Get your SubscriptionId from the Get-AzSubscription command
Get-AzSubscription
# Use your SubscriptionId in place of {subscription-id} below
Select-AzSubscription -SubscriptionId {subscription-id}
# Replace {rg-name} with the resource group for your managed instance, and replace {mi-name} with the name of your managed instance
$mi = Get-AzSqlInstance -ResourceGroupName {rg-name} -Name {mi-name}
$mi = $mi | Set-AzSqlInstance -ProxyOverride "Redirect" -force
```

## <a name="next-steps"></a>Pasos siguientes

- [Restauración de una base de datos en una instancia administrada de SQL](restore-sample-database-quickstart.md)
- Aprenda a [configurar un punto de conexión público en Instancia administrada de SQL](public-endpoint-configure.md)
- Información acerca de la [arquitectura de conectividad de Instancia administrada de SQL](connectivity-architecture-overview.md)
