---
title: Supervisión y administración de los trabajos de Azure Stream Analytics con PowerShell
description: En este artículo se describe cómo usar Azure PowerShell y cmdlets para supervisar y administrar trabajos de Azure Stream Analytics.
author: jseb225
ms.author: jeanb
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 03/28/2017
ms.openlocfilehash: 0387b4363a438ac48142d97b2a829faf38f1895d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87036665"
---
# <a name="monitor-and-manage-stream-analytics-jobs-with-azure-powershell-cmdlets"></a>Supervisar y administrar los trabajos de Stream Analytics con los cmdlets de Azure PowerShell
Aprenda a supervisar y administrar los recursos de Stream Analytics con los cmdlets de Azure PowerShell y el scripting de PowerShell que se encargan de ejecutar las tareas básicas de Stream Analytics.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites-for-running-azure-powershell-cmdlets-for-stream-analytics"></a>Requisitos previos para ejecutar cmdlets de Azure PowerShell en Stream Analytics
* Cree un grupo de recursos de Azure en su suscripción. A continuación se muestra un ejemplo de script de Azure PowerShell. Para obtener información sobre Azure PowerShell, consulte [Instalación y configuración de Azure PowerShell](/powershell/azure/).  

Azure PowerShell 0.9.8:  

```powershell
# Log in to your Azure account
Add-AzureAccount
# Select the Azure subscription you want to use to create the resource group if you have more han one subscription on your account.
Select-AzureSubscription -SubscriptionName <subscription name>
# If Stream Analytics has not been registered to the subscription, remove remark symbol below (#)to run the Register-AzureProvider cmdlet to register the provider namespace.
#Register-AzureProvider -Force -ProviderNamespace 'Microsoft.StreamAnalytics'
# Create an Azure resource group
New-AzureResourceGroup -Name <YOUR RESOURCE GROUP NAME> -Location <LOCATION>
```

Azure PowerShell 1.0:

```powershell
# Log in to your Azure account
Connect-AzAccount
# Select the Azure subscription you want to use to create the resource group.
Get-AzSubscription -SubscriptionName "your sub" | Select-AzSubscription
# If Stream Analytics has not been registered to the subscription, remove remark symbol below (#)to run the Register-AzureProvider cmdlet to register the provider namespace.
#Register-AzResourceProvider -Force -ProviderNamespace 'Microsoft.StreamAnalytics'
# Create an Azure resource group
New-AzResourceGroup -Name <YOUR RESOURCE GROUP NAME> -Location <LOCATION>
```


> [!NOTE]
> Los trabajos de Stream Analytics creados mediante programación no tienen la supervisión habilitada de forma predeterminada.  Para habilitar manualmente la supervisión en Azure Portal, vaya a la página Supervisión del trabajo y haga clic en el botón Habilitar. Si prefiere hacerlo mediante programación, siga los pasos que se encuentran en [Azure Stream Analytics - Supervisión de trabajos de Stream Analytics mediante programación](stream-analytics-monitor-jobs.md).
> 
> 

## <a name="azure-powershell-cmdlets-for-stream-analytics"></a>Cmdlets de PowerShell de Azure en Stream Analytics
Se pueden usar los siguientes cmdlets de PowerShell de Azure para supervisar y administrar trabajos de Azure Stream Analytics. Tenga en cuenta que Azure PowerShell tiene versiones diferentes. 
**En los ejemplos que se muestran el primer comando es para Azure PowerShell 0.9.8 y el segundo para Azure PowerShell 1.0.** Los comandos de Azure PowerShell 1.0 siempre incluirán "Az".

### <a name="get-azurestreamanalyticsjob--get-azstreamanalyticsjob"></a>Get-AzureStreamAnalyticsJob | Get-AzStreamAnalyticsJob
Muestra todos los trabajos de Stream Analytics definidos en la suscripción de Azure o en el grupo de recursos especificado, u obtiene información del trabajo sobre un trabajo específico en un grupo de recursos.

**Ejemplo 1**

