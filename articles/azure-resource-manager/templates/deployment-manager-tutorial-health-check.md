---
title: Uso de la comprobación del estado de Azure Deployment Manager
description: Use la comprobación de estado para implementar de forma segura los recursos de Azure con Azure Deployment Manager.
author: mumian
ms.date: 10/09/2019
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: 3c7b74d31bc3c4e2276cd52c8e6450630dc99bcd
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "86058034"
---
# <a name="tutorial-use-health-check-in-azure-deployment-manager-public-preview"></a>Tutorial: Uso de la comprobación de estado en Azure Deployment Manager (versión preliminar pública)

Aprenda a integrar la comprobación de estado en [Azure Deployment Manager](./deployment-manager-overview.md). Este tutorial se basa en el tutorial [Uso de Azure Deployment Manager con plantillas de Resource Manager](./deployment-manager-tutorial.md), por lo que debe completar este último antes de continuar con el primero.

La plantilla de lanzamiento que utilizó en [Uso de Azure Deployment Manager con plantillas de Resource Manager](./deployment-manager-tutorial.md), contaba con un paso de espera. En este tutorial, el paso de espera se reemplaza por un paso de comprobación de estado.

> [!IMPORTANT]
> Si la suscripción está marcada para Canary para probar las nuevas características de Azure, Azure Deployment Manager solo se puede usar para realizar implementaciones en las regiones de Canary. 

En este tutorial se describen las tareas siguientes:

> [!div class="checklist"]
> * Creación de un simulador del servicio de comprobación de estado
> * Revisión de la plantilla de lanzamiento
> * Implementación de la topología
> * Implementar el lanzamiento con estado incorrecto
> * Comprobar la implementación del lanzamiento
> * Implementar el lanzamiento con estado de correcto
> * Comprobar la implementación del lanzamiento
> * Limpieza de recursos

Recursos adicionales:

