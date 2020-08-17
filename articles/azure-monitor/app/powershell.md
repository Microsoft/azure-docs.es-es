---
title: Automatización de Azure Application Insights con PowerShell | Microsoft Docs
description: Automatice la creación y administración de recursos, alertas y pruebas de disponibilidad en PowerShell mediante una plantilla de Azure Resource Manager.
ms.topic: conceptual
ms.date: 05/02/2020
ms.openlocfilehash: 53cdf338db5cc4ea359f729297fe57e63853aa5c
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87322489"
---
#  <a name="manage-application-insights-resources-using-powershell"></a>Administración de recursos de Application Insights mediante PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Este artículo muestra cómo automatizar la creación y actualización de los recursos de [Application Insights](./app-insights-overview.md) automáticamente mediante Azure Resource Management. Puede hacerlo, por ejemplo, como parte de un proceso de compilación. Junto con el recurso básico de Application Insights, puede crear [pruebas web de disponibilidad](./monitor-web-app-availability.md), [configurar alertas](../platform/alerts-log.md), establecer el [esquema de precios](pricing.md) y crear otros recursos de Azure.

La clave para crear estos recursos es las plantillas JSON para el [Administrador de recursos de Azure](../../azure-resource-manager/management/manage-resources-powershell.md). El procedimiento básico es: descargar las definiciones JSON de los recursos existentes; parametrizar determinados valores como los nombres; y luego ejecutar la plantilla siempre que se quiera crear un nuevo recurso. Puede empaquetar varios recursos juntos para crearlos todos en un solo paso, por ejemplo, un monitor de aplicaciones con pruebas de disponibilidad, alertas y almacenamiento para la exportación continua. Existen algunos matices a algunas de las parametrizaciones automáticas, que se explican aquí.

## <a name="one-time-setup"></a>Instalación única
Si no ha usado PowerShell con su suscripción de Azure antes:

Instale el módulo de Azure PowerShell en el equipo donde desea ejecutar los scripts:

