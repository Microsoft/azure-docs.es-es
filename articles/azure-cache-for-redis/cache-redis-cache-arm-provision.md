---
title: Implementación de Azure Cache for Redis con Azure Resource Manager
description: Use una plantilla de Azure Resource Manager para implementar Azure Cache for Redis. Se suministran plantillas para escenarios frecuentes.
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: conceptual
ms.date: 01/23/2017
ms.openlocfilehash: 2d00a6b7753a61bb2527a56231b2fe054736f1b0
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88008583"
---
# <a name="create-an-azure-cache-for-redis-using-a-template"></a>Creación de una instancia de Azure Cache for Redis mediante una plantilla

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

En este tema, aprenderá a crear una plantilla de Azure Resource Manager que implementa Azure Cache for Redis. La memoria caché se puede usar con una cuenta de almacenamiento existente para mantener los datos de diagnóstico. Aprenderá a definir los recursos que se implementan y los parámetros que se especifican cuando se ejecuta la implementación. Puede usar esta plantilla para sus propias implementaciones o personalizarla para satisfacer sus necesidades.

Actualmente, se comparten los ajustes de configuración de diagnóstico para todas las cachés de la misma región para una suscripción. Actualizar una caché en la región afectará a todas las demás cachés de la región.

Para obtener más información sobre la creación de plantillas, consulte [Creación de plantillas de Administrador de recursos de Azure](../azure-resource-manager/templates/template-syntax.md). Para información sobre la sintaxis y las propiedades de JSON para los tipos de recursos de caché, consulte [Tipos de recursos Microsoft.Cache](/azure/templates/microsoft.cache/allversions).

Para ver la plantilla completa, consulte [Azure Cache for Redis template](https://github.com/Azure/azure-quickstart-templates/blob/master/101-redis-cache/azuredeploy.json) (Plantilla de Azure Cache for Redis).

> [!NOTE]
> Las plantillas de Resource Manager para el nuevo [nivel Premium](cache-overview.md#service-tiers) están disponibles. 
> 
> * [Creación de una instancia de Azure Cache for Redis de nivel Prémium con agrupación en clústeres](https://azure.microsoft.com/resources/templates/201-redis-premium-cluster-diagnostics/)
> * [Creación de una instancia de Azure Cache for Redis de nivel Prémium con persistencia de datos](https://azure.microsoft.com/resources/templates/201-redis-premium-persistence/)
> * [Creación de una instancia de Redis Cache premium implementada en una red virtual](https://azure.microsoft.com/resources/templates/201-redis-premium-vnet/)
> 
> Para buscar las últimas plantillas, diríjase a [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/documentation/templates/) y busque `Azure Cache for Redis`.
> 
> 

## <a name="what-you-will-deploy"></a>Lo que implementará
En esta plantilla, implementará una instancia de Azure Cache for Redis que utiliza una cuenta de almacenamiento de datos de diagnóstico.

Para ejecutar automáticamente la implementación, haga clic en el botón siguiente:

[![Implementación en Azure](./media/cache-redis-cache-arm-provision/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-redis-cache%2Fazuredeploy.json)

## <a name="parameters"></a>Parámetros
Con el Administrador de recursos de Azure, se definen los parámetros de los valores que desea especificar al implementar la plantilla. La plantilla incluye una sección denominada Parámetros que contiene todos los valores de los parámetros.
Debe definir un parámetro para esos valores que variarán según el proyecto que vaya a implementar o según el entorno en el que vaya a realizar la implementación. No defina parámetros para valores que siempre permanezcan igual. Cada valor de parámetro se usa en la plantilla para definir los recursos que se implementan. 

[!INCLUDE [app-service-web-deploy-redis-parameters](../../includes/cache-deploy-parameters.md)]

### <a name="rediscachelocation"></a>redisCacheLocation
La ubicación de la instancia de Azure Cache for Redis. Para un mejor rendimiento, utilice la misma ubicación que la aplicación que se usará con la memoria caché.

```json
  "redisCacheLocation": {
    "type": "string"
  }
```

### <a name="existingdiagnosticsstorageaccountname"></a>existingDiagnosticsStorageAccountName
Nombre de la cuenta de almacenamiento existente que desea usar para el diagnóstico. 

```json
  "existingDiagnosticsStorageAccountName": {
    "type": "string"
  }
```

### <a name="enablenonsslport"></a>enableNonSslPort
Valor booleano que indica si se debe permitir el acceso a través de puertos no SSL.

```json
  "enableNonSslPort": {
    "type": "bool"
  }
```

### <a name="diagnosticsstatus"></a>diagnosticsStatus
Un valor que indica si están activados los diagnósticos. Utilice ON (activados) u OFF (desactivados).

```json
  "diagnosticsStatus": {
    "type": "string",
    "defaultValue": "ON",
    "allowedValues": [
          "ON",
          "OFF"
      ]
  }
```

## <a name="resources-to-deploy"></a>Recursos para implementar
### <a name="azure-cache-for-redis"></a>Azure Cache for Redis
Crea la instancia de Azure Cache for Redis.

```json
  {
    "apiVersion": "2015-08-01",
    "name": "[parameters('redisCacheName')]",
    "type": "Microsoft.Cache/Redis",
    "location": "[parameters('redisCacheLocation')]",
    "properties": {
      "enableNonSslPort": "[parameters('enableNonSslPort')]",
      "sku": {
        "capacity": "[parameters('redisCacheCapacity')]",
        "family": "[parameters('redisCacheFamily')]",
        "name": "[parameters('redisCacheSKU')]"
      }
    },
    "resources": [
      {
        "apiVersion": "2017-05-01-preview",
        "type": "Microsoft.Cache/redis/providers/diagnosticsettings",
        "name": "[concat(parameters('redisCacheName'), '/Microsoft.Insights/service')]",
        "location": "[parameters('redisCacheLocation')]",
        "dependsOn": [
          "[concat('Microsoft.Cache/Redis/', parameters('redisCacheName'))]"
        ],
        "properties": {
          "status": "[parameters('diagnosticsStatus')]",
          "storageAccountName": "[parameters('existingDiagnosticsStorageAccountName')]"
        }
      }
    ]
  }
```

## <a name="commands-to-run-deployment"></a>Comandos para ejecutar la implementación
[!INCLUDE [app-service-deploy-commands](../../includes/app-service-deploy-commands.md)]

### <a name="powershell"></a>PowerShell

```azurepowershell
    New-AzResourceGroupDeployment -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-redis-cache/azuredeploy.json -ResourceGroupName ExampleDeployGroup -redisCacheName ExampleCache
```

### <a name="azure-cli"></a>Azure CLI

```azurecli
    azure group deployment create --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-redis-cache/azuredeploy.json -g ExampleDeployGroup
```