Azure PowerShell 0.9.8:  

```powershell
Get-AzureStreamAnalyticsJob
```

Azure PowerShell 1.0:  

```powershell
Get-AzStreamAnalyticsJob
```

Este comando de PowerShell devuelve información sobre todos los trabajos de Stream Analytics en la suscripción de Azure.

**Ejemplo 2**

Azure PowerShell 0.9.8:  

```powershell
Get-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US 
```

Azure PowerShell 1.0:  

```powershell
Get-AzStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US 
```

Este comando de PowerShell devuelve información sobre todos los trabajos de Stream Analytics del grupo de recursos StreamAnalytics-Default-Central-US.

**Ejemplo 3**

Azure PowerShell 0.9.8:  

```powershell
Get-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -Name StreamingJob
```

Azure PowerShell 1.0:  

```powershell
Get-AzStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -Name StreamingJob
```

Este comando de PowerShell devuelve información sobre el trabajo StreamingJob de Stream Analytics del grupo de recursos StreamAnalytics-Default-Central-US.

### <a name="get-azurestreamanalyticsinput--get-azstreamanalyticsinput"></a>Get-AzureStreamAnalyticsInput | Get-AzStreamAnalyticsInput
Muestra todas las entradas que se definen en un determinado trabajo de Stream Analytics u obtiene información sobre una entrada concreta.

**Ejemplo 1**

Azure PowerShell 0.9.8:  

```powershell
Get-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob
```

Azure PowerShell 1.0:  

```powershell
Get-AzStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob
```

Este comando de PowerShell devuelve información sobre todas las entradas que se definen en el trabajo StreamingJob.

**Ejemplo 2**

Azure PowerShell 0.9.8:  

```powershell
Get-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -Name EntryStream
```

Azure PowerShell 1.0:  

```powershell
Get-AzStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -Name EntryStream
```

Este comando de PowerShell devuelve información sobre la entrada denominada EntryStream que se define en el trabajo StreamingJob.

### <a name="get-azurestreamanalyticsoutput--get-azstreamanalyticsoutput"></a>Get-AzureStreamAnalyticsOutput | Get-AzStreamAnalyticsOutput
Muestra todas las salidas que se definen en un determinado trabajo de Stream Analytics u obtiene información sobre una salida concreta.

**Ejemplo 1**

Azure PowerShell 0.9.8:  

```powershell
Get-AzureStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob
```

Azure PowerShell 1.0:  

```powershell
Get-AzStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob
```

Este comando de PowerShell devuelve información sobre las salidas que se definen en el trabajo StreamingJob.

**Ejemplo 2**

Azure PowerShell 0.9.8:  

```powershell
Get-AzureStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -Name Output
```

Azure PowerShell 1.0:  

```powershell
Get-AzStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -Name Output
```

Este comando de PowerShell devuelve información sobre la salida denominada Output que se define en el trabajo StreamingJob.

### <a name="get-azurestreamanalyticsquota--get-azstreamanalyticsquota"></a>Get-AzureStreamAnalyticsQuota | Get-AzStreamAnalyticsQuota
Obtiene información sobre la cuota de unidades de streaming de una región determinada.

**Ejemplo 1**

Azure PowerShell 0.9.8:  

```powershell
Get-AzureStreamAnalyticsQuota -Location "Central US" 
```

Azure PowerShell 1.0:  

```powershell
Get-AzStreamAnalyticsQuota -Location "Central US" 
```

Este comando de PowerShell devuelve información sobre la cuota de unidades de streaming y su uso en la región Centro de EE. UU.

### <a name="get-azurestreamanalyticstransformation--get-azstreamanalyticstransformation"></a>Get-AzureStreamAnalyticsTransformation | Get-AzStreamAnalyticsTransformation
Obtiene información sobre una transformación específica definida en un trabajo de Stream Analytics.

**Ejemplo 1**

Azure PowerShell 0.9.8:  

