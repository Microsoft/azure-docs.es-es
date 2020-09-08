---
title: 'Inicio rápido: Configuración de registros de flujo de NSG a partir de una plantilla de Azure Resource Manager'
description: Aprenda a habilitar los registros de flujo de NSG mediante programación con una plantilla de Azure Resource Manager y Azure PowerShell.
services: network-watcher
author: damendo
Customer intent: I need to enable the NSG Flow Logs using Azure Resource Manager Template
ms.service: network-watcher
ms.topic: quickstart
ms.date: 07/22/2020
ms.author: damendo
ms.custom: subject-armqs
ms.openlocfilehash: 7d8cb89b1187bb15e7b361e1b6b9505400c612b5
ms.sourcegitcommit: 25bb515efe62bfb8a8377293b56c3163f46122bf
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87986324"
---
# <a name="quickstart-configure-nsg-flow-logs-from-arm-template"></a>Inicio rápido: Configuración de los registros de NSG mediante la plantilla de Azure Resource Manager

En este inicio rápido habilitará los [registros de flujo de NSG](https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-overview) mediante programación con una plantilla de [Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/management/overview) (plantilla ARM) y Azure PowerShell. 

Comenzaremos con información general de las propiedades del objeto del registro de flujo de NSG, seguida de algunas plantillas de ejemplo. A continuación, usaremos una instancia local de PowerShell para implementar la plantilla.

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="nsg-flow-logs-object"></a>Objeto de los registros de flujo de NSG

A continuación se muestra el objeto de los registros de flujo de NSG con todos los parámetros.
Para obtener una introducción completa de las propiedades, lea la [referencia de la plantilla de registros de flujo de NSG](https://docs.microsoft.com/azure/templates/microsoft.network/networkwatchers/flowlogs).

```json
{
  "name": "string",
  "type": "Microsoft.Network/networkWatchers/flowLogs",
  "location": "string",
  "apiVersion": "2019-09-01",
  "properties": {
    "targetResourceId": "string",
    "storageId": "string",
    "enabled": "boolean",
    "flowAnalyticsConfiguration": {
      "networkWatcherFlowAnalyticsConfiguration": {
         "enabled": "boolean",
         "workspaceResourceId": "string",
          "trafficAnalyticsInterval": "integer"
        },
        "retentionPolicy": {
           "days": "integer",
           "enabled": "boolean"
         },
        "format": {
           "type": "string",
           "version": "integer"
         }
      }
    }
  }
```
Para crear un recurso Microsoft.Network/networkWatchers/flowLogs, agregue el JSON anterior a la sección resources de la plantilla.


## <a name="creating-your-template"></a>Creación de la plantilla

Si es la primera vez que usa plantillas de Azure Resource Manager, puede obtener más información sobre ellas con los vínculos siguientes.

* [Implementación de recursos con las plantillas de Resource Manager y Azure PowerShell](https://docs.microsoft.com/azure/azure-resource-manager/templates/deploy-powershell#deploy-local-template)
* [Tutorial: Creación e implementación de la primera plantilla de Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/templates/template-tutorial-create-first-template?tabs=azure-powershell)

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-networkwatcher-flowlogs-create).

El siguiente ejemplo de plantilla completa es la versión más sencilla con los parámetros mínimos pasados para configurar los registros de flujo de NSG. Para más ejemplos, vaya a este [vínculo](https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-azure-resource-manager).

**Ejemplo**: La siguiente plantilla habilita los registros de flujo de NSG en un NSG de destino y los almacena en una cuenta de almacenamiento específica.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "apiProfile": "2019-09-01",
  "resources": [
 {
    "name": "NetworkWatcher_centraluseuap/Microsoft.NetworkDalanDemoPerimeterNSG",
    "type": "Microsoft.Network/networkWatchers/FlowLogs/",
    "location": "centraluseuap",
    "apiVersion": "2019-09-01",
    "properties": {
      "targetResourceId": "/subscriptions/56abfbd6-ec72-4ce9-831f-bc2b6f2c5505/resourceGroups/DalanDemo/providers/Microsoft.Network/networkSecurityGroups/PerimeterNSG",
      "storageId": "/subscriptions/56abfbd6-ec72-4ce9-831f-bc2b6f2c5505/resourceGroups/MyCanaryFlowLog/providers/Microsoft.Storage/storageAccounts/storagev2ira",
      "enabled": true,
      "flowAnalyticsConfiguration": {},
      "retentionPolicy": {},
      "format": {}
    }

  }
  ]
}
```

> [!NOTE]
> * El nombre del recurso tiene el formato "Parent Resource_Child resource". En este caso, el recurso primario es la instancia regional de Network Watcher (formato: NetworkWatcher_NombreDeRegión. Ejemplo: NetworkWatcher_centraluseuap)
> * targetResourceId es el id. de recurso del NSG de destino
> * storageId es el id. de recurso de la cuenta de almacenamiento de destino


## <a name="deploying-your-azure-resource-manager-template"></a>Implementación de la plantilla de Azure Resource Manager

En este tutorial se da por supuesto que tiene un grupo de recursos existente y un NSG en el que puede habilitar el registro de flujo.
Puede guardar cualquiera de las plantillas de ejemplo anteriores de forma local como `azuredeploy.json`. Actualice los valores de las propiedades para que apunten a recursos válidos de su suscripción.

Para implementar la plantilla, ejecute el siguiente comando en PowerShell.
```azurepowershell-interactive
$context = Get-AzSubscription -SubscriptionId 56acfbd6-vc72-43e9-831f-bcdb6f2c5505
Set-AzContext $context
New-AzResourceGroupDeployment -Name EnableFlowLog -ResourceGroupName NetworkWatcherRG `
    -TemplateFile "C:\MyTemplates\azuredeploy.json"
```