1. Instale el [Instalador de plataforma web de Microsoft (v5 o superior)](https://www.microsoft.com/web/downloads/platform.aspx).
2. Úselo para instalar Microsoft Azure PowerShell.

Además de usar plantillas de Resource Manager, existe un completo conjunto de [cmdlets de PowerShell en Application Insights ](/powershell/module/az.applicationinsights) que facilitan la configuración de recursos de Application Insights mediante programación. Entre las funcionalidades que habilitan los cmdlets se incluyen las siguientes:

* Crear y eliminar recursos de Application Insights
* Obtener listas de recursos de Application Insights y sus propiedades
* Crear y administrar la exportación continua
* Crear y administrar claves de aplicación
* Establecer el límite diario
* Establecer el plan de precios

## <a name="create-application-insights-resources-using-a-powershell-cmdlet"></a>Creación de recursos de Application Insights mediante un cmdlet de PowerShell

Aquí se muestra cómo crear un nuevo recurso de Application Insights en el centro de datos Este de EE. UU. de Azure con el cmdlet [New-AzApplicationInsights](/powershell/module/az.applicationinsights/new-azapplicationinsights):

```PS
New-AzApplicationInsights -ResourceGroupName <resource group> -Name <resource name> -location eastus
```


## <a name="create-application-insights-resources-using-a-resource-manager-template"></a>Creación de recursos de Application Insights mediante una plantilla de Resource Manager

Aquí se muestra cómo crear un nuevo recurso de Application Insights mediante una plantilla de Resource Manager.

### <a name="create-the-azure-resource-manager-template"></a>Creación de la plantilla de Azure Resource Manager

Cree un nuevo archivo .json. Vamos a llamarlo `template1.json` en este ejemplo. Copie este contenido en él:

```JSON
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "appName": {
                "type": "string",
                "metadata": {
                    "description": "Enter the name of your Application Insights resource."
                }
            },
            "appType": {
                "type": "string",
                "defaultValue": "web",
                "allowedValues": [
                    "web",
                    "java",
                    "other"
                ],
                "metadata": {
                    "description": "Enter the type of the monitored application."
                }
            },
            "appLocation": {
                "type": "string",
                "defaultValue": "eastus",
                "metadata": {
                    "description": "Enter the location of your Application Insights resource."
                }
            },
            "retentionInDays": {
                "type": "int",
                "defaultValue": 90,
                "allowedValues": [
                    30,
                    60,
                    90,
                    120,
                    180,
                    270,
                    365,
                    550,
                    730
                ],
                "metadata": {
                    "description": "Data retention in days"
                }
            },
            "ImmediatePurgeDataOn30Days": {
                "type": "bool",
                "defaultValue": false,
                "metadata": {
                    "description": "If set to true when changing retention to 30 days, older data will be immediately deleted. Use this with extreme caution. This only applies when retention is being set to 30 days."
                }
            },
            "priceCode": {
                "type": "int",
                "defaultValue": 1,
                "allowedValues": [
                    1,
                    2
                ],
                "metadata": {
                    "description": "Pricing plan: 1 = Per GB (or legacy Basic plan), 2 = Per Node (legacy Enterprise plan)"
                }
            },
            "dailyQuota": {
                "type": "int",
                "defaultValue": 100,
                "minValue": 1,
                "metadata": {
                    "description": "Enter daily quota in GB."
                }
            },
            "dailyQuotaResetTime": {
                "type": "int",
                "defaultValue": 0,
                "metadata": {
                    "description": "Enter daily quota reset hour in UTC (0 to 23). Values outside the range will get a random reset hour."
                }
            },
            "warningThreshold": {
                "type": "int",
                "defaultValue": 90,
                "minValue": 1,
                "maxValue": 100,
                "metadata": {
                    "description": "Enter the % value of daily quota after which warning mail to be sent. "
                }
            }
        },
        "variables": {
            "priceArray": [
                "Basic",
                "Application Insights Enterprise"
            ],
            "pricePlan": "[take(variables('priceArray'),parameters('priceCode'))]",
            "billingplan": "[concat(parameters('appName'),'/', variables('pricePlan')[0])]"
        },
        "resources": [
            {
                "type": "microsoft.insights/components",
                "kind": "[parameters('appType')]",
                "name": "[parameters('appName')]",
                "apiVersion": "2014-04-01",
                "location": "[parameters('appLocation')]",
                "tags": {},
                "properties": {
                    "ApplicationId": "[parameters('appName')]",
                    "retentionInDays": "[parameters('retentionInDays')]"
                },
                "dependsOn": []
            },
            {
                "name": "[variables('billingplan')]",
                "type": "microsoft.insights/components/CurrentBillingFeatures",
                "location": "[parameters('appLocation')]",
                "apiVersion": "2015-05-01",
                "dependsOn": [
                    "[resourceId('microsoft.insights/components', parameters('appName'))]"
                ],
                "properties": {
                    "CurrentBillingFeatures": "[variables('pricePlan')]",
                    "DataVolumeCap": {
                        "Cap": "[parameters('dailyQuota')]",
                        "WarningThreshold": "[parameters('warningThreshold')]",
                        "ResetTime": "[parameters('dailyQuotaResetTime')]"
                    }
                }
            }
        ]
    }
```

### <a name="use-the-resource-manager-template-to-create-a-new-application-insights-resource"></a>Uso de la plantilla de Resource Manager para crear un nuevo recurso de Application Insights

1. En PowerShell, inicie sesión en Azure mediante `$Connect-AzAccount`.
2. Establezca el contexto en una suscripción con `Set-AzContext "<subscription ID>"`.
2. Ejecute una nueva implementación para crear un nuevo recurso de Application Insights:
   
    ```PS
        New-AzResourceGroupDeployment -ResourceGroupName Fabrikam `
               -TemplateFile .\template1.json `
               -appName myNewApp

    ``` 
   
   * `-ResourceGroupName` es el grupo donde desea crear los nuevos recursos.
   * `-TemplateFile` debe aparecer antes que los parámetros personalizados.
   * `-appName` es el nombre del recurso que se va a crear.

Puede agregar otros parámetros, cuyas descripciones se encuentran en la sección de parámetros de la plantilla.

## <a name="get-the-instrumentation-key"></a>Obtención de la clave de instrumentación

Después de crear un recurso de aplicación, querrá la clave de instrumentación: 

1. `$Connect-AzAccount`
2. `Set-AzContext "<subscription ID>"`
3. `$resource = Get-AzResource -Name "<resource name>" -ResourceType "Microsoft.Insights/components"`
4. `$details = Get-AzResource -ResourceId $resource.ResourceId`
5. `$details.Properties.InstrumentationKey`

Para ver una lista de muchas otras propiedades del recurso de Application Insights, use el siguiente comando:

```PS
Get-AzApplicationInsights -ResourceGroupName Fabrikam -Name FabrikamProd | Format-List
```

Dispone de propiedades adicionales a través de los siguientes cmdlets:
* `Set-AzApplicationInsightsDailyCap`
* `Set-AzApplicationInsightsPricingPlan`
* `Get-AzApplicationInsightsApiKey`
* `Get-AzApplicationInsightsContinuousExport`

Consulte la [documentación detallada](/powershell/module/az.applicationinsights) para información sobre los parámetros de estos cmdlets.  

## <a name="set-the-data-retention"></a>Establecimiento de la retención de datos

A continuación se muestran tres métodos para establecer la retención de datos mediante programación en un recurso de Application Insights.

### <a name="setting-data-retention-using-a-powershell-commands"></a>Establecimiento de la retención de datos mediante comandos de PowerShell

A continuación se muestra un conjunto sencillo de comandos de PowerShell para establecer la retención de datos para el recurso de Application Insights:

```PS
$Resource = Get-AzResource -ResourceType Microsoft.Insights/components -ResourceGroupName MyResourceGroupName -ResourceName MyResourceName
$Resource.Properties.RetentionInDays = 365
$Resource | Set-AzResource -Force
```

### <a name="setting-data-retention-using-rest"></a>Establecimiento de la retención de datos mediante REST

Para obtener la retención actual de datos del recurso de Application Insights, puede usar la herramienta OSS [ARMClient](https://github.com/projectkudu/ARMClient)  (puede obtener más información sobre ARMClient en los artículos de [David Ebbo](http://blog.davidebbo.com/2015/01/azure-resource-manager-client.html) y [Daniel Bowbyes](https://blog.bowbyes.co.nz/2016/11/02/using-armclient-to-directly-access-azure-arm-rest-apis-and-list-arm-policy-details/)).  Este es un ejemplo del uso de `ARMClient` para obtener la retención actual:

```PS
armclient GET /subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/MyResourceGroupName/providers/microsoft.insights/components/MyResourceName?api-version=2018-05-01-preview
```

Para establecer la retención, se usa un comando PUT similar:

```PS
armclient PUT /subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/MyResourceGroupName/providers/microsoft.insights/components/MyResourceName?api-version=2018-05-01-preview "{location: 'eastus', properties: {'retentionInDays': 365}}"
```

Para establecer la retención de datos en 365 días mediante la plantilla anterior, ejecute:

```PS
New-AzResourceGroupDeployment -ResourceGroupName "<resource group>" `
       -TemplateFile .\template1.json `
       -retentionInDays 365 `
       -appName myApp
```

### <a name="setting-data-retention-using-a-powershell-script"></a>Establecimiento de la retención de datos mediante un script de PowerShell

El siguiente script también se puede usar para cambiar la retención. Copie este script para guardarlo como `Set-ApplicationInsightsRetention.ps1`.

```PS
Param(
    [Parameter(Mandatory = $True)]
    [string]$SubscriptionId,

    [Parameter(Mandatory = $True)]
    [string]$ResourceGroupName,

    [Parameter(Mandatory = $True)]
    [string]$Name,

    [Parameter(Mandatory = $True)]
    [string]$RetentionInDays
)
$ErrorActionPreference = 'Stop'
if (-not (Get-Module Az.Accounts)) {
    Import-Module Az.Accounts
}
$azProfile = [Microsoft.Azure.Commands.Common.Authentication.Abstractions.AzureRmProfileProvider]::Instance.Profile
if (-not $azProfile.Accounts.Count) {
    Write-Error "Ensure you have logged in before calling this function."    
}
$currentAzureContext = Get-AzContext
$profileClient = New-Object Microsoft.Azure.Commands.ResourceManager.Common.RMProfileClient($azProfile)
$token = $profileClient.AcquireAccessToken($currentAzureContext.Tenant.TenantId)
$UserToken = $token.AccessToken
$RequestUri = "https://management.azure.com/subscriptions/$($SubscriptionId)/resourceGroups/$($ResourceGroupName)/providers/Microsoft.Insights/components/$($Name)?api-version=2015-05-01"
$Headers = @{
    "Authorization"         = "Bearer $UserToken"
    "x-ms-client-tenant-id" = $currentAzureContext.Tenant.TenantId
}
## Get Component object via ARM
$GetResponse = Invoke-RestMethod -Method "GET" -Uri $RequestUri -Headers $Headers 

## Update RetentionInDays property
if($($GetResponse.properties | Get-Member "RetentionInDays"))
{
    $GetResponse.properties.RetentionInDays = $RetentionInDays
}
else
{
    $GetResponse.properties | Add-Member -Type NoteProperty -Name "RetentionInDays" -Value $RetentionInDays
}
## Upsert Component object via ARM
$PutResponse = Invoke-RestMethod -Method "PUT" -Uri "$($RequestUri)" -Headers $Headers -Body $($GetResponse | ConvertTo-Json) -ContentType "application/json"
$PutResponse
```

Después, este script se puede utilizar para:

```PS
Set-ApplicationInsightsRetention `
        [-SubscriptionId] <String> `
        [-ResourceGroupName] <String> `
        [-Name] <String> `
        [-RetentionInDays <Int>]
```

## <a name="set-the-daily-cap"></a>Establecimiento del límite diario

Para obtener las propiedades de límite diario, use el cmdlet [Set-AzApplicationInsightsPricingPlan](/powershell/module/az.applicationinsights/set-azapplicationinsightspricingplan): 

```PS
Set-AzApplicationInsightsDailyCap -ResourceGroupName <resource group> -Name <resource name> | Format-List
```

Para establecer las propiedades de límite diario, use el mismo cmdlet. Por ejemplo, para establecer el límite en 300 GB/día:

```PS
Set-AzApplicationInsightsDailyCap -ResourceGroupName <resource group> -Name <resource name> -DailyCapGB 300
```

También puede usar [ARMClient](https://github.com/projectkudu/ARMClient) para obtener y establecer parámetros de límite diarios.  Para obtener los valores actuales, use:

```PS
armclient GET /subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/MyResourceGroupName/providers/microsoft.insights/components/MyResourceName/CurrentBillingFeatures?api-version=2018-05-01-preview
```

## <a name="set-the-daily-cap-reset-time"></a>Establecimiento del tiempo de restablecimiento del límite diario

Para establecer el tiempo de restablecimiento del límite diario, puede usar [ARMClient](https://github.com/projectkudu/ARMClient). Este es un ejemplo de uso de `ARMClient`, para establecer el tiempo de restablecimiento en una hora nueva (en este ejemplo 12:00 UTC):

```PS
armclient PUT /subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/MyResourceGroupName/providers/microsoft.insights/components/MyResourceName/CurrentBillingFeatures?api-version=2018-05-01-preview "{'CurrentBillingFeatures':['Basic'],'DataVolumeCap':{'ResetTime':12}}"
```

<a id="price"></a>
## <a name="set-the-pricing-plan"></a>Establecimiento del plan de precios 

Para obtener el plan de precios actual, use el cmdlet [Set-AzApplicationInsightsPricingPlan](/powershell/module/az.applicationinsights/set-azapplicationinsightspricingplan):

```PS
Set-AzApplicationInsightsPricingPlan -ResourceGroupName <resource group> -Name <resource name> | Format-List
```

Para establecer el plan de precios, use el mismo cmdlet con la especificación de `-PricingPlan`:  

```PS
Set-AzApplicationInsightsPricingPlan -ResourceGroupName <resource group> -Name <resource name> -PricingPlan Basic
```

También puede establecer el plan de precios de un recurso de Application Insights existente mediante la plantilla de Resource Manager anterior, omitiendo el recurso "microsoft.insights/components" y el nodo `dependsOn` del recurso de facturación. Por ejemplo, para establecerlo en el plan Por GB (antes plan Básico), ejecute:

```PS
        New-AzResourceGroupDeployment -ResourceGroupName "<resource group>" `
               -TemplateFile .\template1.json `
               -priceCode 1 `
               -appName myApp
```

`priceCode` se define como:

|priceCode|plan|
|---|---|
|1|Por GB (anteriormente denominado plan Básico)|
|2|Por nodo (anteriormente denominado plan Enterprise)|

Por último, puede usar [ARMClient](https://github.com/projectkudu/ARMClient) para obtener y establecer planes de precios y parámetros de límite diarios.  Para obtener los valores actuales, use:

```PS
armclient GET /subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/MyResourceGroupName/providers/microsoft.insights/components/MyResourceName/CurrentBillingFeatures?api-version=2018-05-01-preview
```

Y puede establecer todos estos parámetros mediante:

```PS
armclient PUT /subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/MyResourceGroupName/providers/microsoft.insights/components/MyResourceName/CurrentBillingFeatures?api-version=2018-05-01-preview
"{'CurrentBillingFeatures':['Basic'],'DataVolumeCap':{'Cap':200,'ResetTime':12,'StopSendNotificationWhenHitCap':true,'WarningThreshold':90,'StopSendNotificationWhenHitThreshold':true}}"
```

De este modo, establecerá el límite diario en 200 GB/día, configurará el tiempo de restablecimiento del límite diario en 12:00 UTC, enviará mensajes de correo electrónico cuando se alcance el límite y se cumpla el nivel de advertencia, y establecerá el umbral de advertencia en el 90 % del límite.  

## <a name="add-a-metric-alert"></a>Agregar una alerta de métrica

Para automatizar la creación de las alertas de métricas, consulte el artículo sobre la [plantilla de alertas de métricas](../platform/alerts-metric-create-templates.md#template-for-a-simple-static-threshold-metric-alert).


## <a name="add-an-availability-test"></a>Agregar una prueba de disponibilidad

Para automatizar las pruebas de disponibilidad, consulte el artículo sobre la [plantilla de alertas de métricas](../platform/alerts-metric-create-templates.md#template-for-an-availability-test-along-with-a-metric-alert).

## <a name="add-more-resources"></a>Agregar más recursos

Para automatizar la creación de otros recursos de cualquier variante, cree un ejemplo manualmente y luego copie y parametrice su código en [Azure Resource Manager](https://resources.azure.com/). 

1. Abra el [Administrador de recursos de Azure](https://resources.azure.com/). Desplácese hacia abajo por `subscriptions/resourceGroups/<your resource group>/providers/Microsoft.Insights/components` hasta el recurso de la aplicación. 
   
    ![Navegación en el Explorador de recursos de Azure](./media/powershell/01.png)
   
    *Componentes* son los recursos básicos de Application Insights para mostrar aplicaciones. Existen recursos distintos para las reglas de alerta asociadas y las pruebas web de disponibilidad.
2. Copie el código JSON del componente en el lugar adecuado en `template1.json`.
3. Elimine estas propiedades:
   
   * `id`
   * `InstrumentationKey`
   * `CreationDate`
   * `TenantId`
4. Abra las secciones `webtests` y `alertrules` y copie el código JSON de los elementos individuales en la plantilla. No copie de los nodos `webtests` o `alertrules`: vaya a los elementos que hay debajo de ellos.
   
    Cada prueba web tiene una regla de alerta asociada, por lo que tiene que copiar las dos.
   
    También puede incluir alertas en las métricas. [Nombres de métricas](powershell-alerts.md#metric-names).
5. Inserte esta línea en cada recurso:
   
    `"apiVersion": "2015-05-01",`

### <a name="parameterize-the-template"></a>Parametrización de la plantilla
Ahora, debe reemplazar los nombres específicos por parámetros. Para [parametrizar una plantilla](../../azure-resource-manager/templates/template-syntax.md), escriba expresiones mediante un [conjunto de funciones auxiliares](../../azure-resource-manager/templates/template-functions.md). 

No se puede parametrizar solo una parte de una cadena, así que use `concat()` para compilar las cadenas.

Éstos son ejemplos de sustituciones que querrá realizar. Hay varias repeticiones de cada sustitución. Y puede que tenga otras en la plantilla. En estos ejemplos se usan los parámetros y las variables que se han definido en la parte superior de la plantilla.

| find | reemplazar por |
| --- | --- |
| `"hidden-link:/subscriptions/.../../components/MyAppName"` |`"[concat('hidden-link:',`<br/>`resourceId('microsoft.insights/components',` <br/> `parameters('appName')))]"` |
| `"/subscriptions/.../../alertrules/myAlertName-myAppName-subsId",` |`"[resourceId('Microsoft.Insights/alertrules', variables('alertRuleName'))]",` |
| `"/subscriptions/.../../webtests/myTestName-myAppName",` |`"[resourceId('Microsoft.Insights/webtests', parameters('webTestName'))]",` |
| `"myWebTest-myAppName"` |`"[variables(testName)]"'` |
| `"myTestName-myAppName-subsId"` |`"[variables('alertRuleName')]"` |
| `"myAppName"` |`"[parameters('appName')]"` |
| `"myappname"` (minúscula) |`"[toLower(parameters('appName'))]"` |
| `"<WebTest Name=\"myWebTest\" ...`<br/>`Url=\"http://fabrikam.com/home\" ...>"` |`[concat('<WebTest Name=\"',` <br/> `parameters('webTestName'),` <br/> `'\" ... Url=\"', parameters('Url'),` <br/> `'\"...>')]"`|

### <a name="set-dependencies-between-the-resources"></a>Establecimiento de dependencias entre los recursos
Azure debe instalar los recursos en un orden estricto. Para tener la seguridad de que una instalación finaliza antes de que comience la siguiente, agregue líneas de dependencia:

* En el recurso de la prueba de disponibilidad:
  
    `"dependsOn": ["[resourceId('Microsoft.Insights/components', parameters('appName'))]"],`
* En el recurso de la alerta para una prueba de disponibilidad:
  
    `"dependsOn": ["[resourceId('Microsoft.Insights/webtests', variables('testName'))]"],`



## <a name="next-steps"></a>Pasos siguientes
Otros artículos de automatización:

* [Script de PowerShell para crear un recurso de Application Insights](./create-new-resource.md#creating-a-resource-automatically) : método rápido sin necesidad de plantilla.
* [Uso de PowerShell para configurar alertas en Application Insights](powershell-alerts.md)
* [Creación de pruebas web](https://azure.microsoft.com/blog/creating-a-web-test-alert-programmatically-with-application-insights/)
* [Envío de Azure Diagnostics a Application Insights](powershell-azure-diagnostics.md)
* [Creación de anotaciones de versión](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/API/CreateReleaseAnnotation.ps1)