```powershell
Get-AzureStreamAnalyticsTransformation -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -Name StreamingJob
```

Azure PowerShell 1.0:  

```powershell
Get-AzStreamAnalyticsTransformation -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -Name StreamingJob
```

Este comando de PowerShell devuelve información sobre la transformación denominada StreamingJob en el trabajo StreamingJob.

### <a name="new-azurestreamanalyticsinput--new-azstreamanalyticsinput"></a>New-AzureStreamAnalyticsInput | New-AzStreamAnalyticsInput
Crea otra entrada en un trabajo de Stream Analytics o actualiza una determinada entrada existente.

El nombre de la entrada puede especificarse en el archivo .JSON o en la línea de comandos. Si se especifican ambos, el nombre en la línea de comandos debe coincidir con el que aparece en el archivo.

Si especifica una entrada que ya existe y no especifica el parámetro –Force, el cmdlet le preguntará si desea reemplazar la entrada existente.

Si especifica el parámetro –Force y el nombre de una entrada existente, esta se reemplazará sin confirmación.

Para información detallada sobre la estructura de archivos JSON y el contenido, consulte la sección [Creación de entrada (Azure Stream Analytics)][msdn-rest-api-create-stream-analytics-input] de la [Biblioteca de referencia de API REST de administración de Stream Analytics][stream.analytics.rest.api.reference].

**Ejemplo 1**

Azure PowerShell 0.9.8:  

```powershell
New-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -File "C:\Input.json" 
```

Azure PowerShell 1.0:  

```powershell
New-AzStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -File "C:\Input.json" 
```

Este comando de PowerShell crea una nueva entrada desde el archivo Input.json. Si ya se ha definido una entrada existente con el nombre especificado en el archivo de definición de entrada, el cmdlet le preguntará si quiere reemplazarlo o no.

**Ejemplo 2**

Azure PowerShell 0.9.8:  

```powershell
New-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -File "C:\Input.json" -Name EntryStream
```

Azure PowerShell 1.0:  

```powershell
New-AzStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -File "C:\Input.json" -Name EntryStream
```

Este comando de PowerShell crea una entrada en el trabajo denominada EntryStream. Si ya se ha definido una entrada existente con este nombre, el cmdlet le preguntará si quiere reemplazarlo o no.

**Ejemplo 3**

Azure PowerShell 0.9.8:  

```powershell
New-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -File "C:\Input.json" -Name EntryStream -Force
```

Azure PowerShell 1.0:  

```powershell
New-AzStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -File "C:\Input.json" -Name EntryStream -Force
```

Este comando de PowerShell reemplaza la definición del origen de entrada existente denominado EntryStream por la definición del archivo.

### <a name="new-azurestreamanalyticsjob--new-azstreamanalyticsjob"></a>New-AzureStreamAnalyticsJob | New-AzStreamAnalyticsJob
Crea un trabajo de Stream Analytics en Microsoft Azure o actualiza la definición de un determinado trabajo existente.

El nombre del trabajo puede especificarse en el archivo .JSON o en la línea de comandos. Si se especifican ambos, el nombre en la línea de comandos debe coincidir con el que aparece en el archivo.

Si especifica un nombre de trabajo que ya existe y no especifica el parámetro -Force, el cmdlet le preguntará si quiere reemplazar el trabajo existente.

Si especifica el parámetro -Force y el nombre de un trabajo existente, la definición del trabajo se reemplazará sin confirmación.

Para información detallada sobre la estructura de archivos JSON y el contenido, consulte la sección [Creación de trabajo de Stream Analytics][msdn-rest-api-create-stream-analytics-job] de la [Biblioteca de referencia de API REST de administración de Stream Analytics][stream.analytics.rest.api.reference].

**Ejemplo 1**

Azure PowerShell 0.9.8:  

```powershell
New-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -File "C:\JobDefinition.json" 
```

Azure PowerShell 1.0:  

```powershell
New-AzStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -File "C:\JobDefinition.json" 
```

