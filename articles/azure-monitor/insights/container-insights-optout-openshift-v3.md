---
title: Cómo detener la supervisión del clúster de Red Hat OpenShift en Azure v3 | Microsoft Docs
description: En este artículo se describe cómo puede detener la supervisión del clúster de Red Hat OpenShift en Azure con Azure Monitor para contenedores.
ms.topic: conceptual
ms.date: 04/24/2020
ms.openlocfilehash: 6ed92cd4cda1f3b5d43cc605d7224236528b94bf
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88815443"
---
# <a name="how-to-stop-monitoring-your-azure-red-hat-openshift-v3-cluster"></a>Cómo detener la supervisión del clúster de Red Hat OpenShift en Azure v3 

Después de habilitar la supervisión del clúster de Red Hat OpenShift en Azure versión 3.x, puede detener la supervisión del clúster con Azure Monitor para contenedores si decide que ya no desea continuar haciéndolo. En este artículo se muestra cómo realizar esta tarea mediante la plantilla de Azure Resource Manager proporcionada. 

## <a name="azure-resource-manager-template"></a>Plantilla del Administrador de recursos de Azure

Se proporcionan dos plantillas de Azure Resource Manager para permitir la eliminación de los recursos de la solución de manera coherente y repetida en el grupo de recursos. Una es una plantilla JSON que especifica la configuración para dejar de supervisar y la otra contiene valores de parámetros que debe configurar para especificar el id. de recurso del clúster de AKS y la región de Azure en la que se implementa el clúster.

Si no conoce el concepto de implementación de recursos mediante una plantilla, consulte:
* [Implementación de recursos con las plantillas de Resource Manager y Azure PowerShell](../../azure-resource-manager/templates/deploy-powershell.md)
* [Implementación de recursos con plantillas de Resource Manager y la CLI de Azure](../../azure-resource-manager/templates/deploy-cli.md)

Si decide usar la CLI de Azure, primero debe instalar y usar la CLI localmente. Debe ejecutar la versión 2.0.65 de la CLI de Azure o cualquier versión posterior. Para identificar la versión, ejecute `az --version`. Si necesita instalar o actualizar la CLI de Azure, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

### <a name="create-template"></a>Creación de una plantilla

1. Copie y pegue la siguiente sintaxis JSON en el archivo:

    ```json
    {
       "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
       "contentVersion": "1.0.0.0",
       "parameters": {
         "aroResourceId": {
           "type": "string",
           "metadata": {
             "description": "ARO Cluster Resource ID"
          }
        },
        "aroResourceLocation": {
          "type": "string",
          "metadata": {
            "description": "Location of the aro cluster resource e.g. westcentralus"
          }
        }
      },
      "resources": [
        {
           "name": "[split(parameters('aroResourceId'),'/')[8]]",
           "type": "Microsoft.ContainerService/openShiftManagedClusters",
           "location": "[parameters('aroResourceLocation')]",
           "apiVersion": "2019-09-30-preview",
           "properties": {
             "mode": "Incremental",
             "id": "[parameters('aroResourceId')]",
             "monitorProfile": {
               "workspaceResourceID": null,
               "enabled": false
            }
          }
        }
      ]
    }
    ```

2. Guarde este archivo como **OptOutTemplate.json** en una carpeta local.

3. Pegue la siguiente sintaxis de JSON en el archivo:

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "aroResourceId": {
          "value": "/subscriptions/<subscriptionId>/resourceGroups/<ResourceGroupName>/providers/Microsoft.ContainerService/openShiftManagedClusters/<clusterName>"
        },
        "aroResourceLocation": {
          "value": "<azure region of the cluster e.g. westcentralus>"
        }
      }
    }
    ```

4. Edite los valores de **aroResourceId** y **aroResourceLocation** usando los valores del clúster de OpenShift, que puede encontrar en la página **Propiedades** del clúster seleccionado.

    ![Página de propiedades del contenedor](media/container-insights-optout-openshift/cluster-properties-page.png)

5. Guarde este archivo como **OptOutParam.json** en una carpeta local.

6. Está listo para implementar esta plantilla.

### <a name="remove-the-solution-using-azure-cli"></a>Eliminación de la solución mediante la CLI de Azure

Ejecute el siguiente comando con la CLI de Azure en Linux para eliminar la solución y borrar la configuración en el clúster.

```azurecli
az login   
az account set --subscription "Subscription Name"
az group deployment create --resource-group <ResourceGroupName> --template-file ./OptOutTemplate.json --parameters @./OptOutParam.json  
```

El cambio de configuración puede tardar unos minutos en completarse. Cuando termine, verá un mensaje similar al siguiente que incluye el resultado:

```output
ProvisioningState       : Succeeded
```

### <a name="remove-the-solution-using-powershell"></a>Eliminación de la solución mediante PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Ejecute los siguientes comandos de PowerShell en la carpeta que contiene la plantilla para eliminar la solución y borrar la configuración del clúster.    

```powershell
Connect-AzAccount
Select-AzSubscription -SubscriptionName <yourSubscriptionName>
New-AzResourceGroupDeployment -Name opt-out -ResourceGroupName <ResourceGroupName> -TemplateFile .\OptOutTemplate.json -TemplateParameterFile .\OptOutParam.json
```

El cambio de configuración puede tardar unos minutos en completarse. Cuando termine, verá un mensaje similar al siguiente que incluye el resultado:

```output
ProvisioningState       : Succeeded
```

## <a name="next-steps"></a>Pasos siguientes

Si el área de trabajo se creó solamente para admitir la supervisión del clúster y ya no es necesario, tiene que eliminarlo manualmente. Si no está familiarizado con la eliminación de un área de trabajo, consulte [Eliminación de un área de trabajo de Azure Log Analytics](../platform/delete-workspace.md).