> [!NOTE]
> Los comandos anteriores implementan un recurso en el grupo de recursos NetworkWatcherRG y no en el grupo de recursos que contiene NSG.


## <a name="validate-the-deployment"></a>Validación de la implementación

Hay un par de maneras de comprobar si la implementación se ha realizado correctamente. La consola de PowerShell debería mostrar "ProvisioningState" como "Succeeded". Además, puede visitar la [página del portal de registros de flujo de NSG](https://ms.portal.azure.com/#blade/Microsoft_Azure_Network/NetworkWatcherMenuBlade/flowLogs) para confirmar los cambios. Si se produjeron problemas con la implementación, eche un vistazo a [Solución de errores comunes de implementación de Azure con Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/templates/common-deployment-errors).

## <a name="deleting-your-resource"></a>Eliminación del recurso
Azure permite la eliminación de recursos mediante el modo de implementación "Completo". Para eliminar un recurso de registros de flujo, especifique una implementación en modo completo sin incluir el recurso que quiere eliminar. Obtenga más información sobre el [modo de implementación completo](https://docs.microsoft.com/azure/azure-resource-manager/templates/deployment-modes#complete-mode).

Como alternativa, puede deshabilitar un registro de flujo de NSG desde Azure Portal como se indica en los pasos siguientes:
1. Inicie sesión en Azure Portal.
2. En la esquina superior izquierda del portal, seleccione **Todos los servicios**. En el cuadro **Filtrar**, escriba *Network Watcher*. Cuando aparezca la opción **Network Watcher** en los resultados de búsqueda, selecciónela.
3. En **REGISTROS**, seleccione **Registros de flujo de NSG**.
4. En la lista de NSG, seleccione el NSG para el que desea deshabilitar los registros de flujo.
5. En **Configuración de los registros de flujo**, establezca el estado del registro de flujos como **Desactivado**.
6. Desplácese hacia abajo y seleccione **Guardar**.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha habilitado los registros de flujo de NSG. Ahora, tiene que aprender a visualizar los datos de registros de flujo de NSG con: 

* [Microsoft Power BI](network-watcher-visualize-nsg-flow-logs-power-bi.md)
* [Herramientas de código abierto](network-watcher-visualize-nsg-flow-logs-open-source-tools.md)
* [Análisis de tráfico de Azure](https://docs.microsoft.com/azure/network-watcher/traffic-analytics)
