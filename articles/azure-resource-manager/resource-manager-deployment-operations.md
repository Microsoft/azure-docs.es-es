---
title: Historial de implementación con Azure Resource Manager | Microsoft Docs
description: Se describe cómo ver las operaciones de implementación de Azure Resource Manager con el portal, PowerShell, CLI de Azure y API de REST.
tags: top-support-issue
author: tfitzmac
ms.service: azure-resource-manager
ms.topic: conceptual
ms.date: 05/13/2019
ms.author: tomfitz
ms.openlocfilehash: 58d22e3fcae5c30e5d7dcc39b317afeef4a693ee
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65606037"
---
# <a name="view-deployment-history-with-azure-resource-manager"></a>Ver historial de implementación con Azure Resource Manager

Azure Resource Manager permite ver el historial de implementación y examinar las operaciones específicas en las implementaciones anteriores. Puede ver los recursos que se han implementado y obtener información sobre los errores.

Para obtener ayuda con la resolución de errores de implementación concretos, consulte [Solución de problemas comunes al implementar recursos en Azure con Azure Resource Manager](resource-manager-common-deployment-errors.md).

## <a name="portal"></a>Portal

Para obtener detalles acerca de una implementación del historial de implementación.

1. Seleccione el grupo de recursos que desea examinar.

1. Seleccione el vínculo situado bajo **implementaciones**.

   ![Seleccione el historial de implementación](./media/resource-manager-deployment-operations/select-deployment-history.png)

1. Seleccione una de las implementaciones del historial de implementación.

   ![Seleccione la implementación](./media/resource-manager-deployment-operations/select-details.png)

1. Se muestra un resumen de la implementación, incluida una lista de los recursos que se implementaron.

    ![Resumen de implementación](./media/resource-manager-deployment-operations/view-deployment-summary.png)

1. Para ver la plantilla usada para la implementación, seleccione **plantilla**. Puede descargar la plantilla para volver a usarla.

    ![Mostrar plantilla](./media/resource-manager-deployment-operations/show-template-from-history.png)

1. Si se produjo un error en la implementación, verá un mensaje de error. Seleccione el mensaje de error para obtener más detalles.

    ![ver la implementación con errores](./media/resource-manager-deployment-operations/show-error.png)

1. Se muestra el mensaje de error detallado.

    ![Ver los detalles del error](./media/resource-manager-deployment-operations/show-details.png)

1. El identificador de correlación se utiliza para realizar un seguimiento de eventos relacionados y puede ser útil al trabajar con el soporte técnico para solucionar problemas de una implementación.

    ![Obtener Id. de correlación](./media/resource-manager-deployment-operations/get-correlation-id.png)

1. Para obtener más información sobre el paso con error, seleccione **detalles de la operación**.

    ![Seleccione las operaciones de implementación](./media/resource-manager-deployment-operations/select-deployment-operations.png)

1. Consulte los detalles de ese paso de la implementación.

    ![Mostrar detalles de la operación](./media/resource-manager-deployment-operations/show-operation-details.png)

## <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Para obtener el estado general de una implementación, use el comando **Get-AzResourceGroupDeployment**.

```azurepowershell-interactive
Get-AzResourceGroupDeployment -ResourceGroupName ExampleGroup
```

O bien, puede filtrar los resultados para obtener únicamente las implementaciones en las que se han producido errores.

```azurepowershell-interactive
Get-AzResourceGroupDeployment -ResourceGroupName ExampleGroup | Where-Object ProvisioningState -eq Failed
```

El identificador de correlación se utiliza para realizar un seguimiento de eventos relacionados y puede ser útil al trabajar con el soporte técnico para solucionar problemas de una implementación. Para obtener el Id. de correlación, use:

```azurepowershell-interactive
(Get-AzResourceGroupDeployment -ResourceGroupName ExampleGroup -DeploymentName azuredeploy).CorrelationId
```

Cada implementación incluye varias operaciones. Cada operación representa un paso del proceso de implementación. Para detectar qué salió mal con una implementación, normalmente es necesario ver los detalles de las operaciones de implementación. Puede ver el estado de las operaciones con **Get-AzResourceGroupDeploymentOperation**.

```azurepowershell-interactive
Get-AzResourceGroupDeploymentOperation -ResourceGroupName ExampleGroup -DeploymentName azuredeploy
```

Que devuelve varias operaciones con cada una de ellas en el formato siguiente:

```powershell
Id             : /subscriptions/{guid}/resourceGroups/ExampleGroup/providers/Microsoft.Resources/deployments/Microsoft.Template/operations/A3EB2DA598E0A780
OperationId    : A3EB2DA598E0A780
Properties     : @{provisioningOperation=Create; provisioningState=Succeeded; timestamp=2019-05-13T21:42:40.7151512Z;
                duration=PT23.0227078S; trackingId=11d376e8-5d6d-4da8-847e-6f23c6443fbf;
                serviceRequestId=0196828d-8559-4bf6-b6b8-8b9057cb0e23; statusCode=OK; targetResource=}
PropertiesText : {duration:PT23.0227078S, provisioningOperation:Create, provisioningState:Succeeded,
                serviceRequestId:0196828d-8559-4bf6-b6b8-8b9057cb0e23...}
```

Para más información acerca de las operaciones con errores, recupere las propiedades de aquellas operaciones con el estado **Error** .

```azurepowershell-interactive
(Get-AzResourceGroupDeploymentOperation -DeploymentName azuredeploy -ResourceGroupName ExampleGroup).Properties | Where-Object ProvisioningState -eq Failed
```

Que devuelve todas las operaciones con error con cada una de ellas en el formato siguiente:

```powershell
provisioningOperation : Create
provisioningState     : Failed
timestamp             : 2019-05-13T21:42:40.7151512Z
duration              : PT3.1449887S
trackingId            : f4ed72f8-4203-43dc-958a-15d041e8c233
serviceRequestId      : a426f689-5d5a-448d-a2f0-9784d14c900a
statusCode            : BadRequest
statusMessage         : @{error=}
targetResource        : @{id=/subscriptions/{guid}/resourceGroups/ExampleGroup/providers/
                       Microsoft.Network/publicIPAddresses/myPublicIP;
                       resourceType=Microsoft.Network/publicIPAddresses; resourceName=myPublicIP}
```

Tenga en cuenta los valores serviceRequestId y trackingId de la operación. serviceRequestId puede resultar útil cuando se trabaja con el soporte técnico para solucionar un problema de implementación. Utilizará trackingId en el paso siguiente para centrarse en una operación determinada.

Para obtener el mensaje de estado de una operación con error determinada, use el siguiente comando:

```azurepowershell-interactive
((Get-AzResourceGroupDeploymentOperation -DeploymentName azuredeploy -ResourceGroupName ExampleGroup).Properties | Where-Object trackingId -eq f4ed72f8-4203-43dc-958a-15d041e8c233).StatusMessage.error
```

Que devuelve:

```powershell
code           message                                                                        details
----           -------                                                                        -------
DnsRecordInUse DNS record dns.westus.cloudapp.azure.com is already used by another public IP. {}
```

Cada operación de implementación en Azure incluye el contenido de la solicitud y de la respuesta. Durante la implementación, puede usar **DeploymentDebugLogLevel** parámetro para especificar que la solicitud o respuesta se registra.

Esa información se obtiene del registro y se guarda localmente mediante los siguientes comandos de PowerShell:

```powershell
(Get-AzResourceGroupDeploymentOperation -DeploymentName "TestDeployment" -ResourceGroupName "Test-RG").Properties.request | ConvertTo-Json |  Out-File -FilePath <PathToFile>

(Get-AzResourceGroupDeploymentOperation -DeploymentName "TestDeployment" -ResourceGroupName "Test-RG").Properties.response | ConvertTo-Json |  Out-File -FilePath <PathToFile>
```

## <a name="azure-cli"></a>Azure CLI

Para obtener el estado general de una implementación, use el **show de implementación de grupo de azure** comando.

```azurecli-interactive
az group deployment show -g ExampleGroup -n ExampleDeployment
```
  
El identificador de correlación se utiliza para realizar un seguimiento de eventos relacionados y puede ser útil al trabajar con el soporte técnico para solucionar problemas de una implementación.

```azurecli-interactive
az group deployment show -g ExampleGroup -n ExampleDeployment --query properties.correlationId
```

Para ver las operaciones para una implementación, use:

```azurecli-interactive
az group deployment operation list -g ExampleGroup -n ExampleDeployment
```

## <a name="rest"></a>REST

Para obtener información acerca de una implementación, use el [obtener información acerca de una implementación de plantilla](https://docs.microsoft.com/rest/api/resources/deployments) operación.

```
GET https://management.azure.com/subscriptions/{subscription-id}/resourcegroups/{resource-group-name}/providers/microsoft.resources/deployments/{deployment-name}?api-version={api-version}
```

En la respuesta, observe en particular los elementos **provisioningState**, **correlationId** y **error**. **correlationId** se usa para realizar un seguimiento de eventos relacionados y puede ser útil al colaborar con el soporte técnico para solucionar un problema de implementación.

```json
{ 
 ...
 "properties": {
   "provisioningState":"Failed",
   "correlationId":"d5062e45-6e9f-4fd3-a0a0-6b2c56b15757",
   ...
   "error":{
     "code":"DeploymentFailed","message":"At least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/arm-debug for usage details.",
     "details":[{"code":"Conflict","message":"{\r\n  \"error\": {\r\n    \"message\": \"Conflict\",\r\n    \"code\": \"Conflict\"\r\n  }\r\n}"}]
   }  
 }
}
```

Para obtener información acerca de las implementaciones, use [enumera todas las operaciones de implementación de plantilla](https://docs.microsoft.com/rest/api/resources/deployments). 

```
GET https://management.azure.com/subscriptions/{subscription-id}/resourcegroups/{resource-group-name}/providers/microsoft.resources/deployments/{deployment-name}/operations?$skiptoken={skiptoken}&api-version={api-version}
```
   
La respuesta incluye información de la solicitud o la respuesta según lo especificado en la propiedad **debugSetting** durante la implementación.

```json
{
 ...
 "properties": 
 {
   ...
   "request":{
     "content":{
       "location":"West US",
       "properties":{
         "accountType": "Standard_LRS"
       }
     }
   },
   "response":{
     "content":{
       "error":{
         "message":"Conflict","code":"Conflict"
       }
     }
   }
 }
}
```

## <a name="next-steps"></a>Pasos siguientes
* Para obtener ayuda con la resolución de errores de implementación concretos, consulte [Solución de problemas comunes al implementar recursos en Azure con Azure Resource Manager](resource-manager-common-deployment-errors.md).
* Para obtener información sobre cómo usar los registros de actividad para supervisar otros tipos de acciones, vea [View activity logs to manage Azure resources](resource-group-audit.md) (Ver registros de actividad para administrar recursos de Azure).
* Para validar la implementación antes de ejecutarla, consulte [Implementación de recursos con las plantillas de Resource Manager y Azure PowerShell](resource-group-template-deploy.md).