Este comando de PowerShell crea un nuevo trabajo a partir de la definición de JobDefinition.json. Si ya se ha definido un trabajo existente con el nombre especificado en el archivo de definición de trabajo, el cmdlet le preguntará si quiere reemplazarlo o no.

**Ejemplo 2**

Azure PowerShell 0.9.8:  

```powershell
New-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -File "C:\JobDefinition.json" -Name StreamingJob -Force
```

Azure PowerShell 1.0:  

```powershell
New-AzStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -File "C:\JobDefinition.json" -Name StreamingJob -Force
```

Este comando de PowerShell reemplaza la definición del trabajo de StreamingJob.

### <a name="new-azurestreamanalyticsoutput--new-azstreamanalyticsoutput"></a>New-AzureStreamAnalyticsOutput | New-AzStreamAnalyticsOutput
Crea una salida en un trabajo de Stream Analytics o actualiza una salida existente.  

El nombre de la salida puede especificarse en el archivo .JSON o en la línea de comandos. Si se especifican ambos, el nombre en la línea de comandos debe coincidir con el que aparece en el archivo.

Si especifica una salida que ya existe y no especifica el parámetro -Force, el cmdlet le preguntará si quiere reemplazar la salida existente.

Si especifica el parámetro -Force y el nombre de una salida existente, la salida se reemplazará sin confirmación.

Para información detallada sobre la estructura de archivos JSON y el contenido, consulte la sección [Creación de salida (Azure Stream Analytics)][msdn-rest-api-create-stream-analytics-output] de la [Biblioteca de referencia de API REST de Stream Analytics][stream.analytics.rest.api.reference].

**Ejemplo 1**

Azure PowerShell 0.9.8:  

```powershell
New-AzureStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -File "C:\Output.json" -JobName StreamingJob -Name output
```

Azure PowerShell 1.0:  

```powershell
New-AzStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -File "C:\Output.json" -JobName StreamingJob -Name output
```

Este comando de PowerShell crea una nueva salida denominada "output" en el trabajo StreamingJob. Si ya se ha definido un resultado existente con este nombre, el cmdlet le preguntará si quiere reemplazarlo o no.

**Ejemplo 2**

Azure PowerShell 0.9.8:  

```powershell
New-AzureStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -File "C:\Output.json" -JobName StreamingJob -Name output -Force
```

Azure PowerShell 1.0:  

```powershell
New-AzStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -File "C:\Output.json" -JobName StreamingJob -Name output -Force
```

Este comando de PowerShell reemplaza la definición de "output" en el trabajo StreamingJob.

### <a name="new-azurestreamanalyticstransformation--new-azstreamanalyticstransformation"></a>New-AzureStreamAnalyticsTransformation | New-AzStreamAnalyticsTransformation
Crea una transformación en un trabajo de Stream Analytics o actualiza la transformación existente.

El nombre de la transformación puede especificarse en el archivo .JSON o en la línea de comandos. Si se especifican ambos, el nombre en la línea de comandos debe coincidir con el que aparece en el archivo.

Si especifica una transformación que ya existe y no se especifica el parámetro -Force, el cmdlet le preguntará si quiere reemplazar la transformación existente.

Si especifica el parámetro -Force y el nombre de una transformación existente, la transformación se reemplazará sin confirmación.

Para información detallada sobre la estructura de archivos JSON y el contenido, consulte la sección [Creación de transformación (Azure Stream Analytics)][msdn-rest-api-create-stream-analytics-transformation] de la [Biblioteca de referencia de API REST de administración de Stream Analytics][stream.analytics.rest.api.reference].

**Ejemplo 1**

Azure PowerShell 0.9.8:  

```powershell
New-AzureStreamAnalyticsTransformation -ResourceGroupName StreamAnalytics-Default-Central-US -File "C:\Transformation.json" -JobName StreamingJob -Name StreamingJobTransform
```

Azure PowerShell 1.0:  

