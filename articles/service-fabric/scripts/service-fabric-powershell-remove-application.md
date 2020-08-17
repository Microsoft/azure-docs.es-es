---
title: Eliminación de una aplicación de un clúster de PowerShell
description: 'Ejemplo de script de Azure PowerShell: quitar una aplicación de un clúster de Service Fabric.'
services: service-fabric
documentationcenter: ''
author: athinanthny
manager: chackdan
editor: ''
tags: azure-service-management
ms.assetid: ''
ms.service: service-fabric
ms.workload: multiple
ms.topic: sample
ms.date: 01/18/2018
ms.author: atsenthi
ms.custom: mvc
ms.openlocfilehash: 8b4b07288ce2c3570da5482a446b9418c7319011
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87086207"
---
# <a name="remove-an-application-from-a-service-fabric-cluster-using-powershell"></a>Eliminación de una aplicación de un clúster de Service Fabric mediante PowerShell

Este script de ejemplo elimina una instancia de aplicación de Service Fabric en ejecución y anula el registro del tipo y la versión de una aplicación del clúster.  La eliminación de la instancia de la aplicación también elimina todas las instancias de servicio en ejecución asociadas a esa aplicación. Personalice los parámetros según sea necesario. 

Si es necesario, instale el módulo Service Fabric PowerShell con el [SDK de Service Fabric](../service-fabric-get-started.md). 

## <a name="sample-script"></a>Script de ejemplo

[!code-powershell[main](../../../powershell_scripts/service-fabric/remove-application/remove-application.ps1 "Remove an application from a cluster")]

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [Remove-ServiceFabricApplication](/powershell/module/servicefabric/remove-servicefabricapplication?view=azureservicefabricps) | Quita una instancia de aplicación de Service Fabric en ejecución del clúster.  |
| [Unregister-ServiceFabricApplicationType](/powershell/module/servicefabric/unregister-servicefabricapplicationtype?view=azureservicefabricps) | Anula el registro del tipo y la versión de una aplicación de Service Fabric del clúster. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre el módulo Service Fabric PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/service-fabric/overview?view=azureservicefabricps).

Puede encontrar ejemplos de PowerShell para Azure Service Fabric en los [ejemplos de Azure PowerShell](../service-fabric-powershell-samples.md).
