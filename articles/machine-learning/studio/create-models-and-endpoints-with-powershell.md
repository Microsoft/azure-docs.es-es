---
title: 'ML Studio (clásico): Creación de varios modelos y puntos de conexión - Azure'
description: Use PowerShell para crear varios modelos de Machine Learning y puntos de conexión de servicio web con el mismo algoritmo pero con conjuntos de datos de entrenamiento distintos.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: seodec18
ms.date: 04/04/2017
ms.openlocfilehash: 744b8e3ecf067e6f5986dde4fbf8abbe3a4c6224
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87431978"
---
# <a name="create-multiple-web-service-endpoints-from-one-experiment-with-ml-studio-classic-and-powershell"></a>Creación de varios puntos de conexión de servicio web a partir de un experimento con ML Studio (clásico) y PowerShell

**SE APLICA A:**  ![yes](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (clásico)   ![no](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../compare-azure-ml-to-studio-classic.md)

El siguiente es un problema de aprendizaje automático habitual: quiere crear muchos modelos que tienen el mismo flujo de trabajo de entrenamiento y utilizan el mismo algoritmo. Pero desea que tengan conjuntos de datos de entrenamiento distintos como entrada. Este artículo muestra cómo hacer esto a escala en Azure Machine Learning Studio (clásico) simplemente con un solo experimento.

Por ejemplo, digamos que posee una empresa de franquicias de alquiler de bicicletas global. Desea crear un modelo de regresión para predecir la demanda de alquiler basada en datos históricos. Dispone de mil ubicaciones de alquiler en todo el mundo y ha recopilado un conjunto de datos para cada ubicación. Incluyen características importantes como la fecha, la hora e información meteorológica y sobre el tráfico que son específicas de cada ubicación.

Puede entrenar el modelo una vez usando una versión combinada de todos los conjuntos de datos en todas las ubicaciones. Sin embargo, cada una de las ubicaciones tiene un entorno único. Por tanto, un mejor enfoque sería entrenar el modelo de regresión por separado mediante el conjunto de datos de cada ubicación. De este modo, cada modelo entrenado podría tener en cuenta los diferentes tamaños de tienda, el volumen, la geografía, la población, el entorno de tráfico preparado para bicicletas, etc.

Ese puede que sea el mejor enfoque, pero no desea crear 1000 experimentos de entrenamiento en Azure Machine Learning Studio (clásico), y cada uno de ellos representa una ubicación única. Además de ser una tarea abrumadora, también parece ineficaz, ya que cada experimento tendría exactamente los mismos componentes, excepto el conjunto de datos de entrenamiento.

Por suerte, puede lograrlo con la [API para volver a entrenar de Azure Machine Learning Studio (clásico)](/azure/machine-learning/studio/retrain-machine-learning-model) y automatizando la tarea con [PowerShell de Azure Machine Learning Studio (clásico)](powershell-module.md).

> [!NOTE]
> Para que nuestro ejemplo se ejecute más rápido, reduzca el número de ubicaciones de mil a diez. Pero se aplican los mismos principios y procedimientos a 1000 ubicaciones. Sin embargo, si desea entrenar mil conjuntos de datos, puede ejecutar los siguientes scripts de PowerShell en paralelo. Cómo hacerlo queda fuera del ámbito de este artículo, pero puede encontrar ejemplos de subprocesamiento múltiple de PowerShell en Internet.  
> 
> 

## <a name="set-up-the-training-experiment"></a>Configuración del experimento de entrenamiento
Use el [experimento de entrenamiento](https://gallery.azure.ai/Experiment/Bike-Rental-Training-Experiment-1) de ejemplo que se encuentra en la [Galería de Cortana Intelligence](https://gallery.azure.ai). Abra este experimento en su área de trabajo [Azure Machine Learning Studio (clásico)](https://studio.azureml.net).

> [!NOTE]
> Para seguir este ejemplo, puede que le interese usar un área de trabajo estándar en lugar de un área de trabajo gratis. Cree un punto de conexión para cada cliente (para un total de diez puntos de conexión) y que requiere un área de trabajo estándar, ya que un área de trabajo gratis se limita a tres puntos de conexión.
> 
> 

El experimento usa un módulo **Importar datos** para importar el conjunto de datos de entrenamiento *customer001.csv* desde una cuenta de almacenamiento de Azure. Supongamos que ha recopilado los conjuntos de datos de entrenamiento de todas las ubicaciones de alquiler de bicicletas y los ha almacenado en la misma ubicación de Blob Storage con nombres de archivo que van desde *rentalloc001.csv* a *rentalloc10.csv*.

![El módulo de lector importa datos de un blob de Azure](./media/create-models-and-endpoints-with-powershell/reader-module.png)

Tenga en cuenta que se ha agregado el módulo **Web Service Output** (Resultados de servicio web) al módulo **Train Model** (Entrenar modelo).
Cuando este experimento se implementa como un servicio web, el punto de conexión asociado a esa salida devuelve el modelo entrenado con el formato de un archivo .ilearner.

Observe también que configura un parámetro de servicio web que define la dirección URL que el módulo **Importar datos** utiliza. Esto le permite utilizar el parámetro para especificar conjuntos de datos de entrenamiento individuales para entrenar el modelo para cada ubicación.
Hay otras maneras de hacer esto. Puede usar una consulta SQL con un parámetro de servicio web para obtener datos de una base de datos de Azure SQL Database. O bien, puede usar el módulo **Web Service Input** (Entrada de servicio web) para pasar un conjunto de datos al servicio web.

![Salidas del módulo de un módulo entrenado a un módulo de salida de servicio web](./media/create-models-and-endpoints-with-powershell/web-service-output.png)

Ahora, vamos a ejecutar este experimento de entrenamiento utilizando el valor predeterminado *rental001.csv* como el conjunto de datos de entrenamiento. Si ve los resultados del módulo **Evaluar**, después de hacer clic en los resultados y seleccionar **Visualizar**, verá que obtiene un rendimiento aceptable de *AUC* = 0,91. En este momento, está listo para implementar un servicio web fuera de este experimento de entrenamiento.

## <a name="deploy-the-training-and-scoring-web-services"></a>Implementación del entrenamiento y puntuación de servicios web
Para implementar el servicio web de entrenamiento, haga clic en el botón **Set Up Web Service** (Configurar servicio web) situado debajo del lienzo del experimento y seleccione **Deploy Web Service** (Implementar servicio web). Asigne el nombre “Bike Rental Training” al servicio web.

Ahora debe implementar el servicio web de puntuación.
Para ello, haga clic en **Set Up Web Service** (Configurar servicio web) bajo el lienzo y seleccione **Predictive Web Service** (Servicio web predictivo). Esto crear un experimento de puntuación.
Debe realizar algunos ajustes menores para que funcione como un servicio web. Quite la columna de etiqueta "cnt" de los datos de entrada y limite las salida a solo el identificador de instancia y el valor de predicción correspondiente.

Para ahorrarse ese trabajo, puede abrir el [experimento predictivo](https://gallery.azure.ai/Experiment/Bike-Rental-Predicative-Experiment-1) en la galería que ya se ha preparado.

Para implementar el servicio web, ejecute el experimento predictivo y luego haga clic en el botón **Deploy Web Service** (Implementar servicio web) bajo el lienzo. Asigne el nombre “Bike Rental Scoring” al servicio web de puntuación.

## <a name="create-10-identical-web-service-endpoints-with-powershell"></a>Creación de 10 puntos de conexión de servicio web idénticos con PowerShell
Este servicio web incluye un punto de conexión predeterminado. Pero el interés no está en el punto de conexión predeterminado, ya que no se puede actualizar. Lo que necesita hacer es crear diez puntos de conexión adicionales, uno para cada ubicación. Puede hacer esto con PowerShell.

En primer lugar, configure el entorno de PowerShell:

```powershell
Import-Module .\AzureMLPS.dll
# Assume the default configuration file exists and is properly set to point to the valid Workspace.
$scoringSvc = Get-AmlWebService | where Name -eq 'Bike Rental Scoring'
$trainingSvc = Get-AmlWebService | where Name -eq 'Bike Rental Training'
```

Después, ejecute el siguiente comando de PowerShell:

```powershell
# Create 10 endpoints on the scoring web service.
For ($i = 1; $i -le 10; $i++){
    $seq = $i.ToString().PadLeft(3, '0');
    $endpointName = 'rentalloc' + $seq;
    Write-Host ('adding endpoint ' + $endpointName + '...')
    Add-AmlWebServiceEndpoint -WebServiceId $scoringSvc.Id -EndpointName $endpointName -Description $endpointName     
}
```

Ahora ha creado diez puntos de conexión y todos ellos contienen el mismo modelo entrenado en *customer001.csv*. Puede verlos en Azure Portal.

![Ver la lista de modelos entrenados en el portal](./media/create-models-and-endpoints-with-powershell/created-endpoints.png)

## <a name="update-the-endpoints-to-use-separate-training-datasets-using-powershell"></a>Actualización de los puntos de conexión para usar conjuntos de datos de entrenamiento independientes mediante PowerShell
El siguiente paso consiste en actualizar los puntos de conexión con modelos entrenados excepcionalmente en datos individuales de cada cliente. Pero primero necesita generar estos modelos desde el servicio web **Bike Rental Training** (Entrenamiento para alquiler de bicicletas). Volvamos al servicio web **Bike Rental Training** (Entrenamiento para alquiler de bicicletas). Necesita llamar a su punto de conexión BES diez veces con diez conjuntos de datos de entrenamiento distintos para generar diez modelos diferentes. Utilice el cmdlet de PowerShell **InovkeAmlWebServiceBESEndpoint** para hacerlo.

También debe proporcionar credenciales para su cuenta de almacenamiento de blobs en `$configContent`. Concretamente, en los campos `AccountName`, `AccountKey` y `RelativeLocation`. `AccountName` puede ser uno de los nombres de cuenta, como se muestra en **Azure Portal** (pestaña *Almacenamiento*). Cuando haga clic en una cuenta de almacenamiento, su `AccountKey` puede encontrarse presionando el botón **Administrar claves de acceso** , situado en la parte inferior, y copiando la *clave de acceso principal*. El campo `RelativeLocation` es la ruta de acceso relativa al almacenamiento donde se almacenará un nuevo modelo. Por ejemplo, la ruta de acceso `hai/retrain/bike_rental/` en el script siguiente señala a un contenedor denominado `hai` y `/retrain/bike_rental/` son subcarpetas. Actualmente, no puede crear subcarpetas a través del portal de interfaz de usuario, pero hay [varios exploradores de Azure Storage](../../storage/common/storage-explorers.md) que le permiten hacerlo. Se recomienda crear un contenedor en el almacenamiento para almacenar los nuevos modelos entrenados (archivos .ilearner) de esta forma: en la página de almacenamiento, haga clic en el botón **Agregar** en la parte inferior y asígnele el nombre `retrain`. En resumen, los cambios necesarios para el siguiente script se refieren a `AccountName`, `AccountKey` y `RelativeLocation` (:`"retrain/model' + $seq + '.ilearner"`).

```powershell
# Invoke the retraining API 10 times
# This is the default (and the only) endpoint on the training web service
$trainingSvcEp = (Get-AmlWebServiceEndpoint -WebServiceId $trainingSvc.Id)[0];
$submitJobRequestUrl = $trainingSvcEp.ApiLocation + '/jobs?api-version=2.0';
$apiKey = $trainingSvcEp.PrimaryKey;
For ($i = 1; $i -le 10; $i++){
    $seq = $i.ToString().PadLeft(3, '0');
    $inputFileName = 'https://bostonmtc.blob.core.windows.net/hai/retrain/bike_rental/BikeRental' + $seq + '.csv';
    $configContent = '{ "GlobalParameters": { "URI": "' + $inputFileName + '" }, "Outputs": { "output1": { "ConnectionString": "DefaultEndpointsProtocol=https;AccountName=<myaccount>;AccountKey=<mykey>", "RelativeLocation": "hai/retrain/bike_rental/model' + $seq + '.ilearner" } } }';
    Write-Host ('training regression model on ' + $inputFileName + ' for rental location ' + $seq + '...');
    Invoke-AmlWebServiceBESEndpoint -JobConfigString $configContent -SubmitJobRequestUrl $submitJobRequestUrl -ApiKey $apiKey
}
```

> [!NOTE]
> El punto de conexión BES es el único modo admitido para esta operación. RRS no se puede usar para generar modelos entrenados.
> 
> 

Como puede ver más arriba, en lugar de construir diez archivos json de trabajo BES diferentes, crea dinámicamente la cadena de configuración. A continuación, la pasa al parámetro *jobConfigString* del cmdlet **InvokeAmlWebServceBESEndpoint**. Realmente no hay ninguna necesidad de mantener una copia en disco.

Si todo va bien, después de un tiempo verá diez archivos .iLearner (de *model001.ilearner* a *model010.ilearner*) en la cuenta de Azure Storage. Ahora ya está listo para actualizar los diez puntos de conexión del servicio web de puntuación con estos modelos mediante el cmdlet de PowerShell **Patch-AmlWebServiceEndpoint**. Recuerde una vez más que solo puede revisar los puntos de conexión no predeterminados mediante programación creados anteriormente.

```powershell
# Patch the 10 endpoints with respective .ilearner models
$baseLoc = 'http://bostonmtc.blob.core.windows.net/'
$sasToken = '<my_blob_sas_token>'
For ($i = 1; $i -le 10; $i++){
    $seq = $i.ToString().PadLeft(3, '0');
    $endpointName = 'rentalloc' + $seq;
    $relativeLoc = 'hai/retrain/bike_rental/model' + $seq + '.ilearner';
    Write-Host ('Patching endpoint ' + $endpointName + '...');
    Patch-AmlWebServiceEndpoint -WebServiceId $scoringSvc.Id -EndpointName $endpointName -ResourceName 'Bike Rental [trained model]' -BaseLocation $baseLoc -RelativeLocation $relativeLoc -SasBlobToken $sasToken
}
```

Esto se debe ejecutar bastante rápido. Cuando finaliza la ejecución, habrá creado correctamente diez puntos de conexión de servicio web de predicción. Cada uno de ellos contendrá un modelo entrenado de forma única en el conjunto de datos específico a una ubicación de alquiler, todo ello desde un experimento de entrenamiento único. Para comprobar esto, puede intentar llamar a estos puntos de conexión mediante el cmdlet **InvokeAmlWebServiceRRSEndpoint**, proporcionándoles los mismos datos de entrada. Debería ver resultados de predicción diferentes, ya que los modelos se entrenan con conjuntos de entrenamiento diferentes.

## <a name="full-powershell-script"></a>Script de PowerShell completo
Este es el listado del código fuente completo:

```powershell
Import-Module .\AzureMLPS.dll
# Assume the default configuration file exists and properly set to point to the valid workspace.
$scoringSvc = Get-AmlWebService | where Name -eq 'Bike Rental Scoring'
$trainingSvc = Get-AmlWebService | where Name -eq 'Bike Rental Training'

# Create 10 endpoints on the scoring web service
For ($i = 1; $i -le 10; $i++){
    $seq = $i.ToString().PadLeft(3, '0');
    $endpointName = 'rentalloc' + $seq;
    Write-Host ('adding endpoint ' + $endpontName + '...')
    Add-AmlWebServiceEndpoint -WebServiceId $scoringSvc.Id -EndpointName $endpointName -Description $endpointName     
}

# Invoke the retraining API 10 times to produce 10 regression models in .ilearner format
$trainingSvcEp = (Get-AmlWebServiceEndpoint -WebServiceId $trainingSvc.Id)[0];
$submitJobRequestUrl = $trainingSvcEp.ApiLocation + '/jobs?api-version=2.0';
$apiKey = $trainingSvcEp.PrimaryKey;
For ($i = 1; $i -le 10; $i++){
    $seq = $i.ToString().PadLeft(3, '0');
    $inputFileName = 'https://bostonmtc.blob.core.windows.net/hai/retrain/bike_rental/BikeRental' + $seq + '.csv';
    $configContent = '{ "GlobalParameters": { "URI": "' + $inputFileName + '" }, "Outputs": { "output1": { "ConnectionString": "DefaultEndpointsProtocol=https;AccountName=<myaccount>;AccountKey=<mykey>", "RelativeLocation": "hai/retrain/bike_rental/model' + $seq + '.ilearner" } } }';
    Write-Host ('training regression model on ' + $inputFileName + ' for rental location ' + $seq + '...');
    Invoke-AmlWebServiceBESEndpoint -JobConfigString $configContent -SubmitJobRequestUrl $submitJobRequestUrl -ApiKey $apiKey
}

# Patch the 10 endpoints with respective .ilearner models
$baseLoc = 'http://bostonmtc.blob.core.windows.net/'
$sasToken = '?test'
For ($i = 1; $i -le 10; $i++){
    $seq = $i.ToString().PadLeft(3, '0');
    $endpointName = 'rentalloc' + $seq;
    $relativeLoc = 'hai/retrain/bike_rental/model' + $seq + '.ilearner';
    Write-Host ('Patching endpoint ' + $endpointName + '...');
    Patch-AmlWebServiceEndpoint -WebServiceId $scoringSvc.Id -EndpointName $endpointName -ResourceName 'Bike Rental [trained model]' -BaseLocation $baseLoc -RelativeLocation $relativeLoc -SasBlobToken $sasToken
}
```