```powershell
New-AzStreamAnalyticsTransformation -ResourceGroupName StreamAnalytics-Default-Central-US -File "C:\Transformation.json" -JobName StreamingJob -Name StreamingJobTransform
```

Este comando de PowerShell crea una nueva transformación denominada StreamingJobTransform en el trabajo StreamingJob. Si ya hay definida una transformación existente con este nombre, el cmdlet le preguntará si quiere reemplazarla o no.

**Ejemplo 2**

Azure PowerShell 0.9.8:  

```powershell
New-AzureStreamAnalyticsTransformation -ResourceGroupName StreamAnalytics-Default-Central-US -File "C:\Transformation.json" -JobName StreamingJob -Name StreamingJobTransform -Force
```

Azure PowerShell 1.0:  

```powershell
New-AzStreamAnalyticsTransformation -ResourceGroupName StreamAnalytics-Default-Central-US -File "C:\Transformation.json" -JobName StreamingJob -Name StreamingJobTransform -Force
```

 Este comando de PowerShell reemplaza la definición de StreamingJobTransform en el trabajo StreamingJob.

### <a name="remove-azurestreamanalyticsinput--remove-azstreamanalyticsinput"></a>Remove-AzureStreamAnalyticsInput | Remove-AzStreamAnalyticsInput
Elimina de forma asincrónica una entrada específica desde un trabajo de Stream Analytics en Microsoft Azure.  
Si especifica el parámetro -Force, la entrada se eliminará sin confirmación.

**Ejemplo 1**

Azure PowerShell 0.9.8:  

```powershell
Remove-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -Name EventStream
```

Azure PowerShell 1.0:  

```powershell
Remove-AzStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -Name EventStream
```

Este comando de PowerShell elimina la entrada EventStream en el trabajo StreamingJob.  

### <a name="remove-azurestreamanalyticsjob--remove-azstreamanalyticsjob"></a>Remove-AzureStreamAnalyticsJob | Remove-AzStreamAnalyticsJob
Elimina de forma asincrónica un trabajo específico de Stream Analytics en Microsoft Azure.  
Si especifica el parámetro -Force, el trabajo se eliminará sin confirmación.

**Ejemplo 1**

Azure PowerShell 0.9.8:  

```powershell
Remove-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -Name StreamingJob 
```

Azure PowerShell 1.0:  

```powershell
Remove-AzStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -Name StreamingJob 
```

Este comando de PowerShell elimina el trabajo StreamingJob.  

### <a name="remove-azurestreamanalyticsoutput--remove-azstreamanalyticsoutput"></a>Remove-AzureStreamAnalyticsOutput | Remove-AzStreamAnalyticsOutput
Elimina de forma asincrónica un resultado concreto de un trabajo de Stream Analytics en Microsoft Azure.  
Si especifica el parámetro -Force, la salida se eliminará sin confirmación.

**Ejemplo 1**

Azure PowerShell 0.9.8:  

```powershell
Remove-AzureStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -Name Output
```

Azure PowerShell 1.0:  

```powershell
Remove-AzStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -Name Output
```

Este comando de PowerShell elimina la salida Output en el trabajo StreamingJob.  

### <a name="start-azurestreamanalyticsjob--start-azstreamanalyticsjob"></a>Start-AzureStreamAnalyticsJob | Start-AzStreamAnalyticsJob
Implementa e inicia un trabajo de Stream Analytics de Microsoft Azure de forma asincrónica.

**Ejemplo 1**

Azure PowerShell 0.9.8:  

```powershell
Start-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -Name StreamingJob -OutputStartMode CustomTime -OutputStartTime 2012-12-12T12:12:12Z
```

Azure PowerShell 1.0:  

```powershell
Start-AzStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -Name StreamingJob -OutputStartMode CustomTime -OutputStartTime 2012-12-12T12:12:12Z
```

Este comando de PowerShell inicia el trabajo StreamingJob con una hora de inicio de salida personalizada establecida en el 12 de diciembre de 2012, 12:12:12 UTC.

