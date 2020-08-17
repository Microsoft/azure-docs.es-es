---
title: 'Administración del entorno mediante plantillas de Azure Resource Manager: Azure Time Series Insights | Microsoft Docs'
description: Aprenda a administrar el entorno de Azure Time Series Insights mediante programación con Azure Resource Manager.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.devlang: csharp
ms.workload: big-data
ms.topic: conceptual
ms.date: 06/30/2020
ms.custom: seodec18
ms.openlocfilehash: 3e9075014863e653a986dc4dbec7b9bc5e9f31bc
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87421202"
---
# <a name="create-azure-time-series-insights-gen-1-resources-using-azure-resource-manager-templates"></a>Creación de recursos de Azure Time Series Insights Gen 1 mediante plantillas de Azure Resource Manager

En este artículo se describe cómo crear e implementar recursos de Azure Time Series Insights mediante [plantillas de Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/), PowerShell y el proveedor de recursos de Azure Time Series Insights.

Azure Time Series Insights admite los siguientes recursos:

   | Recurso | Descripción |
   | --- | --- |
   | Entorno | Un entorno de Azure Time Series Insights es una agrupación lógica de eventos que se leen en los agentes de eventos, se almacenan y están disponibles para consulta. Para más información, lea [Planeamiento del entorno de Azure Time Series Insights](time-series-insights-environment-planning.md). |
   | Origen de eventos | Un origen de eventos es una conexión a un agente de eventos del que Azure Time Series Insights lee e ingiere eventos en el entorno. Los orígenes de eventos actualmente admitidos son IoT Hub y Event Hub. |
   | Conjuntos de datos de referencia | Los conjuntos de datos de referencia proporcionan metadatos sobre los eventos del entorno. Los metadatos de los conjuntos de datos de referencia se combinarán con los eventos durante la entrada. Los conjuntos de datos de referencia se definen como recursos según sus propiedades de eventos principales. Los metadatos reales que constituye el conjunto de datos de referencia se cargan o se modifican mediante las API de plano de datos. |
   | Directiva de acceso | Las directivas de acceso conceden permisos para emitir consultas de datos, manipular datos de referencia en el entorno y compartir consultas guardadas y perspectivas asociadas con el entorno. Para obtener más información, consulte [Concesión de acceso a datos en un entorno de Azure Time Series Insights mediante Azure Portal](time-series-insights-data-access.md). |

Una plantilla de Resource Manager es un archivo JSON que define la infraestructura y la configuración de recursos en un grupo de recursos. En los siguientes documentos se describen los archivos de plantilla con mayor detalle:

- [Implementaciones de plantillas de Azure Resource Manager](../azure-resource-manager/templates/overview.md)
- [Implementación de recursos con las plantillas de Resource Manager y Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md)
- [Tipos de recursos de Microsoft.TimeSeriesInsights](/azure/templates/microsoft.timeseriesinsights/allversions)

