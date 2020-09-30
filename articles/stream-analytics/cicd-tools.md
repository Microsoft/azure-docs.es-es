---
title: Automatización de compilaciones, pruebas e implementaciones de un trabajo de Azure Stream Analytics mediante herramientas de CI/CD
description: En este artículo se explica cómo usar herramientas de CI/CD de Azure Stream Analytics para compilar, probar e implementar automáticamente un proyecto de Azure Stream Analytics.
services: stream-analytics
author: su-jie
ms.author: sujie
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 09/10/2020
ms.openlocfilehash: e772701396f172eaab906f99463bd9019728b531
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932021"
---
# <a name="automate-builds-tests-and-deployments-of-an-azure-stream-analytics-job-using-cicd-tools"></a>Automatización de compilaciones, pruebas e implementaciones de un trabajo de Azure Stream Analytics mediante herramientas de CI/CD

Puede usar el paquete npm de CI/CD de Azure Stream Analytics para compilar, probar e implementar automáticamente proyectos de Azure Stream Analytics, Visual Studio Code o Visual Studio. Los proyectos pueden crearse mediante herramientas de desarrollo o pueden exportarse desde trabajos existentes de Stream Analytics. En este artículo se explica cómo usar el paquete npm con cualquier sistema de CI/CD. Para la implementación con Azure Pipelines, vea [Uso de Azure DevOps para crear una canalización CI/CD para un trabajo de Stream Analytics](set-up-cicd-pipeline.md).

## <a name="installation"></a>Instalación

