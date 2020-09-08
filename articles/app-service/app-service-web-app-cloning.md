---
title: Clonación de aplicaciones con PowerShell
description: Aprenda a clonar una aplicación de App Service para crear otra nueva con PowerShell. Se trata una variedad de escenarios de clonación, entre los que se incluye la integración de Traffic Manager.
ms.assetid: f9a5cfa1-fbb0-41e6-95d1-75d457347a35
ms.topic: article
ms.date: 01/14/2016
ms.custom: seodec18
ms.openlocfilehash: 92e893fe9f74d51856faf39d3e4800dd5a8155db
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88815392"
---
# <a name="azure-app-service-app-cloning-using-powershell"></a>Clonación de aplicaciones de Azure App Service mediante PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Con el lanzamiento de Microsoft Azure PowerShell versión 1.1.0, se ha agregado una nueva opción a `New-AzWebApp` que permite clonar una aplicación de App Service existente en una aplicación nueva que se crea en una región diferente o en la misma región. Esta opción permitirá que los clientes implementen varias aplicaciones en diferentes regiones de una forma rápida y sencilla.

La clonación de aplicaciones se admite para los planes de App Service Estándar, Premium, Premium V2 y Aislado. La nueva característica cuenta con las mismas limitaciones que la copia de seguridad de App Service; consulte [Realización de una copia de seguridad de una aplicación en Azure App Service](manage-backup.md).

## <a name="cloning-an-existing-app"></a>Clonación de una aplicación existente
Escenario: tiene una aplicación web existente de la región Centro-sur de EE. UU. y desea clonar el contenido en una nueva aplicación web de la región Centro-norte de EE. UU. Para ello, puede utilizar la versión para Azure Resource Manager del cmdlet de PowerShell y crear una nueva aplicación web con la opción `-SourceWebApp`.

Si conoce el nombre del grupo de recursos que contiene la aplicación de origen, puede usar el siguiente comando de PowerShell para obtener la información de la aplicación de origen (en este caso, llamada "`source-webapp`"):

```powershell
$srcapp = Get-AzWebApp -ResourceGroupName SourceAzureResourceGroup -Name source-webapp
```

Para crear un nuevo plan de App Service, puede usar el comando `New-AzAppServicePlan` como en el ejemplo siguiente:

```powershell
New-AzAppServicePlan -Location "North Central US" -ResourceGroupName DestinationAzureResourceGroup -Name DestinationAppServicePlan -Tier Standard
```

Con el comando `New-AzWebApp`, puede crear la nueva aplicación en la región Centro y norte de EE. UU. y asociarla a un plan existente de App Service. Además, puede usar el mismo grupo de recursos que la aplicación de origen, o bien definir un nuevo grupo de recursos, como se muestra en el siguiente comando:

```powershell
$destapp = New-AzWebApp -ResourceGroupName DestinationAzureResourceGroup -Name dest-webapp -Location "North Central US" -AppServicePlan DestinationAppServicePlan -SourceWebApp $srcapp
```

Para clonar una aplicación existente, incluidas todas las ranuras de implementación asociadas, debe usar el parámetro `IncludeSourceWebAppSlots`.  Tenga en cuenta que el parámetro `IncludeSourceWebAppSlots` solo se admite para clonar una aplicación completa, incluidas todas sus ranuras. El siguiente comando de PowerShell muestra el uso de ese parámetro con el comando `New-AzWebApp`:

```powershell
$destapp = New-AzWebApp -ResourceGroupName DestinationAzureResourceGroup -Name dest-webapp -Location "North Central US" -AppServicePlan DestinationAppServicePlan -SourceWebApp $srcapp -IncludeSourceWebAppSlots
```

Para clonar una aplicación existente en la misma región, debe crear un nuevo grupo de recursos y un nuevo plan de App Service en la misma región y después usar el siguiente comando de PowerShell para clonar la aplicación:

```powershell
$destapp = New-AzWebApp -ResourceGroupName NewAzureResourceGroup -Name dest-webapp -Location "South Central US" -AppServicePlan NewAppServicePlan -SourceWebApp $srcapp
```

## <a name="cloning-an-existing-app-to-an-app-service-environment"></a>Clonación de una aplicación existente en un entorno de App Service
Escenario: tiene una aplicación en la región Centro-sur de EE. UU. y desea clonar el contenido en una nueva aplicación utilizando una instancia de App Service Environment (ASE) existente.

Si conoce el nombre del grupo de recursos que contiene la aplicación de origen, puede usar el siguiente comando de PowerShell para obtener la información de la aplicación de origen (en este caso, llamada "`source-webapp`"):

```powershell
$srcapp = Get-AzWebApp -ResourceGroupName SourceAzureResourceGroup -Name source-webapp
```

Si conoce el nombre del ASE y el nombre del grupo de recursos al que este pertenece, puede crear la nueva aplicación en el ASE existente, tal y como se muestra en el siguiente comando:

```powershell
$destapp = New-AzWebApp -ResourceGroupName DestinationAzureResourceGroup -Name dest-webapp -Location "North Central US" -AppServicePlan DestinationAppServicePlan -ASEName DestinationASE -ASEResourceGroupName DestinationASEResourceGroupName -SourceWebApp $srcapp
```