La plantilla de inicio rápido [201-timeseriesinsights-environment-with-eventhub](https://github.com/Azure/azure-quickstart-templates/tree/master/201-timeseriesinsights-environment-with-eventhub) está publicada en GitHub. Esta plantilla crea un entorno de Azure Time Series Insights, un origen de eventos secundario configurado para consumir eventos de un centro de eventos y directivas de acceso que conceden acceso a los datos del entorno. Si no se especifica un centro de eventos existente, se crea uno con la implementación.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="specify-deployment-template-and-parameters"></a>Especificación de la plantilla y los parámetros de implementación

En el procedimiento siguiente se describe cómo usar PowerShell para implementar una plantilla de Azure Resource Manager que crea un entorno de Azure Time Series Insights, un origen de eventos secundarios configurado para consumir eventos de un centro de eventos y directivas de acceso que conceden acceso a los datos del entorno. Si no se especifica un centro de eventos existente, se crea uno con la implementación.

1. Instale Azure PowerShell siguiendo las instrucciones de [Getting started with Azure PowerShell](https://docs.microsoft.com/powershell/azure/get-started-azureps) (Introducción a Azure PowerShell).

1. Clone o copie la plantilla [201-timeseriesinsights-environment-with-eventhub](https://github.com/Azure/azure-quickstart-templates/blob/master/201-timeseriesinsights-environment-with-eventhub/azuredeploy.json) de GitHub.

   * Creación de un archivo de parámetros

     Para crear un archivo de parámetros, copie el archivo [201-timeseriesinsights-environment-with-eventhub](https://github.com/Azure/azure-quickstart-templates/blob/master/201-timeseriesinsights-environment-with-eventhub/azuredeploy.parameters.json).

      [!code-json[deployment-parameters](~/quickstart-templates/201-timeseriesinsights-environment-with-eventhub/azuredeploy.parameters.json)]

    <div id="required-parameters"></div>

   * Parámetros obligatorios

     | Parámetro | Descripción |
     | --- | --- |
     | eventHubNamespaceName | El espacio de nombres del centro de eventos de origen. |
     | eventHubName | El nombre del centro de eventos de origen. |
     | consumerGroupName | El nombre del grupo de consumidores que usará el servicio Azure Time Series Insights para leer los datos del centro de eventos. **NOTA:** Para evitar la contención de los recursos, este grupo de consumidores tiene que estar dedicado al servicio Azure Time Series Insights y no se debe compartir con otros lectores. |
     | environmentName | El nombre del entorno. El nombre no puede incluir los caracteres `<`, `>`, `%`, `&`, `:`, `\\`, `?`, `/`, ni ningún carácter de control. Todos los demás caracteres se permiten.|
     | eventSourceName | El nombre del recurso de secundarios del origen de eventos. El nombre no puede incluir los caracteres `<`, `>`, `%`, `&`, `:`, `\\`, `?`, `/`, ni ningún carácter de control. Todos los demás caracteres se permiten. |

    <div id="optional-parameters"></div>

   * Parámetros opcionales

     | Parámetro | Descripción |
     | --- | --- |
     | existingEventHubResourceId | Un identificador de recurso opcional de un centro de eventos existente que se conectará al entorno de Azure Time Series Insights mediante el origen de eventos. **NOTA:** El usuario que implementa la plantilla tiene que tener privilegios para realizar la operación listkeys en el centro de eventos. Si no se pasa ningún valor, se crea un nuevo centro de eventos mediante la plantilla. |
     | environmentDisplayName | Un nombre descriptivo opcional que se muestra en las herramientas o las interfaces de usuario en el lugar del nombre del entorno. |
     | environmentSkuName | El nombre de la SKU. Para más información, lea la [página de precios de Azure Time Series Insights](https://azure.microsoft.com/pricing/details/time-series-insights/).  |
     | environmentSkuCapacity | La capacidad unitaria de la SKU. Para más información, lea la [página de precios de Azure Time Series Insights](https://azure.microsoft.com/pricing/details/time-series-insights/).|
     | environmentDataRetentionTime | El intervalo de tiempo mínimo que los eventos del entorno estarán disponibles para consulta. El valor debe especificarse en formato ISO 8601, por ejemplo, `P30D` para una directiva de retención de 30 días. |
     | eventSourceDisplayName | Un nombre descriptivo opcional que se muestra en las herramientas o las interfaces de usuario en el lugar del nombre del origen de eventos. |
     | eventSourceTimestampPropertyName | La propiedad de evento que se usará como intervalo de tiempo del origen de eventos. Si no se especifica un valor para timestampPropertyName, o si se especifica una cadena null o vacía, se usará la hora de creación del evento. |
     | eventSourceKeyName | El nombre de la clave de acceso compartido que usará el servicio Azure Time Series Insights para conectarse al centro de eventos. |
     | accessPolicyReaderObjectIds | Una lista de identificadores de objetos de los usuarios o aplicaciones de Azure AD que deben tener acceso de lectura en el entorno. Se puede obtener el valor de objectId de la entidad de servicio mediante la llamada a los cmdlets **Get-AzADUser** o **Get-AzADServicePrincipal**. Aún no se admite la creación de una directiva de acceso para grupos de Azure AD. |
     | accessPolicyContributorObjectIds | Una lista de identificadores de objeto de los usuarios o aplicaciones de Azure AD que deben tener acceso de colaborador al entorno. Se puede obtener el valor de objectId de la entidad de servicio mediante la llamada a los cmdlets **Get-AzADUser** o **Get-AzADServicePrincipal**. Aún no se admite la creación de una directiva de acceso para grupos de Azure AD. |

   * Como ejemplo, el siguiente archivo de parámetros se usaría para crear un entorno y un origen de eventos que lee los eventos de un centro de eventos existente. También se crean dos directivas de acceso que conceden acceso de colaborador al entorno.

     ```JSON
     {
         "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
         "contentVersion": "1.0.0.0",
         "parameters": {
             "eventHubNamespaceName": {
                 "value": "tsiTemplateTestNamespace"
             },
             "eventHubName": {
                 "value": "tsiTemplateTestEventHub"
             },
             "consumerGroupName": {
                 "value": "tsiTemplateTestConsumerGroup"
             },
             "environmentName": {
                 "value": "tsiTemplateTestEnvironment"
             },
             "eventSourceName": {
                 "value": "tsiTemplateTestEventSource"
             },
             "existingEventHubResourceId": {
                 "value": "/subscriptions/{yourSubscription}/resourceGroups/MyDemoRG/providers/Microsoft.EventHub/namespaces/tsiTemplateTestNamespace/eventhubs/tsiTemplateTestEventHub"
             },
             "accessPolicyContributorObjectIds": {
                 "value": [
                     "AGUID001-0000-0000-0000-000000000000",
                     "AGUID002-0000-0000-0000-000000000000"
                 ]
             }    
         }
     }
     ```

    * Para más información, lea el artículo [Parámetros](../azure-resource-manager/templates/parameter-files.md).

## <a name="deploy-the-quickstart-template-locally-using-powershell"></a>Implementación de la plantilla de inicio rápido localmente mediante PowerShell

> [!IMPORTANT]
> Las operaciones de línea de comandos que aparecen a continuación se describen en el [módulo de Az PowerShell](https://docs.microsoft.com/powershell/azure/).

1. En PowerShell, inicie sesión en la cuenta de Azure.

    * En una secuencia de comandos de PowerShell, ejecute el siguiente comando:

      ```powershell
      Connect-AzAccount
      ```

    * Se le solicitará que inicie sesión en la cuenta de Azure. Después de iniciar sesión, ejecute el siguiente comando para ver sus suscripciones disponibles:

      ```powershell
      Get-AzSubscription
      ```

    * Este comando devuelve una lista de suscripciones de Azure disponibles. Seleccione una suscripción para la sesión actual mediante la ejecución del siguiente comando. Reemplace `<YourSubscriptionId>` por el GUID de la suscripción de Azure que desea usar:

      ```powershell
      Set-AzContext -SubscriptionID <YourSubscriptionId>
      ```

1. Si no existe, cree un nuevo grupo de recursos.

   * Si no tiene un grupo de recursos existente, cree uno con el comando **New-AzResourceGroup**. Proporcione el nombre del grupo de recursos y la ubicación que desee utilizar. Por ejemplo:

     ```powershell
     New-AzResourceGroup -Name MyDemoRG -Location "West US"
     ```

   * Si es correcto, se muestra un resumen del nuevo grupo de recursos.

     ```powershell
     ResourceGroupName : MyDemoRG
     Location          : westus
     ProvisioningState : Succeeded
     Tags              :
     ResourceId        : /subscriptions/<GUID>/resourceGroups/MyDemoRG
     ```

1. Pruebe la implementación.

   * Valide la implementación mediante la ejecución del cmdlet `Test-AzResourceGroupDeployment`. Al probar la implementación, proporcione los parámetros exactamente como lo haría al ejecutar la implementación.

     ```powershell
     Test-AzResourceGroupDeployment -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json -TemplateParameterFile <path to parameters file>\azuredeploy.parameters.json
     ```

1. Creación de la implementación

    * Para crear la nueva implementación, ejecute el cmdlet `New-AzResourceGroupDeployment` y proporcione los parámetros necesarios cuando se le solicite. Los parámetros incluyen un nombre para la implementación, el nombre del grupo de recursos y la ruta de acceso o la dirección URL al archivo de plantilla. Si no se especifica el parámetro **Modo**, se usa el valor predeterminado **Incremental**. Para más información, lea [Implementaciones incrementales y completas](../azure-resource-manager/templates/deployment-modes.md).

    * El siguiente comando le solicita los cinco parámetros necesarios en la ventana de PowerShell:

      ```powershell
      New-AzResourceGroupDeployment -Name MyDemoDeployment -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json
      ```

    * Para especificar un archivo de parámetros en su lugar, use el siguiente comando:

      ```powershell
      New-AzResourceGroupDeployment -Name MyDemoDeployment -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json -TemplateParameterFile <path to parameters file>\azuredeploy.parameters.json
      ```

    * También puede utilizar parámetros en línea cuando ejecute el cmdlet de implementación. El comando es el siguiente:

      ```powershell
      New-AzResourceGroupDeployment -Name MyDemoDeployment -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json -parameterName "parameterValue"
      ```

    * Para ejecutar una implementación [completa](../azure-resource-manager/templates/deployment-modes.md), establezca el parámetro **Modo** en **Completo**:

      ```powershell
      New-AzResourceGroupDeployment -Name MyDemoDeployment -Mode Complete -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json
      ```

1. Comprobar la implementación

    * Si los recursos se implementan correctamente, aparecerá un resumen de la implementación en la ventana de PowerShell:

      ```powershell
       DeploymentName          : MyDemoDeployment
       ResourceGroupName       : MyDemoRG
       ProvisioningState       : Succeeded
       Timestamp               : 10/11/2019 3:20:37 AM
       Mode                    : Incremental
       TemplateLink            :
       Parameters              :
                                 Name                                Type                       Value
                                 ==================================  =========================  ==========
                                 eventHubNewOrExisting               String                     new
                                 eventHubResourceGroup               String                     MyDemoRG
                                 eventHubNamespaceName               String                     tsiquickstartns
                                 eventHubName                        String                     tsiquickstarteh
                                 consumerGroupName                   String                     tsiquickstart
                                 environmentName                     String                     tsiquickstart
                                 environmentDisplayName              String                     tsiquickstart
                                 environmentSkuName                  String                     S1
                                 environmentSkuCapacity              Int                        1
                                 environmentDataRetentionTime        String                     P30D
                                 eventSourceName                     String                     tsiquickstart
                                 eventSourceDisplayName              String                     tsiquickstart
                                 eventSourceTimestampPropertyName    String
                                 eventSourceKeyName                  String                     manage
                                 accessPolicyReaderObjectIds         Array                      []
                                 accessPolicyContributorObjectIds    Array                      []
                                 location                            String                     westus

       Outputs                 :
                                  Name              Type                       Value
                                  ================  =========================  ==========
                                  dataAccessFQDN    String
                                  11aa1aa1-a1aa-1a1a-a11a-aa111a111a11.env.timeseries.azure.com

       DeploymentDebugLogLevel :
      ```

1. Implementación de la plantilla de inicio rápido mediante Azure Portal

   * La página principal de la plantilla de inicio rápido de GitHub incluye también un botón **Deploy to Azure** (Implementar en Azure). Al hacer clic en él se abre una página de implementación personalizado en Azure Portal. En esta página, puede definir o seleccionar valores para cada uno de los parámetros en las tablas de [parámetros obligatorios](#required-parameters) o [parámetros opcionales](#optional-parameters). Después de rellenar la configuración, al hacer clic en el botón **Purchase** (Comprar) se inicia la implementación de la plantilla.
    </br>
    </br>
    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-timeseriesinsights-environment-with-eventhub%2Fazuredeploy.json" target="_blank">
       <img src="https://azuredeploy.net/deploybutton.png" alt="The Deploy to Azure button."/>
    </a>

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre la administración de recursos de Azure Time Series Insights mediante programación por medio de las API REST, lea [Administración de Azure Time Series Insights](https://docs.microsoft.com/rest/api/time-series-insights-management/).