* La [referencia de API REST de Azure Deployment Manager](/rest/api/deploymentmanager/).
* [Un ejemplo de Azure Deployment Manager](https://github.com/Azure-Samples/adm-quickstart).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para completar este artículo, necesitará lo siguiente:

* Completar [Uso de Azure Deployment Manager con plantillas de Resource Manager](./deployment-manager-tutorial.md).

## <a name="install-the-artifacts"></a>Instalación de los artefactos

Descargue [las plantillas y los artefactos](https://github.com/Azure/azure-docs-json-samples/raw/master/tutorial-adm/ADMTutorial.zip), y descomprímalos en el entorno local si no lo ha hecho aún. A continuación, ejecute el script de PowerShell que se encuentra en [Preparación de los artefactos](./deployment-manager-tutorial.md#prepare-the-artifacts). El script crea un grupo de recursos, un contenedor de almacenamiento y un contenedor de blobs, carga los archivos descargados y crea un token de SAS.

Copie la dirección URL con el token de SAS. Esta dirección URL es necesaria para rellenar un campo en los dos archivos de parámetros, el archivo de parámetros de topología y el archivo de parámetros de lanzamiento.

Abra CreateADMServiceTopology.Parameters.json y actualice los valores de **projectName** y **artifactSourceSASLocation**.

Abra CreateADMRollout.Parameters.json y actualice los valores de **projectName** y **artifactSourceSASLocation**.

## <a name="create-a-health-check-service-simulator"></a>Creación de un simulador del servicio de comprobación de estado

En producción, se suele utilizar uno o varios proveedores de supervisión. Para que la integración del mantenimiento sea lo más sencilla posible, Microsoft ha trabajado con varias de las compañías de seguimiento del estado de los servicios más punteras para ofrecer una solución sencilla de copiar y pegar para integrar las comprobaciones de estado en las implementaciones. Para obtener una lista de estas empresas, consulte [Proveedores de seguimiento de estado](./deployment-manager-health-check.md#health-monitoring-providers). Para este tutorial, cree una [función de Azure](../../azure-functions/index.yml) que simule un servicio de supervisión de estado. Dicha función toma un código de estado y devuelve el mismo código. La plantilla de Azure Deployment Manager usa el código de estado para determinar cómo realizar la implementación.

Los dos archivos siguientes se usan para implementar la función de Azure. Para completar el tutorial no es preciso descargarlos.

* Una plantilla de Resource Manager se encuentra en [https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/tutorial-adm/deploy_hc_azure_function.json](https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/tutorial-adm/deploy_hc_azure_function.json). Implemente esta plantilla para crear una función de Azure.
* Un archivo zip del código fuente de la función de Azure, [https://github.com/Azure/azure-docs-json-samples/raw/master/tutorial-adm/ADMHCFunction0417.zip](https://github.com/Azure/azure-docs-json-samples/raw/master/tutorial-adm/ADMHCFunction0417.zip). La plantilla de Resource Manager llama a este archivo zip.

Para implementar la función de Azure, seleccione **Pruébelo** para abrir Azure Cloud Shell y pegue el siguiente script en la ventana del shell.  Para pegar el código, haga clic con el botón derecho en la ventana del shell y seleccione **Pegar**.

```azurepowershell
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/tutorial-adm/deploy_hc_azure_function.json" -projectName $projectName
```

Para comprobar y probar la función de Azure:

1. Abra [Azure Portal](https://portal.azure.com).
1. Abra el grupo de recursos.  El nombre predeterminado es el nombre del proyecto con **rg** anexado.
1. Seleccione el servicio de aplicaciones en el grupo de recursos.  El nombre predeterminado del servicio de aplicaciones es el nombre del proyecto con **webapp** anexado.
1. Expanda **Functions** y seleccione **HttpTrigger1**.

    ![Función de Azure de la comprobación de estado de Azure Deployment Manager](./media/deployment-manager-tutorial-health-check/azure-deployment-manager-hc-function.png)

1. Seleccione **&lt;/> Obtener la dirección URL de la función**.
1. Seleccione **Copiar** para copiar la dirección URL en el Portapapeles.  La dirección URL es similar a:

    ```url
    https://myhc0417webapp.azurewebsites.net/api/healthStatus/{healthStatus}?code=hc4Y1wY4AqsskAkVw6WLAN1A4E6aB0h3MbQ3YJRF3XtXgHvooaG0aw==
    ```

    Reemplace `{healthStatus}` en la dirección URL por un código de estado. En este tutorial, use **unhealthy** para probar el escenario incorrecto y use **healthy** o **warning** para probar el escenario correcto. Cree dos direcciones URL, una con el estado incorrecto y la otra con el estado correcto. Por ejemplo:

    ```url
    https://myhc0417webapp.azurewebsites.net/api/healthStatus/unhealthy?code=hc4Y1wY4AqsskAkVw6WLAN1A4E6aB0h3MbQ3YJRF3XtXgHvooaG0aw==
    https://myhc0417webapp.azurewebsites.net/api/healthStatus/healthy?code=hc4Y1wY4AqsskAkVw6WLAN1A4E6aB0h3MbQ3YJRF3XtXgHvooaG0aw==
    ```

    Para completar este tutorial, necesita ambas direcciones URL.

1. Para probar el simulador de comprobación de estado, abra las direcciones URL que creó en el último paso.  Los resultados del estado incorrecto deberán ser similares a:

    ```
    Status: unhealthy
    ```

## <a name="revise-the-rollout-template"></a>Revisión de la plantilla de lanzamiento

El propósito de esta sección es mostrar cómo incluir un paso de comprobación de estado en la plantilla de lanzamiento.

1. Abra el archivo **CreateADMRollout.json** que creó en [Uso del Administrador de implementaciones de Azure con plantillas de Resource Manager](./deployment-manager-tutorial.md). Este archivo JSON forma parte de la descarga.  Consulte [Requisitos previos](#prerequisites).
1. Agregue dos parámetros más:

    ```json
    "healthCheckUrl": {
      "type": "string",
      "metadata": {
        "description": "Specifies the health check URL."
      }
    },
    "healthCheckAuthAPIKey": {
      "type": "string",
      "metadata": {
          "description": "Specifies the health check Azure Function function authorization key."
      }
    }
    ```

1. Reemplace la definición de recursos del paso de espera por una definición de recursos del paso de comprobación del estado:

    ```json
    {
      "type": "Microsoft.DeploymentManager/steps",
      "apiVersion": "2018-09-01-preview",
      "name": "healthCheckStep",
      "location": "[parameters('azureResourceLocation')]",
      "properties": {
        "stepType": "healthCheck",
        "attributes": {
          "waitDuration": "PT0M",
          "maxElasticDuration": "PT0M",
          "healthyStateDuration": "PT1M",
          "type": "REST",
          "properties": {
            "healthChecks": [
              {
                "name": "appHealth",
                "request": {
                  "method": "GET",
                  "uri": "[parameters('healthCheckUrl')]",
                  "authentication": {
                    "type": "ApiKey",
                    "name": "code",
                    "in": "Query",
                    "value": "[parameters('healthCheckAuthAPIKey')]"
                  }
                },
                "response": {
                  "successStatusCodes": [
                    "200"
                  ],
                  "regex": {
                    "matches": [
                      "Status: healthy",
                      "Status: warning"
                    ],
                    "matchQuantifier": "Any"
                  }
                }
              }
            ]
          }
        }
      }
    },
    ```

    Según la definición, el lanzamiento continúa si el estado de mantenimiento es *healthy* o *warning*.

1. Actualice la sección **dependsON** de la definición del lanzamiento para incluir el paso de comprobación del estado recién definido:

    ```json
    "dependsOn": [
      "[resourceId('Microsoft.DeploymentManager/artifactSources', variables('rolloutArtifactSource').name)]",
      "[resourceId('Microsoft.DeploymentManager/steps/', 'healthCheckStep')]"
    ],
    ```

1. Actualice **stepGroups** para incluir el paso de comprobación del estado. **healthCheckStep** se llama en **postDeploymentSteps** de **stepGroup2**. **stepGroup3** y **stepGroup4** solo se implementan si el estado es *healthy* o *warning*.

    ```json
    "stepGroups": [
      {
        "name": "stepGroup1",
          "preDeploymentSteps": [],
          "deploymentTargetId": "[resourceId('Microsoft.DeploymentManager/serviceTopologies/services/serviceUnits', variables('serviceTopology').name, variables('serviceTopology').serviceWUS.name,  variables('serviceTopology').serviceWUS.serviceUnit2.name)]",
          "postDeploymentSteps": []
        },
        {
          "name": "stepGroup2",
          "dependsOnStepGroups": ["stepGroup1"],
          "preDeploymentSteps": [],
          "deploymentTargetId": "[resourceId('Microsoft.DeploymentManager/serviceTopologies/services/serviceUnits', variables('serviceTopology').name, variables('serviceTopology').serviceWUS.name,  variables('serviceTopology').serviceWUS.serviceUnit1.name)]",
          "postDeploymentSteps": [
            {
              "stepId": "[resourceId('Microsoft.DeploymentManager/steps/', 'healthCheckStep')]"
            }
          ]
        },
        {
          "name": "stepGroup3",
          "dependsOnStepGroups": ["stepGroup2"],
          "preDeploymentSteps": [],
          "deploymentTargetId": "[resourceId('Microsoft.DeploymentManager/serviceTopologies/services/serviceUnits', variables('serviceTopology').name, variables('serviceTopology').serviceEUS.name,  variables('serviceTopology').serviceEUS.serviceUnit2.name)]",
           "postDeploymentSteps": []
        },
        {
          "name": "stepGroup4",
          "dependsOnStepGroups": ["stepGroup3"],
          "preDeploymentSteps": [],
          "deploymentTargetId": "[resourceId('Microsoft.DeploymentManager/serviceTopologies/services/serviceUnits', variables('serviceTopology').name, variables('serviceTopology').serviceEUS.name,  variables('serviceTopology').serviceEUS.serviceUnit1.name)]",
          "postDeploymentSteps": []
        }
    ]
    ```

    Si compara la sección **stepGroup3** sección antes y después de que se revise, se observará que sección ahora depende de **stepGroup2**.  Esto es necesario cuando tanto **stepGroup3** como los grupos de pasos posteriores dependen de los resultados del seguimiento del estado.

    La siguiente captura de pantalla ilustra las áreas modificadas y cómo se usa el paso de comprobación del estado:

    ![Plantilla de comprobación del estado de Azure Deployment Manager](./media/deployment-manager-tutorial-health-check/azure-deployment-manager-hc-rollout-template.png)

## <a name="deploy-the-topology"></a>Implementación de la topología

Ejecute el siguiente script de PowerShell para implementar la topología. Necesita los mismos archivos **CreateADMServiceTopology.json** y **CreateADMServiceTopology.Parameters.json** que usó en [Uso del Administrador de implementaciones de Azure con plantillas de Resource Manager](./deployment-manager-tutorial.md).

```azurepowershell
# Create the service topology
New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile "$filePath\ADMTemplates\CreateADMServiceTopology.json" `
    -TemplateParameterFile "$filePath\ADMTemplates\CreateADMServiceTopology.Parameters.json"
```

Compruebe la topología del servicio y que los recursos subyacentes se hayan creado correctamente mediante Azure Portal:

![Tutorial de Azure Deployment Manager: recursos de topología del servicio implementados](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-deployed-topology-resources.png)

Se debe seleccionar **Mostrar tipos ocultos** para ver los recursos.

## <a name="deploy-the-rollout-with-the-unhealthy-status"></a>Implementación del lanzamiento con estado incorrecto

Use la dirección URL del estado incorrecto que creó en [Creación de un simulador del servicio de comprobación de estado](#create-a-health-check-service-simulator). Necesita el archivo **CreateADMServiceTopology.json** revisado y el mismo **CreateADMServiceTopology.Parameters.json** que usó en [Uso del Administrador de implementaciones de Azure con plantillas de Resource Manager](./deployment-manager-tutorial.md).

```azurepowershell-interactive
$healthCheckUrl = Read-Host -Prompt "Enter the health check Azure function URL"
$healthCheckAuthAPIKey = $healthCheckUrl.Substring($healthCheckUrl.IndexOf("?code=")+6, $healthCheckUrl.Length-$healthCheckUrl.IndexOf("?code=")-6)
$healthCheckUrl = $healthCheckUrl.Substring(0, $healthCheckUrl.IndexOf("?"))

# Create the rollout
New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile "$filePath\ADMTemplates\CreateADMRollout.json" `
    -TemplateParameterFile "$filePath\ADMTemplates\CreateADMRollout.Parameters.json" `
    -healthCheckUrl $healthCheckUrl `
    -healthCheckAuthAPIKey $healthCheckAuthAPIKey
```

> [!NOTE]
> `New-AzResourceGroupDeployment` es una llamada asincrónica. El mensaje de operación correcta solo significa que la implementación ha comenzado correctamente. Para comprobar la implementación, use `Get-AZDeploymentManagerRollout`.  Consulte el procedimiento siguiente.

Para comprobar el progreso del lanzamiento, use el siguiente script de PowerShell:

```azurepowershell
$projectName = Read-Host -Prompt "Enter the same project name used earlier in this tutorial"
$resourceGroupName = "${projectName}rg"
$rolloutName = "${projectName}Rollout"

# Get the rollout status
Get-AzDeploymentManagerRollout `
    -ResourceGroupName $resourceGroupName `
    -Name $rolloutName `
    -Verbose
```

La siguiente salida de ejemplo muestra el error de la implementación debido al estado incorrecto:

```output
Service: myhc0417ServiceWUSrg
    TargetLocation: WestUS
    TargetSubscriptionId: <Subscription ID>

    ServiceUnit: myhc0417ServiceWUSWeb
        TargetResourceGroup: myhc0417ServiceWUSrg

        Step: RestHealthCheck/healthCheckStep.PostDeploy
            Status: Failed
            StepGroup: stepGroup2
            Operation Info:
                Start Time: 05/06/2019 17:58:31
                End Time: 05/06/2019 17:58:32
                Total Duration: 00:00:01
                Error:
                    Code: ResourceReportedUnhealthy
                    Message: Health checks failed as the following resources were unhealthy: '05/06/2019 17:58:32 UTC: Health check 'appHealth' failed with the following errors: Response from endpoint 'https://myhc0417webapp.azurewebsites.net/api/healthStatus/unhealthy' does not match the regex pattern(s): 'Status: healthy, Status: warning.'. Response content: "Status: unhealthy"..'.
Get-AzDeploymentManagerRollout :
Service: myhc0417ServiceWUSrg
    ServiceUnit: myhc0417ServiceWUSWeb
        Step: RestHealthCheck/healthCheckStep.PostDeploy
            Status: Failed
            StepGroup: stepGroup2
            Operation Info:
                Start Time: 05/06/2019 17:58:31
                End Time: 05/06/2019 17:58:32
                Total Duration: 00:00:01
                Error:
                    Code: ResourceReportedUnhealthy
                    Message: Health checks failed as the following resources were unhealthy: '05/06/2019 17:58:32 UTC: Health check 'appHealth' failed with the following errors: Response from endpoint 'https://myhc0417webapp.azurewebsites.net/api/healthStatus/unhealthy' does not match the regex pattern(s): 'Status: healthy, Status: warning.'. Response content: "Status: unhealthy"..'.
At line:1 char:1
+ Get-AzDeploymentManagerRollout `
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
+ CategoryInfo          : NotSpecified: (:) [Get-AzDeploymentManagerRollout], Exception
+ FullyQualifiedErrorId : RolloutFailed,Microsoft.Azure.Commands.DeploymentManager.Commands.GetRollout


ResourceGroupName       : myhc0417rg
BuildVersion            : 1.0.0.0
ArtifactSourceId        : /subscriptions/<Subscription ID>/resourceGroups/myhc0417rg/providers/Mi
                          crosoft.DeploymentManager/artifactSources/myhc0417ArtifactSourceRollout
TargetServiceTopologyId : /subscriptions/<Subscription ID>/resourceGroups/myhc0417rg/providers/Mi
                          crosoft.DeploymentManager/serviceTopologies/myhc0417ServiceTopology
Status                  : Failed
TotalRetryAttempts      : 0
Identity                : Microsoft.Azure.Commands.DeploymentManager.Models.PSIdentity
OperationInfo           : Microsoft.Azure.Commands.DeploymentManager.Models.PSRolloutOperationInfo
Services                : {myhc0417ServiceWUS, myhc0417ServiceWUSrg}
Name                    : myhc0417Rollout
Type                    : Microsoft.DeploymentManager/rollouts
Location                : centralus
Id                      : /subscriptions/<Subscription ID>/resourcegroups/myhc0417rg/providers/Mi
                          crosoft.DeploymentManager/rollouts/myhc0417Rollout
Tags                    :
```

Una vez que se complete el lanzamiento, verá que se ha creado un grupo de recursos adicional para Oeste de EE. UU.

## <a name="deploy-the-rollout-with-the-healthy-status"></a>Implementación del lanzamiento con estado correcto

Repita esta sección para volver a implementar el lanzamiento con la dirección URL de estado correcto.  Una vez que se complete el lanzamiento, verá que se ha creado un grupo de recursos más para Este de EE. UU.

## <a name="verify-the-deployment"></a>Comprobar la implementación

1. Abra [Azure Portal](https://portal.azure.com).
2. Vaya a las aplicaciones web recién creadas en los grupos de recursos creados mediante la implementación de lanzamiento.
3. Abra la aplicación web en un explorador web. Compruebe la ubicación y la versión del archivo index.html.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando los recursos de Azure ya no sean necesarios, limpie los recursos que implementó eliminando el grupo de recursos.

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
2. Use el campo **Filtrar por nombre** para limitar los grupos de recursos creados en este tutorial. Habrá 3 o 4:

    * **&lt;projectName>rg**: contiene los recursos del Administrador de implementaciones.
    * **&lt;projectName>ServiceWUSrg**: contiene los recursos definidos por ServiceWUS.
    * **&lt;projectName>ServiceEUSrg**: contiene los recursos definidos por ServiceEUS.
    * El grupo de recursos de la identidad administrada definida por el usuario.
3. Seleccione el nombre del grupo de recursos.
4. Seleccione **Eliminar grupo de recursos** del menú superior.
5. Repita los dos últimos pasos para eliminar otros grupos de recursos creados en este tutorial.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a la característica de comprobación de estado de Azure Deployment Manager. Para más información, consulte la [documentación de Azure Resource Manager](../index.yml).
