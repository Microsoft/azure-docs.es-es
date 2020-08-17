---
title: Conexión de la función de Java a Azure Storage
description: Aprenda a conectar una función de Java desencadenada por HTTP a Azure Storage mediante un enlace de salida de Queue Storage.
author: KarlErickson
ms.custom: devx-track-java
ms.author: karler
ms.date: 10/14/2019
ms.topic: quickstart
zone_pivot_groups: java-build-tools-set
ms.openlocfilehash: 9f512e3bbf7947361fa9890e9514693610c9f99d
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87321962"
---
# <a name="connect-your-java-function-to-azure-storage"></a>Conexión de la función de Java a Azure Storage

[!INCLUDE [functions-add-storage-binding-intro](../../includes/functions-add-storage-binding-intro.md)]

En este artículo se muestra cómo integrar la función que creó en el [artículo de inicio rápido anterior](./functions-create-first-azure-function-azure-cli.md?pivots=programming-language-java&tabs=bash,browser) con una cola de Azure Storage. El enlace de salida que se agrega a esta función escribe datos de una solicitud HTTP en un mensaje de la cola.

La mayoría de los enlaces requieren una cadena de conexión almacenada que se usa en Functions para acceder al servicio enlazado. Para facilitar la conexión, usará la cuenta de almacenamiento que creó con la aplicación de funciones. La conexión a esta cuenta ya está almacenada en una configuración de aplicación llamada `AzureWebJobsStorage`.  

## <a name="prerequisites"></a>Prerrequisitos

Antes de empezar este artículo, realice los pasos de la [parte 1 del inicio rápido de Java](./functions-create-first-azure-function-azure-cli.md?pivots=programming-language-java&tabs=bash,browser).

## <a name="download-the-function-app-settings"></a>Descarga de la configuración de la aplicación de función

[!INCLUDE [functions-app-settings-download-cli](../../includes/functions-app-settings-download-local-cli.md)]

## <a name="enable-extension-bundles"></a>Habilitación de conjuntos de extensiones

[!INCLUDE [functions-extension-bundles](../../includes/functions-extension-bundles.md)]

Ahora podrá agregar el enlace de salida de almacenamiento al proyecto.

## <a name="add-an-output-binding"></a>Adición de un enlace de salida

[!INCLUDE [functions-add-output-binding-java-cli](../../includes/functions-add-output-binding-java-cli.md)]

## <a name="add-code-that-uses-the-output-binding"></a>Adición de código que utilice el enlace de salida

[!INCLUDE [functions-add-output-binding-java-code](../../includes/functions-add-output-binding-java-code.md)]

[!INCLUDE [functions-add-output-binding-java-test-cli](../../includes/functions-add-output-binding-java-test-cli.md)]

Ahora ya está listo para probar el nuevo enlace de salida de forma local.

## <a name="run-the-function-locally"></a>Ejecución local de la función

Como antes, use el siguiente comando para crear el proyecto e iniciar el sistema en tiempo de ejecución de Functions localmente:

# <a name="maven"></a>[Maven](#tab/maven)
```bash
mvn clean package 
mvn azure-functions:run
```
# <a name="gradle"></a>[Gradle](#tab/gradle) 
```bash
gradle jar --info
gradle azureFunctionsRun
```
---

> [!NOTE]  
> Como ya habilitó las agrupaciones de extensiones en el archivo host.json, la [extensión de enlace de Storage](functions-bindings-storage-blob.md#add-to-your-functions-app) se descargó e instaló automáticamente durante el inicio junto con el resto de extensiones de enlace de Microsoft.

Al igual que antes, desencadene la función desde la línea de comandos mediante cURL en una ventana de terminal nueva:

```CMD
curl -w "\n" http://localhost:7071/api/HttpTrigger-Java --data AzureFunctions
```

Esta vez, el enlace de salida también crea una cola denominada `outqueue` en la cuenta de almacenamiento y agrega un mensaje con esta misma cadena.

A continuación, se usa la CLI de Azure para ver la nueva cola y comprobar que se ha agregado un mensaje. También puede ver la cola mediante el [Explorador de Microsoft Azure Storage][Azure Storage Explorer] o en [Azure Portal](https://portal.azure.com).

[!INCLUDE [functions-storage-account-set-cli](../../includes/functions-storage-account-set-cli.md)]

[!INCLUDE [functions-query-storage-cli](../../includes/functions-query-storage-cli.md)]

### <a name="redeploy-the-project"></a>Reimplementación del proyecto 

Para actualizar la aplicación publicada, vuelva a ejecutar el siguiente comando:  

# <a name="maven"></a>[Maven](#tab/maven)  
```bash
mvn azure-functions:deploy
```
# <a name="gradle"></a>[Gradle](#tab/gradle)  
```bash
gradle azureFunctionsDeploy
```
---

De nuevo, puede usar cURL para probar la función implementada. Como antes, pase el valor `AzureFunctions` del cuerpo de la solicitud POST a la dirección URL, como en este ejemplo:

```bash
curl -w "\n" https://fabrikam-functions-20190929094703749.azurewebsites.net/api/HttpTrigger-Java?code=zYRohsTwBlZ68YF.... --data AzureFunctions
```

Puede [examinar el mensaje de la cola de almacenamiento](#query-the-storage-queue) otra vez para comprobar que el enlace de salida genera un nuevo mensaje en la cola según lo previsto.

[!INCLUDE [functions-cleanup-resources](../../includes/functions-cleanup-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

Ha actualizado la función desencadenada por HTTP para escribir datos en una cola de almacenamiento. Para más información sobre el desarrollo de Azure Functions con Java, consulte la [Guía de Azure Functions para desarrolladores de Java](functions-reference-java.md) y [Desencadenadores y enlaces en Azure Functions](functions-triggers-bindings.md). Para obtener ejemplos de proyectos de Functions completos en Java, consulte los [ejemplos de Functions para Java](/samples/browse/?products=azure-functions&languages=Java). 

A continuación, debe habilitar la supervisión de Application Insights para su aplicación de función:

> [!div class="nextstepaction"]
> [Habilitación de la integración de Application Insights](functions-monitoring.md#manually-connect-an-app-insights-resource)


[Azure Storage Explorer]: https://storageexplorer.com/
