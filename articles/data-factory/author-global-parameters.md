---
title: Parámetros globales
description: Establecimiento de los parámetros globales para cada uno de los entornos de Azure Data Factory
services: data-factory
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
author: djpmsft
ms.author: daperlov
ms.date: 08/05/2020
ms.openlocfilehash: 052f502ed27db9ade0fd2916f91d6922c52a5a98
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87854168"
---
# <a name="global-parameters-in-azure-data-factory"></a>Parámetros globales en Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Los parámetros globales son constantes en una factoría de datos que una canalización puede consumir en cualquier expresión. Resultan útiles cuando se tienen varias canalizaciones con nombres y valores de parámetros idénticos. Al promover una factoría de datos mediante el proceso de integración e implementación continuas (CI/CD), puede invalidar estos parámetros en cada entorno. 

## <a name="creating-global-parameters"></a>Creación de parámetros globales

Para crear un parámetro global, vaya a la pestaña *Parámetros globales* de la sección *Administrar*. Seleccione **Nuevo** para abrir el panel de navegación lateral de creación.

![Creación de parámetros globales](media/author-global-parameters/create-global-parameter-1.png)

En el panel de navegación lateral, escriba un nombre, seleccione un tipo de datos y especifique el valor del parámetro.

![Creación de parámetros globales](media/author-global-parameters/create-global-parameter-2.png)

Una vez creado un parámetro global, puede editarlo con un clic en el nombre del parámetro. Para modificar varios parámetros a la vez, seleccione **Editar todo**.

![Creación de parámetros globales](media/author-global-parameters/create-global-parameter-3.png)

## <a name="using-global-parameters-in-a-pipeline"></a>Uso de parámetros globales en una canalización

Los parámetros globales se pueden usar en cualquier [expresión de canalización](control-flow-expression-language-functions.md). Si una canalización hace referencia a otro recurso, como un conjunto de datos o un flujo de datos, puede pasar el valor del parámetro global a través de los parámetros de ese recurso. Se hace referencia a los parámetros globales como `pipeline().globalParameters.<parameterName>`.

![Uso de parámetros globales](media/author-global-parameters/expression-global-parameters.png)

## <a name="global-parameters-in-cicd"></a><a name="cicd"></a> Parámetros globales en CI/CD

Los parámetros globales tienen un proceso de CI/CD único relativo a otras entidades de Azure Data Factory. Al publicar una factoría o exportar una plantilla de Resource Manager con parámetros globales, se crea una carpeta llamada *globalParameters* con un archivo denominado *nombre-de-su-factoría.json*. Este archivo es un objeto JSON que contiene cada tipo y valor de parámetro global en la factoría publicada.

![Publicación de parámetros globales](media/author-global-parameters/global-parameters-adf-publish.png)

Si va a realizar la implementación en un entorno nuevo como TEST o PROD, se recomienda crear una copia de este archivo de parámetros globales y sobrescribir los valores específicos del entorno. Cuando vuelva a realizar la publicación, el archivo de parámetros globales original se sobrescribirá, pero la copia del otro entorno no se modificará.

Por ejemplo, si tiene una factoría denominada "ADF-DEV" y un parámetro global de tipo cadena denominado "environment" con un valor "dev", cuando publique un archivo denominado *ADF-DEV_GlobalParameters.json*, se generará. Si realiza la implementación en una factoría de prueba denominada "ADF_TEST", cree una copia del archivo JSON (por ejemplo, con el nombre ADF-TEST_GlobalParameters.json) y reemplace los valores de parámetro por los valores específicos del entorno. El parámetro "environment" ahora puede tener un valor "test". 

![Implementación de parámetros globales](media/author-global-parameters/powershell-task.png)

Use el script de PowerShell siguiente para promover los parámetros globales a otros entornos. Agregue una tarea de DevOps de Azure PowerShell antes de la implementación de plantillas de Resource Manager. En la tarea DevOps, debe especificar la ubicación del archivo de parámetros nuevo, el grupo de recursos de destino y la factoría de datos de destino.

> [!NOTE]
> Para implementar parámetros globales mediante PowerShell, debe usar al menos la versión 4.4.0 del módulo AZ.

```powershell
param
(
    [parameter(Mandatory = $true)] [String] $globalParametersFilePath,
    [parameter(Mandatory = $true)] [String] $resourceGroupName,
    [parameter(Mandatory = $true)] [String] $dataFactoryName
)

Import-Module Az.DataFactory

$newGlobalParameters = New-Object 'system.collections.generic.dictionary[string,Microsoft.Azure.Management.DataFactory.Models.GlobalParameterSpecification]'

Write-Host "Getting global parameters JSON from: " $globalParametersFilePath
$globalParametersJson = Get-Content $globalParametersFilePath

Write-Host "Parsing JSON..."
$globalParametersObject = [Newtonsoft.Json.Linq.JObject]::Parse($globalParametersJson)

foreach ($gp in $globalParametersObject.GetEnumerator()) {
    Write-Host "Adding global parameter:" $gp.Key
    $globalParameterValue = $gp.Value.ToObject([Microsoft.Azure.Management.DataFactory.Models.GlobalParameterSpecification])
    $newGlobalParameters.Add($gp.Key, $globalParameterValue)
}

$dataFactory = Get-AzDataFactoryV2 -ResourceGroupName $resourceGroupName -Name $dataFactoryName
$dataFactory.GlobalParameters = $newGlobalParameters

Write-Host "Updating" $newGlobalParameters.Count "global parameters."

Set-AzDataFactoryV2 -InputObject $dataFactory -Force
```

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre el [proceso de integración e implementación continuas](continuous-integration-deployment.md) de Azure Data Factory.
* Más información sobre cómo usar el [lenguaje de expresiones del flujo de control](control-flow-expression-language-functions.md).