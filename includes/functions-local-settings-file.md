---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 04/14/2019
ms.author: glenga
ms.openlocfilehash: 47e1c509e8b7b60e889e1202b49b1a145c68162c
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88929504"
---
## <a name="local-settings-file"></a>Archivo de configuración local

El archivo local.settings.json almacena la configuración de la aplicación, las cadenas de conexión y la configuración que usan las herramientas locales de desarrollo. La configuración del archivo local.settings.json solo se usa al ejecutar los proyectos de forma local. El archivo de configuración local tiene la siguiente estructura:

```json
{
  "IsEncrypted": false,
  "Values": {
    "FUNCTIONS_WORKER_RUNTIME": "<language worker>",
    "AzureWebJobsStorage": "<connection-string>",
    "AzureWebJobsDashboard": "<connection-string>",
    "MyBindingConnection": "<binding-connection-string>",
    "AzureWebJobs.HttpExample.Disabled": "true"
  },
  "Host": {
    "LocalHttpPort": 7071,
    "CORS": "*",
    "CORSCredentials": false
  },
  "ConnectionStrings": {
    "SQLConnectionString": "<sqlclient-connection-string>"
  }
}
```

Esta configuración se admite al ejecutar los proyectos de forma local:

| Configuración      | Descripción                            |
| ------------ | -------------------------------------- |
| **`IsEncrypted`** | Cuando la configuración se establece en `true`, todos los valores se cifran con una clave de máquina local. Se usa con los comandos `func settings`. El valor predeterminado es `false`. |
| **`Values`** | Matriz de opciones de configuración de la aplicación y cadenas de conexión que se usan al ejecutar un proyecto de forma local. Estos pares clave-valor (cadena-cadena) corresponden a la configuración de la aplicación en su aplicación de funciones de Azure, como [`AzureWebJobsStorage`]. Muchos desencadenadores y enlaces tienen una propiedad que hace referencia a una configuración de la aplicación de cadena de conexión, por ejemplo, `Connection` para el [desencadenador del almacén de blobs](../articles/azure-functions/functions-bindings-storage-blob-trigger.md#configuration). Para estas propiedades, se necesita una configuración de la aplicación definida en la matriz `Values`. Consulte la tabla posterior, donde encontrará una lista de los valores que se usan con más frecuencia. <br/>Los valores deben ser cadenas y no objetos JSON o matrices. Los nombres de la configuración no pueden incluir dos puntos (`:`) ni un subrayado doble (`__`). Los caracteres dobles de subrayado están reservados para el tiempo de ejecución, mientras que el signo de dos puntos está reservado para admitir [la inserción de dependencias](../articles/azure-functions/functions-dotnet-dependency-injection.md#working-with-options-and-settings). |
| **`Host`** | La configuración que se muestra en esta sección permite personalizar el proceso de host de Functions al ejecutar los proyectos de forma local. Dicha configuración es independiente de la de host.json, que también se aplica al ejecutar proyectos en Azure. |
| **`LocalHttpPort`** | Establece el puerto predeterminado que se usa cuando al ejecutar el host de Functions local (`func host start` y `func run`). La opción de línea de comandos `--port` tiene prioridad sobre esta configuración. |
| **`CORS`** | Define los orígenes permitidos para el [uso compartido de recursos entre orígenes (CORS)](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing). Los orígenes se proporcionan en una lista de valores separados por comas y sin espacios. Se admite el valor comodín (\*), lo que permite realizar solicitudes desde cualquier origen. |
| **`CORSCredentials`** |  Al establecer el valor en `true`, se permiten las solicitudes `withCredentials`. |
| **`ConnectionStrings`** | Identificador de colección. No use dicha colección para las cadenas de conexión que empleen enlaces de función. Esta colección solo la usan los marcos que habitualmente obtienen las cadenas de conexión de la sección `ConnectionStrings` de un archivo de configuración, como [Entity Framework](https://msdn.microsoft.com/library/aa937723(v=vs.113).aspx). Las cadenas de conexión de este objeto se agregan al entorno con el tipo de proveedor de [System.Data.SqlClient](https://msdn.microsoft.com/library/system.data.sqlclient(v=vs.110).aspx). Los elementos de esta colección no se publican en Azure con otra configuración de aplicación. Debe agregar explícitamente estos valores a la colección `Connection strings` de la configuración de la aplicación de función. Si quiere crear un objeto [`SqlConnection`](https://msdn.microsoft.com/library/system.data.sqlclient.sqlconnection(v=vs.110).aspx) en el código de la función, debe almacenar el valor de la cadena de conexión con las otras conexiones en **Configuración de la aplicación**, en el portal. |

Los siguientes valores de la aplicación se pueden incluir en la matriz **`Values`** cuando la ejecución se realiza en un entorno local:

| Configuración | Valores | Descripción |
|-----|-----|-----|
|**`AzureWebJobsStorage`**| Cadena de conexión de la cuenta de almacenamiento, o bien<br/>`UseDevelopmentStorage=true`| Contiene la cadena de conexión de una cuenta de Azure Storage. Se requiere cuando se utilizan desencadenadores distintos de HTTP. Para más información, consulte la referencia de [`AzureWebJobsStorage`].<br/>Si tiene el [emulador de Azure Storage](../articles/storage/common/storage-use-emulator.md) instalado localmente y ha establecido [`AzureWebJobsStorage`] en `UseDevelopmentStorage=true`, Core Tools usará el emulador. Dicho emulador es útil durante el desarrollo, pero conviene probarlo con una conexión de almacenamiento real antes de la implementación.| 
|**`AzureWebJobs.<FUNCTION_NAME>.Disabled`**| `true`\|`false` | Para deshabilitar una función cuando la ejecución se realizar en un entorno local, agregue `"AzureWebJobs.<FUNCTION_NAME>.Disabled": "true"` a la colección, donde `<FUNCTION_NAME>` es el nombre de la función. Para más información, consulte [Deshabilitamiento de funciones en Azure Functions](../articles/azure-functions/disable-function.md#localsettingsjson) |
|**`FUNCTIONS_WORKER_RUNTIME`** | `dotnet`<br/>`node`<br/>`java`<br/>`powershell`<br/>`python`| Indica el lenguaje de destino del runtime de Functions. Se requiere para la versión 2.x y superior del runtime de Functions. Este valor lo genera Azure Functions Core Tools para el proyecto. Para más información, consulte la referencia de [`FUNCTIONS_WORKER_RUNTIME`](../articles/azure-functions/functions-app-settings.md#functions_worker_runtime).|
| **`FUNCTIONS_WORKER_RUNTIME_VERSION`** | `~7` |Indica que PowerShell 7 se debe usar cuando la ejecución se realice localmente. Si no se establece, se usa PowerShell Core 6. Este valor solo se usa cuando la ejecución se realice localmente. Cuando se ejecuta en Azure, la versión del runtime de PowerShell viene determinada por el valor de configuración del sitio `powerShellVersion`, que se puede [establecer en el portal](../articles/azure-functions/functions-reference-powershell.md#changing-the-powershell-version). | 

[`AzureWebJobsStorage`]: ../articles/azure-functions/functions-app-settings.md#azurewebjobsstorage
