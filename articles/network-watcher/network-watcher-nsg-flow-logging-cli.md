---
title: 'Administración de registros de flujos de NSG: CLI de Azure'
titleSuffix: Azure Network Watcher
description: Esta página explica cómo administrar registros de flujo de grupos de seguridad de red en Azure Network Watcher con la CLI de Azure
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: damendo
ms.openlocfilehash: c62fbe1621e396ab938327d5159ed8ca8fb84fbf
ms.sourcegitcommit: afa1411c3fb2084cccc4262860aab4f0b5c994ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/23/2020
ms.locfileid: "88756867"
---
# <a name="configuring-network-security-group-flow-logs-with-azure-cli"></a>Configuración de registros de flujo de grupos de seguridad de red con la CLI de Azure

> [!div class="op_single_selector"]
> - [Azure Portal](network-watcher-nsg-flow-logging-portal.md)
> - [PowerShell](network-watcher-nsg-flow-logging-powershell.md)
> - [CLI de Azure](network-watcher-nsg-flow-logging-cli.md)
> - [REST API](network-watcher-nsg-flow-logging-rest.md)

Los registros de flujo de grupos de seguridad de red son una característica de Network Watcher que permite ver información acerca del tráfico IP de entrada y de salida en un grupo de seguridad de red. Estos registros de flujo se escriben en formato JSON y muestran los flujos de entrada y salida en función de cada regla, la NIC a la que se aplica el flujo, información de 5-tupla sobre el flujo (IP de origen/destino, puerto de origen/destino, protocolo), y si se permitió o denegó el tráfico.

Para seguir los pasos de este artículo, es preciso [instalar la interfaz de la línea de comandos de Azure para Mac, Linux y Windows (CLI)](/cli/azure/install-azure-cli).

## <a name="register-insights-provider"></a>Registro del proveedor de Insights

Para que el registro del flujo de trabajo funcione correctamente, es necesario registrar el proveedor **Microsoft.Insights**. Si no está seguro de si el proveedor **Microsoft.Insights**está registrado, ejecute el siguiente script.

```azurecli
az provider register --namespace Microsoft.Insights
```

## <a name="enable-network-security-group-flow-logs"></a>Habilitación de los registros de flujo de grupos de seguridad de red

El ejemplo siguiente muestra el comando para habilitar los registros de flujo:

```azurecli
az network watcher flow-log create --resource-group resourceGroupName --enabled true --nsg nsgName --storage-account storageAccountName --location location
# Configure 
az network watcher flow-log create --resource-group resourceGroupName --enabled true --nsg nsgName --storage-account storageAccountName --location location --format JSON --log-version 2
```

La cuenta de almacenamiento que especifique no puede tener configuradas reglas de red que restrinjan el acceso a la red solo a servicios de Microsoft o a redes virtuales específicas. La cuenta de almacenamiento puede estar en la misma suscripción de Azure, o en una diferente, que el NSG para el que habilite el registro de flujo. Si utiliza distintas suscripciones, ambas deben estar asociadas al mismo inquilino de Azure Active Directory. La cuenta que utilice para cada suscripción debe tener los [permisos necesarios](required-rbac-permissions.md). 

Si la cuenta de almacenamiento está en un grupo de recursos o suscripción distintos al grupo de seguridad de la red, especifique el identificador completo de la cuenta de almacenamiento en lugar de su nombre. Por ejemplo, si la cuenta de almacenamiento está en un grupo de recursos denominado *RG-Storage*, en vez de especificar *storageAccountName* en el comando anterior, debería especificar */subscriptions/{SubscriptionID}/resourceGroups/RG-Storage/providers/Microsoft.Storage/storageAccounts/storageAccountName*.

## <a name="disable-network-security-group-flow-logs"></a>Deshabilitación de los registros de flujo de grupos de seguridad de red

Use el ejemplo siguiente para deshabilitar los registros de flujo:

```azurecli
az network watcher flow-log configure --resource-group resourceGroupName --enabled false --nsg nsgName
```

## <a name="download-a-flow-log"></a>Descarga de un registro de flujo

La ubicación de almacenamiento de un registro de flujo se define en el momento de la creación. Una herramienta práctica para acceder a estos registros de flujo guardados en una cuenta de almacenamiento es el Explorador de Microsoft Azure Storage, que puede descargarse aquí: https://storageexplorer.com/

Si se especifica una cuenta de almacenamiento, los archivos de registro de flujo se guardan en una cuenta de almacenamiento en la siguiente ubicación:

```
https://{storageAccountName}.blob.core.windows.net/insights-logs-networksecuritygroupflowevent/resourceId=/SUBSCRIPTIONS/{subscriptionID}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/{nsgName}/y={year}/m={month}/d={day}/h={hour}/m=00/macAddress={macAddress}/PT1H.json
```


## <a name="next-steps"></a>Pasos siguientes

Aprenda a [visualizar los registros de flujo de NSG con Power BI](network-watcher-visualize-nsg-flow-logs-power-bi.md)

Aprenda a [visualizar los registros de flujo de NSG con herramientas de código abierto](network-watcher-visualize-nsg-flow-logs-open-source-tools.md)