El parámetro `Location` es necesario por el motivo anterior, pero se omite cuando se crea la aplicación en un ASE. 

## <a name="cloning-an-existing-app-slot"></a>Clonación de una ranura de aplicación existente
Escenario: desea clonar la ranura de implementación de una aplicación en una aplicación o una ranura nuevas. La nueva aplicación puede estar en la misma región que la ranura de la aplicación original o en otra distinta.

Si conoce el nombre del grupo de recursos que contiene la aplicación de origen, puede usar el siguiente comando de PowerShell para obtener la información de la ranura de la aplicación de origen (en este caso, llamada "`source-appslot`") vinculada a `source-app`:

```powershell
$srcappslot = Get-AzWebAppSlot -ResourceGroupName SourceAzureResourceGroup -Name source-app -Slot source-appslot
```

En el siguiente comando, se muestra cómo se crea un clon de la aplicación de origen en una nueva aplicación:

```powershell
$destapp = New-AzWebApp -ResourceGroupName DestinationAzureResourceGroup -Name dest-app -Location "North Central US" -AppServicePlan DestinationAppServicePlan -SourceWebApp $srcappslot
```

## <a name="configuring-traffic-manager-while-cloning-an-app"></a>Configuración de Traffic Manager durante la clonación de una aplicación
Para garantizar que las aplicaciones de los clientes tienen una elevada disponibilidad, es importante crear aplicaciones en varias regiones y configurar Azure Traffic Manager para que enrute el tráfico a todas estas aplicaciones. Cuando clone una aplicación existente, tendrá la opción de conectar las dos aplicaciones a un nuevo perfil de Traffic Manager o a uno existente. Solo se admite la versión de Traffic Manager de Azure Resource Manager.

### <a name="creating-a-new-traffic-manager-profile-while-cloning-an-app"></a>Creación de un nuevo perfil de Traffic Manager durante la clonación de una aplicación
Escenario: desea clonar una aplicación en otra región y configurar al mismo tiempo un perfil de Traffic Manager de Azure Resource Manager que incluya las dos aplicaciones. En el siguiente comando, se muestra cómo se crea un clon de la aplicación de origen en una nueva aplicación al tiempo que se configura un nuevo perfil de Traffic Manager:

```powershell
$destapp = New-AzWebApp -ResourceGroupName DestinationAzureResourceGroup -Name dest-webapp -Location "South Central US" -AppServicePlan DestinationAppServicePlan -SourceWebApp $srcapp -TrafficManagerProfileName newTrafficManagerProfile
```

### <a name="adding-new-cloned-app-to-an-existing-traffic-manager-profile"></a>Incorporación de una nueva aplicación clonada a un perfil existente de Traffic Manager
Escenario: ya tiene un perfil de Traffic Manager de Azure Resource Manager al que quiere agregar tanto aplicaciones como puntos de conexión. Para ello, primero debe ensamblar el identificador de perfil existente de Traffic Manager. Necesita el identificador de suscripción, el nombre del grupo de recursos y el nombre del perfil existente de Traffic Manager.

```powershell
$TMProfileID = "/subscriptions/<Your subscription ID goes here>/resourceGroups/<Your resource group name goes here>/providers/Microsoft.TrafficManagerProfiles/ExistingTrafficManagerProfileName"
```

Después de obtener el identificador de Traffic Manager, el siguiente comando muestra cómo crear un clon de la aplicación de origen en una nueva aplicación al mismo tiempo que se agregan ambas aplicaciones a un perfil de Traffic Manager existente:

```powershell
$destapp = New-AzWebApp -ResourceGroupName <Resource group name> -Name dest-webapp -Location "South Central US" -AppServicePlan DestinationAppServicePlan -SourceWebApp $srcapp -TrafficManagerProfileId $TMProfileID
```
> [!NOTE]
> Si recibe un error con el texto "Error de validación de SSL en el nombre de host de Traffic Manager", se recomienda usar el atributo -IgnoreCustomHostNames en el cmdlet de PowerShell al realizar la operación de clonación, o bien usar el portal.

## <a name="current-restrictions"></a>Restricciones actuales
Estas son las restricciones conocidas de la clonación de aplicaciones:

* No se clona la configuración de escalado automático.
* No se clona la configuración de programación de copia de seguridad.
* No se clona la configuración de red virtual.
* App Insights no se instala automáticamente en la aplicación de destino.
* No se clona la configuración de Easy Auth.
* No se clona la extensión Kudu.
* No se clonan las reglas de TiP.
* No se clona el contenido de la base de datos.
* Las direcciones IP de salida cambiarán si la clonación se realiza en una unidad de escalado distinta.
* No está disponible para las aplicaciones de Linux.

### <a name="references"></a>Referencias
* [Clonación de App Service](app-service-web-app-cloning.md)
* [Realización de una copia de seguridad de una aplicación en Azure App Service](manage-backup.md)
* [Compatibilidad del Administrador de recursos de Azure con la vista previa del Administrador de tráfico de Azure](../traffic-manager/traffic-manager-powershell-arm.md)
* [Introducción al entorno de App Service](environment/intro.md)
* [Uso de Azure PowerShell con Azure Resource Manager](../azure-resource-manager/management/manage-resources-powershell.md)