Puede [descargar el paquete](https://www.npmjs.com/package/azure-streamanalytics-cicd) directamente o instalarlo [globalmente](https://docs.npmjs.com/downloading-and-installing-packages-globally) con el comando `npm install -g azure-streamanalytics-cicd`. Se recomienda usar el comando, que también se puede emplear en una tarea de script de PowerShell o la CLI de Azure de una canalización de compilación en **Azure Pipelines**.

## <a name="build-the-project"></a>Compilar el proyecto

El paquete npm **asa-streamanalytics-cicd** proporciona las herramientas para generar plantillas de Azure Resource Manager de [proyectos de Visual Studio Code](quick-create-vs-code.md) o [proyectos de Visual Studio](stream-analytics-quick-create-vs.md) de Stream Analytics. También puede usar el paquete npm en Windows, macOS y Linux sin instalar Visual Studio Code ni Visual Studio.

Una vez instalado el paquete, use el siguiente comando para compilar los proyectos de Stream Analytics.

```powershell
azure-streamanalytics-cicd build -project <projectFullPath> [-outputPath <outputPath>]
```

El comando *build* realiza una comprobación de la sintaxis de las palabras clave y genera la plantilla de Azure Resource Manager.

| Parámetro | Descripción |
|---|---|
| `-project` | La ruta absoluta del archivo **asaproj.json** en el caso del proyecto de Visual Studio Code o **[Nombre del proyecto].asaproj** en el caso del proyecto de Visual Studio. |
| `-outputPath` | La ruta de la carpeta de salida en el caso de las plantillas de Azure Resource Manager. Si no se especifica, las plantillas se colocan en el directorio actual. |

#### <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

```powershell
azure-streamanalytics-cicd build -project "/Users/username/projects/samplejob/asaproj.json"
```

#### <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

```powershell
azure-streamanalytics-cicd build -project "/Users/username/projects/samplejob/samplejob.asaproj"
```

--- 

Cuando un proyecto de Stream Analytics se compila correctamente, genera los siguientes dos archivos en la carpeta de salida:

* Archivo de plantilla de Azure Resource Manager

   `[ProjectName].JobTemplate.json`

* Archivo de parámetros de Azure Resource Manager

   `[ProjectName].JobTemplate.parameters.json`

Los parámetros predeterminados del archivo parameters.json proceden de la configuración del proyecto de Visual Studio Code o Visual Studio. Si quiere implementar en otro entorno, reemplace los parámetros según corresponda.

Los valores predeterminados de todas las credenciales son **null**. Se le pide que establezca los valores antes de implementar en Azure.

```json
"Input_EntryStream_sharedAccessPolicyKey": {
      "value": null
    },
```

Para usar la identidad administrada de Azure Data Lake Store Gen1 como receptor de salida, tiene que proporcionar acceso a la entidad de servicio mediante PowerShell antes de implementar en Azure. Más información acerca de cómo [implementar ADLS Gen1 con identidad administrada con la plantilla de Resource Manager](stream-analytics-managed-identities-adls.md#resource-manager-template-deployment).

## <a name="local-run"></a>Ejecución local

Si el proyecto ha especificado archivos de entrada locales, puede ejecutar un script de Stream Analytics en local mediante el comando `localrun`.

```powershell
azure-streamanalytics-cicd localrun -project <projectFullPath> [-outputPath <outputPath>] [-customCodeZipFilePath <zipFilePath>]
```

| Parámetro | Descripción |
|---|---|
| `-project` | La ruta del archivo **asaproj.json** en el caso del proyecto de Visual Studio Code o **[Nombre del proyecto].asaproj** en el caso del proyecto de Visual Studio. |
| `-outputPath` | La ruta de la carpeta de salida. Si no se especifica, los archivos de resultados de salida se colocan en el directorio actual. |
| `-customCodeZipFilePath` | La ruta del archivo zip en el caso del código personalizado de C#, como una UDF o un deserializador, si se usan. Empaquete las DLL en un archivo zip y especifique esta ruta. |

#### <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

```powershell
azure-streamanalytics-cicd localrun -project "/Users/roger/projects/samplejob/asaproj.json"
```

#### <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

```powershell
azure-streamanalytics-cicd localrun -project "/Users/roger/projects/samplejob/samplejob.asaproj"
```

---

> [!Note] 
> Las UDF de JavaScript solo funcionan en Windows.

## <a name="automated-test"></a>Prueba automatizada

Puede usar el paquete npm de CI/CD para configurar y ejecutar pruebas automatizadas del script de Stream Analytics.

### <a name="add-a-test-case"></a>Incorporación de un caso de prueba

Los casos de prueba se describen en un archivo de configuración de prueba. Para empezar, use el comando `addtestcase` para agregar una plantilla de caso de prueba al archivo de configuración de prueba. Si no hay ningún archivo de configuración de prueba, se crea uno de forma predeterminada.

```powershell
azure-streamanalytics-cicd addtestcase -project <projectFullPath> [-testConfigPath <testConfigFileFullPath>]
```

| Parámetro | Descripción |
|---|---|
| `-project` | La ruta del archivo **asaproj.json** en el caso del proyecto de Visual Studio Code o **[Nombre del proyecto].asaproj** en el caso del proyecto de Visual Studio. |
| `-testConfigPath` | La ruta del archivo de configuración de prueba. Si no se especifica, el archivo se busca en **\test** en el directorio actual del archivo **asaproj.json**, con el nombre de archivo predeterminado **testConfig.json**. Se crea un nuevo archivo si no existe. |

#### <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

```powershell
azure-streamanalytics-cicd addtestcase -project "/Users/roger/projects/samplejob/asaproj.json"
```

#### <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

```powershell
azure-streamanalytics-cicd addtestcase -project "/Users/roger/projects/samplejob/samplejob.asaproj"
```
---

Si el archivo de configuración de prueba está vacío, se escribe el siguiente contenido en el archivo. De lo contrario, se agrega un caso de prueba a la matriz de **TestCases**. Las configuraciones de entrada necesarias se rellenan automáticamente según los archivos de configuración de entrada, si existen. De lo contrario, se configuran los valores predeterminados. Se debe especificar el elemento **FilePath** de cada entrada y salida esperada antes de ejecutar la prueba. Se puede cambiar la configuración manualmente.

Si quiere que la validación de la prueba omita una determinada salida, establezca el campo **Requerido** de esa salida esperada en **false**.

```json
{
  "Script": "",
  "TestCases": [
    {
      "Name": "Case 1",
      "Inputs": [
        {
          "InputAlias": [Input alias string],
          "Type": "Data Stream",
          "Format": "JSON",
          "FilePath": [Required],
          "ScriptType": "InputMock"
        }
      ],
      "ExpectedOutputs": [
        {
          "OutputAlias": [Output alias string],
          "FilePath": "Required",
          "Required": true
        }
      ]
    }
  ]
}
```

### <a name="run-a-unit-test"></a>Ejecución de una prueba unitaria

Puede usar el siguiente comando para ejecutar varios casos de prueba para el proyecto. Se genera un resumen de los resultados de prueba en la carpeta de salida. El proceso sale con el código **0** para todas las pruebas correctas; **-1** para las excepciones; **-2** para las pruebas erróneas.

```powershell
azure-streamanalytics-cicd test -project <projectFullPath> [-testConfigPath <testConfigFileFullPath>] [-outputPath <outputPath>] [-customCodeZipFilePath <zipFilePath>]
```

| Parámetro | Descripción |
|---|---|
| `-project` | La ruta del archivo **asaproj.json** en el caso del proyecto de Visual Studio Code o **[Nombre del proyecto].asaproj** en el caso del proyecto de Visual Studio. |
| `-testConfigPath` | La ruta del archivo de configuración de prueba. Si no se especifica, el archivo se busca en **\test** en el directorio actual del archivo **asaproj.json**, con el nombre de archivo predeterminado **testConfig.json**.
| `-outputPath` | La ruta de la carpeta de salida del resultado de la prueba. Si no se especifica, los archivos de resultados de salida se colocan en el directorio actual. |
| `-customCodeZipFilePath` | La ruta del archivo zip en el caso del código personalizado, como una UDF o un deserializador, si se usan. |

Cuando terminan todas las pruebas, se genera un resumen de los resultados de prueba en formato JSON en la carpeta de salida. El archivo de resumen se denomina **testResultSummary.json**.

```json
{
  "Total": (integer) total_number_of_test_cases,
  "Passed": (integer) number_of_passed_test_cases,
  "Failed": (integer) number_of_failed_test_cases,
  "Script": (string) absolute_path_to_asaql_file,
  "Results": [ (array) detailed_results_of_test_cases
    {
      "Name": (string) name_of_test_case,
      "Status": (integer) 0(passed)_or_1(failed),
      "Time": (string) time_span_of_running_test_case,
      "OutputMatched": [ (array) records_of_actual_outputs_equal_to_expected_outputs
        {
          "OutputAlias": (string) output_alias,
          "ExpectedOutput": (string) path_to_the_expected_output_file,
          "Output": (string) path_to_the_actual_output_file
        }
      ],
      "OutputNotEqual": [ (array) records_of_actual_outputs_not_equal_to_expected_outputs
        {
          "OutputAlias": (string) output_alias,
          "ExpectedOutput": (string) path_to_the_expected_output_file,
          "Output": (string) path_to_the_actual_output_file
        }
      ],
      "OutputMissing": [ (array) records_of_actual_outputs_missing
        {
          "OutputAlias": (string) output_alias,
          "ExpectedOutput": (string) path_to_the_expected_output_file,
          "Output": ""
        }
      ],
      "OutputUnexpected": [ (array) records_of_actual_outputs_unexpected
        {
          "OutputAlias": (string) output_alias,
          "ExpectedOutput": "",
          "Output": (string) path_to_the_actual_output_file
        }
      ],
      "OutputUnrequired": [ (array) records_of_actual_outputs_unrequired_to_be_checked
        {
          "OutputAlias": (string) output_alias,
          "ExpectedOutput": (string) path_to_the_expected_output_file,
          "Output": (string) path_to_the_actual_output_file
        }
      ]
    }
  ],
  "Time": (string) time_span_of_running_all_test_cases,
}
```

## <a name="deploy-to-azure"></a>Implementar en Azure

Puede usar la plantilla de Azure Resource Manager y los archivos de parámetros generados a partir de la compilación para [implementar el trabajo en Azure](../azure-resource-manager/templates/template-tutorial-use-parameter-file.md?tabs=azure-powershell#deploy-template).

## <a name="next-steps"></a>Pasos siguientes

* [Integración continua e implementación continua en Azure Stream Analytics](cicd-overview.md)
* [Configuración de canalizaciones de integración y entrega continuas para trabajos de Stream Analytics mediante Azure Pipelines](set-up-cicd-pipeline.md)