### <a name="stop-azurestreamanalyticsjob--stop-azstreamanalyticsjob"></a>Stop-AzureStreamAnalyticsJob | Stop-AzStreamAnalyticsJob
Detiene la ejecución de un trabajo de Stream Analytics en Microsoft Azure y desasigna recursos que se usaban de forma asincrónica. La definición del trabajo y los metadatos seguirán estando disponible en su suscripción a través de Azure Portal y de las API de administración, para que el trabajo se pueda editar y reiniciar. No se realizará ningún cobro por un trabajo en estado Detenido.

**Ejemplo 1**

Azure PowerShell 0.9.8:  

```powershell
Stop-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -Name StreamingJob 
```

Azure PowerShell 1.0:  

```powershell
Stop-AzStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -Name StreamingJob 
```

Este comando de PowerShell detiene el trabajo StreamingJob.  

### <a name="test-azurestreamanalyticsinput--test-azstreamanalyticsinput"></a>Test-AzureStreamAnalyticsInput | Test-AzStreamAnalyticsInput
Comprueba la capacidad de Stream Analytics para conectarse a una entrada especificada.

**Ejemplo 1**

Azure PowerShell 0.9.8:  

```powershell
Test-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -Name EntryStream
```

Azure PowerShell 1.0:  

```powershell
Test-AzStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -Name EntryStream
```

Este comando de PowerShell comprueba el estado de conexión de la entrada EntryStream en StreamingJob.  

### <a name="test-azurestreamanalyticsoutput--test-azstreamanalyticsoutput"></a>Test-AzureStreamAnalyticsOutput | Test-AzStreamAnalyticsOutput
Comprueba la capacidad de Stream Analytics para conectarse a una salida especificada.

**Ejemplo 1**

Azure PowerShell 0.9.8:  

```powershell
Test-AzureStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -Name Output
```

Azure PowerShell 1.0:  

```powershell
Test-AzStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -Name Output
```

Este comando de PowerShell comprueba el estado de conexión de la salida Output en StreamingJob.  

## <a name="get-support"></a>Obtención de soporte técnico
Para más ayuda, pruebe nuestra [Página de preguntas y respuestas de Microsoft sobre Azure Stream Analytics](https://docs.microsoft.com/answers/topics/azure-stream-analytics.html). 

## <a name="next-steps"></a>Pasos siguientes
* [Introducción a Azure Stream Analytics](stream-analytics-introduction.md)
* [Introducción al uso de Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Escalación de trabajos de Azure Stream Analytics](stream-analytics-scale-jobs.md)
* [Referencia del lenguaje de consulta de Azure Stream Analytics](https://docs.microsoft.com/stream-analytics-query/stream-analytics-query-language-reference)
* [Referencia de API de REST de administración de Azure Stream Analytics](https://msdn.microsoft.com/library/azure/dn835031.aspx)

[msdn-switch-azuremode]: https://msdn.microsoft.com/library/dn722470.aspx
[powershell-install]: https://azure.microsoft.com/documentation/articles/powershell-install-configure/
[msdn-rest-api-create-stream-analytics-job]: https://msdn.microsoft.com/library/dn834994.aspx
[msdn-rest-api-create-stream-analytics-input]: https://msdn.microsoft.com/library/dn835010.aspx
[msdn-rest-api-create-stream-analytics-output]: https://msdn.microsoft.com/library/dn835015.aspx
[msdn-rest-api-create-stream-analytics-transformation]: https://msdn.microsoft.com/library/dn835007.aspx

[stream.analytics.introduction]: stream-analytics-introduction.md
[stream.analytics.get.started]: stream-analytics-real-time-fraud-detection.md
[stream.analytics.developer.guide]: ../stream-analytics-developer-guide.md
[stream.analytics.scale.jobs]: stream-analytics-scale-jobs.md
[stream.analytics.query.language.reference]: https://go.microsoft.com/fwlink/?LinkID=513299
[stream.analytics.rest.api.reference]: https://go.microsoft.com/fwlink/?LinkId=517301

