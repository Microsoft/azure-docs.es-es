---
title: Registro y diagnóstico
description: Explicación detallada sobre cómo generar y recuperar registros y diagnósticos en Azure Spatial Anchors.
author: ramonarguelles
manager: vriveras
services: azure-spatial-anchors
ms.author: rgarcia
ms.date: 02/22/2019
ms.topic: conceptual
ms.service: azure-spatial-anchors
ms.custom: devx-track-csharp
ms.openlocfilehash: da941aa8b616fffeb2e283480c2a1e2f9cf7a093
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89003123"
---
# <a name="logging-and-diagnostics-in-azure-spatial-anchors"></a>Registro y diagnóstico en Azure Spatial Anchors

Azure Spatial Anchors brinda un mecanismo de registro estándar que resulta útil para el desarrollo de aplicaciones. El modo de registro de diagnóstico de Spatial Anchors es útil cuando se necesita más información para realizar una depuración. El registro de diagnóstico almacena imágenes del entorno.

## <a name="standard-logging"></a>Registro estándar
En la API Spatial Anchors, puede suscribirse al mecanismo de registro para obtener registros útiles para el desarrollo y la depuración de aplicaciones. Las API de registro estándar no almacenan imágenes del entorno en el disco del dispositivo. El SDK proporciona estos registros como devoluciones de llamada de eventos. De usted depende integrar estos registros en el mecanismo de registro de la aplicación.

### <a name="configuration-of-log-messages"></a>Configuración de los mensajes de registro
Existen dos devoluciones de llamada de interés para el usuario. En el ejemplo siguiente se muestra cómo configurar la sesión.

```csharp
    cloudSpatialAnchorSession = new CloudSpatialAnchorSession();
    . . .
    // set up the log level for the runtime session
    cloudSpatialAnchorSession.LogLevel = SessionLogLevel.Information;

    // configure the callback for the debug log
    cloudSpatialAnchorSession.OnLogDebug += CloudSpatialAnchorSession_OnLogDebug;

    // configure the callback for the error log
    cloudSpatialAnchorSession.Error += CloudSpatialAnchorSession_Error;
```

### <a name="events-and-properties"></a>Eventos y propiedades

Estas devoluciones de llamada de eventos se proporcionan para procesar los registros y errores de la sesión:

- [LogLevel](https://docs.microsoft.com/dotnet/api/microsoft.azure.spatialanchors.cloudspatialanchorsession.loglevel): especifica el nivel de detalle de los eventos que se van a recibir desde el entorno en tiempo de ejecución.
- [OnLogDebug](https://docs.microsoft.com/dotnet/api/microsoft.azure.spatialanchors.cloudspatialanchorsession.onlogdebug): proporciona eventos de registro de depuración estándar.
- [Error](https://docs.microsoft.com/dotnet/api/microsoft.azure.spatialanchors.cloudspatialanchorsession.error): proporciona eventos de registro que el entorno en tiempo de ejecución considera como errores.

## <a name="diagnostics-logging"></a>Registro de diagnóstico

Además del modo estándar de operación para el registro, Spatial Anchors también tiene un modo de diagnóstico. El modo de diagnóstico captura imágenes del entorno y las registra en el disco. Puede usar este modo para depurar ciertos tipos de problemas, como errores al ubicar un delimitador de manera predecible. Habilite el registro de diagnóstico solo para reproducir un problema específico. Luego, deshabilítelo. No habilite el diagnóstico si está ejecutando las aplicaciones con normalidad.

Durante una interacción de soporte técnico con Microsoft, un representante de Microsoft podría preguntarle si está dispuesto a enviar un conjunto de diagnósticos para realizar una investigación más detallada. En este caso, puede decidir habilitar el diagnóstico y reproducir el error para poder enviar el conjunto de diagnósticos.

Si envía un registro de diagnóstico a Microsoft sin el conocimiento previo de un representante de Microsoft, el envío no recibirá respuesta.

En las secciones siguientes se muestra cómo habilitar el modo de diagnóstico y también cómo enviar registros de diagnóstico a Microsoft.

### <a name="enable-diagnostics-logging"></a>Habilitación del registro de diagnósticos

Cuando se habilita una sesión para el registro de diagnóstico, todas las operaciones de la sección tienen un registro de diagnóstico correspondiente en el sistema de archivos local. Durante el registro, las imágenes del entorno se guardan en el disco.

```csharp
private void ConfigureSession()
{
    cloudSpatialAnchorSession = new CloudSpatialAnchorSession();
    . . .

    // set up the log level for the runtime session
    cloudSpatialAnchorSession.LogLevel = SessionLogLevel.Information;

    // configure the callbacks for logging and errors
    cloudSpatialAnchorSession.OnLogDebug += CloudSpatialAnchorSession_OnLogDebug;
    cloudSpatialAnchorSession.Error += CloudSpatialAnchorSession_Error;

    // opt in to diagnostics logging of environment images
    // if this is enabled, the diagnostics bundle includes images of the environment captured by the session
    cloudSpatialAnchorSession.Diagnostics.ImagesEnabled = true;

    // set the level of detail to be collected in the diagnostics log by the session
    cloudSpatialAnchorSession.Diagnostics.LogLevel = SessionLogLevel.All;

    // set the max bundle size to capture the bug information
    cloudSpatialAnchorSession.Diagnostics.MaxDiskSizeInMB = 200;
    . . .
}
```

### <a name="submit-the-diagnostics-bundle"></a>Envío del conjunto de diagnósticos

En el fragmento de código siguiente se muestra cómo enviar un conjunto de diagnósticos a Microsoft. Este conjunto incluirá las imágenes del entorno que capturó la sesión después de que habilitó los diagnósticos.

```csharp
// method to handle the diagnostics bundle submission
private async Task CreateAndSubmitBundle()
{
    // create the diagnostics bundle manifest to collect the session information
    string path = await cloudSpatialAnchorSession
                              .Diagnostics
                              .CreateManifestAsync("Description of the issue");

    // submit the manifest and data to send feedback to Microsoft
    await cloudSpatialAnchorSession.Diagnostics.SubmitManifestAsync(path);
}
```

### <a name="parts-of-a-diagnostics-bundle"></a>Partes de un conjunto de diagnósticos
El conjunto de diagnósticos podría contener esta información:

- **Imágenes de fotograma clave**: las imágenes del entorno capturadas durante la sesión mientras el modo de diagnóstico estaba habilitado.
- **Registros**: los eventos de registro registrados por el entorno en tiempo de ejecución.
- **Metadatos de la sesión**: los metadatos que identifican a la sesión.
