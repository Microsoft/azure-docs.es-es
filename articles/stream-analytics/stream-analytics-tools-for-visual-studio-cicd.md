---
title: Integrar continuamente y desarrollar con el paquete NuGet de CI/CD de Azure Stream Analytics
description: En este artículo se describe cómo usar el paquete NuGet de CI/CD de Azure Stream Analytics para configurar una integración continua y el proceso de implementación.
services: stream-analytics
author: su-jie
ms.author: sujie
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 05/15/2019
ms.openlocfilehash: f34139dafffe3d4890f17988114dffdd8b480d2d
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65827310"
---
# <a name="continuously-integrate-and-develop-with-azure-stream-analytics-cicd-nuget-package"></a>Integrar continuamente y desarrollar con el paquete NuGet de CI/CD de Azure Stream Analytics
En este artículo se describe cómo usar el paquete NuGet de CI/CD de Azure Stream Analytics para configurar una integración continua y el proceso de implementación.

Use la versión 2.3.0000.0 o superior de las [herramientas de Stream Analytics para Visual Studio](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-tools-for-visual-studio) para obtener compatibilidad para MSBuild.

Hay disponible un paquete NuGet: [Microsoft.Azure.Stream Analytics.CICD](https://www.nuget.org/packages/Microsoft.Azure.StreamAnalytics.CICD/). Proporciona MSBuild, ejecución local y herramientas de implementación que admiten el proceso de integración e implementación continuas de [proyectos de Stream Analytics para Visual Studio](stream-analytics-vs-tools.md). 
> [!NOTE]
> El paquete NuGet solo se puede usar con la versión 2.3.0000.0 o superior de las herramientas de Stream Analytics para Visual Studio. Si tiene proyectos creados en versiones anteriores de las herramientas de Visual Studio, ábralos con la versión 2.3.0000.0 o superior y guárdelos. A continuación, se habilitarán las nuevas funcionalidades. 

Para más información, consulte [Uso de herramientas de Azure Stream Analytics para Visual Studio](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-tools-for-visual-studio).

## <a name="msbuild"></a>MSBuild
Al igual que con la experiencia estándar de Visual Studio MSBuild, para compilar un proyecto tiene dos opciones. Puede hacer clic con el botón derecho en el proyecto y, después, elegir **Compilar**. También puede usar **MSBuild** en el paquete de NuGet desde la línea de comandos.
```
./build/msbuild /t:build [Your Project Full Path] /p:CompilerTaskAssemblyFile=Microsoft.WindowsAzure.StreamAnalytics.Common.CompileService.dll  /p:ASATargetsFilePath="[NuGet Package Local Path]\build\StreamAnalytics.targets"

```

Cuando un proyecto de Stream Analytics para Visual Studio se compila correctamente, genera los dos siguientes archivos de plantilla de Azure Resource Manager en la carpeta **bin/[Debug/Retail]/Deploy**: 

*  Archivo de plantilla de Resource Manager

       [ProjectName].JobTemplate.json 

*  Archivo de parámetros de Resource Manager

       [ProjectName].JobTemplate.parameters.json   

Los parámetros predeterminados en el archivo parameters.json provienen de la configuración del proyecto de Visual Studio. Si quiere implementar en otro entorno, reemplace los parámetros según corresponda.

> [!NOTE]
> En todas las credenciales, los valores predeterminados están establecidos en null. Es **imperativo** establecer los valores antes de realizar la implementación en la nube.

```json
"Input_EntryStream_sharedAccessPolicyKey": {
      "value": null
    },
```
Obtenga más información acerca de cómo [implementar con un archivo de plantilla de Resource Manager y Azure PowerShell](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-deploy). Obtenga más información acerca de cómo [utilizar un objeto como un parámetro en una plantilla de Resource Manager](https://docs.microsoft.com/azure/architecture/building-blocks/extending-templates/objects-as-parameters).

Para usar la identidad administrada para Azure Data Lake Store Gen1 como receptor de salida, deberá proporcionar acceso a la entidad de servicio con PowerShell antes de la implementación en Azure. Más información acerca de cómo [implementar ADLS Gen1 con identidad administrada con la plantilla de Resource Manager](stream-analytics-managed-identities-adls.md#resource-manager-template-deployment).


## <a name="command-line-tool"></a>Herramienta de línea de comandos

### <a name="build-the-project"></a>Compilación del proyecto
El paquete NuGet tiene una herramienta de línea de comandos denominada **SA.exe**. Admite la compilación del proyecto y las pruebas locales en una máquina arbitraria, que puede usar en el proceso de integración y entrega continuas. 

De manera predeterminada, los archivos de implementación se colocan en el directorio actual. Puede especificar la ruta de acceso de salida mediante el uso del parámetro -OutputPath.

```
./tools/SA.exe build -Project [Your Project Full Path] [-OutputPath <outputPath>] 
```

### <a name="test-the-script-locally"></a>Prueba local del script

Si el proyecto tiene archivos de entrada locales especificados en Visual Studio, puede ejecutar una prueba de script automatizada con el comando *localrun*. El resultado de salida se coloca en el directorio actual.
 
```
localrun -Project [ProjectFullPath]
```

### <a name="generate-a-job-definition-file-to-use-with-the-stream-analytics-powershell-api"></a>Genere un archivo de definición del trabajo para usarlo con la API de Stream Analytics PowerShell

El comando *arm* toma la plantilla de trabajo y los archivos de parámetro de la plantilla de trabajo que se generaron como entrada en la compilación. Luego los combina en un archivo JSON de definición de trabajo que se puede usar con la API de PowerShell de Stream Analytics.

```powershell
arm -JobTemplate <templateFilePath> -JobParameterFile <jobParameterFilePath> [-OutputFile <asaArmFilePath>]
```
Ejemplo:
```powershell
./tools/SA.exe arm -JobTemplate "ProjectA.JobTemplate.json" -JobParameterFile "ProjectA.JobTemplate.parameters.json" -OutputFile "JobDefinition.json" 
```



## <a name="next-steps"></a>Pasos siguientes

* [Inicio rápido: Crear un trabajo en la nube de Azure Stream Analytics en Visual Studio](stream-analytics-quick-create-vs.md)
* [Prueba de las consultas de Stream Analytics localmente con Visual Studio](stream-analytics-vs-tools-local-run.md)
* [Explorar trabajos de Azure Stream Analytics con Visual Studio](stream-analytics-vs-tools.md)
