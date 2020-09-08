---
title: Actualización a la versión 5 del SDK de .NET para Azure Search
titleSuffix: Azure Cognitive Search
description: Migre el código a la versión 5 del SDK de Azure Search para .NET a partir de versiones anteriores. Obtenga información sobre las novedades y los cambios de código necesarios.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
ms.service: cognitive-search
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: ade326b7dda19478398de4ec3ab2be8996d98535
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88923218"
---
# <a name="upgrade-to-azure-search-net-sdk-version-5"></a>Actualización a la versión 5 del SDK de .NET para Azure Search

Si usa la versión preliminar 4.0 o anterior del [SDK de .NET](/dotnet/api/overview/azure/search), este artículo le ayudará a actualizar la aplicación para que use la versión 5.

Para obtener un tutorial más general del SDK que incluya ejemplos, vea [Cómo usar Azure Search desde una aplicación .NET](search-howto-dotnet-sdk.md).

La versión 5 del SDK de .NET para Azure Search contiene algunos cambios de versiones anteriores. La mayoría son de menor importancia, por lo que cambiar el código solo precisará del mínimo esfuerzo. Vea [Pasos para actualizar](#UpgradeSteps) , a fin de obtener instrucciones sobre cómo cambiar el código para usar la nueva versión del SDK.

> [!NOTE]
> Si usa la versión 2.0-preview o anterior, debe actualizar primero a la versión 3 y luego a la versión 5. Consulte [Actualización a la versión 3 del SDK de .NET para Azure Search](search-dotnet-sdk-migration.md) para instrucciones.
>
> La instancia del servicio Azure Search es compatible con varias versiones de API de REST, incluida la más reciente. Puede seguir usando una versión aunque no sea la más reciente, pero es recomendable que migre el código para usar la versión más actualizada. Cuando se usa la API de REST, debe especificar la versión de API en cada solicitud realizada a través del parámetro api-version. Al usar el SDK de .NET, la versión del SDK que usa determina la versión correspondiente de la API de REST. Si usa un SDK anterior, puede seguir ejecutando ese código sin realizar ningún cambio, incluso si el servicio se ha actualizado para admitir una versión más reciente de la API.

<a name="WhatsNew"></a>

## <a name="whats-new-in-version-5"></a>Novedades de la versión 5
La versión 5 del SDK de .NET para Azure Search tiene como destino la versión más reciente con disponibilidad general de la API de REST de Azure Search, en concreto la del 11-11-2017. Esto hace posible el uso de características nuevas de Azure Search desde una aplicación. NET, incluidas las siguientes:

* [Sinónimos](search-synonyms.md).
* Ahora puede acceder mediante programación a las advertencias del historial de ejecución del indizador (consulte la propiedad `Warning` de `IndexerExecutionResult` en la [referencia de .NET](/dotnet/api/microsoft.azure.search.models.indexerexecutionresult?view=azure-dotnet) para obtener más detalles).
* Compatibilidad con .NET Core 2.
* La estructura del nuevo paquete permite utilizar solamente las partes del SDK que se necesitan (vea [cambios importantes en la versión 5](#ListOfChanges) para obtener información detallada).

<a name="UpgradeSteps"></a>

## <a name="steps-to-upgrade"></a>Pasos para actualizar
En primer lugar, actualice la referencia de NuGet para `Microsoft.Azure.Search` mediante la Consola del Administrador de paquetes NuGet, o bien haga clic con el botón derecho en las referencias del proyecto y seleccione "Administrar paquetes NuGet" en Visual Studio.

Una vez que NuGet ha descargado los nuevos paquetes y sus dependencias, recompile el proyecto. Dependiendo de cómo se estructura el código, se podrá volver a compilar correctamente. Si lo consigue, ya estará listo para empezar.

Si se produce un error en la compilación, verá un mensaje similar al siguiente:

```output
The name 'SuggesterSearchMode' does not exist in the current context
```

El paso siguiente consiste en corregir los errores de compilación. Consulte [Cambios importantes en la versión 5](#ListOfChanges) para más información sobre las causas del error y cómo corregirlo.

Tenga en cuenta que, debido a cambios efectuados en el empaquetado del SDK de .NET de Azure Search, deberá recompilar la aplicación para poder usar la versión 5. Estos cambios se detallan en [Cambios importantes en la versión 5](#ListOfChanges).

Puede ver las advertencias de compilación adicionales relacionadas con propiedades o métodos obsoletos. Las advertencias incluyen instrucciones sobre qué utilizar en lugar de la característica en desuso. Por ejemplo, si la aplicación usa el método `IndexingParametersExtensions.DoNotFailOnUnsupportedContentType`, recibirá una advertencia que indica lo siguiente: "Este comportamiento ahora está habilitado de forma predeterminada, por lo que ya no es necesario llamar a este método".

Una vez que haya solucionado los errores o las advertencias de compilación, puede efectuar cambios en la aplicación para aprovechar las ventajas de la nueva funcionalidad, si lo desea. Las nuevas características en el SDK se detallan en [Novedades de la versión 5](#WhatsNew).

<a name="ListOfChanges"></a>

## <a name="breaking-changes-in-version-5"></a>Cambios importantes en la versión 5

### <a name="new-package-structure"></a>Nueva estructura de paquete

El cambio importante más significativo de la versión 5 es que el ensamblado `Microsoft.Azure.Search` y su contenido se han dividido en cuatro ensamblados independientes que ahora se distribuyen como cuatro paquetes de NuGet independientes:

 - `Microsoft.Azure.Search`: es un metapaquete que incluye el resto de los paquetes de Azure Search a modo de dependencias. Si va a actualizar desde una versión anterior del SDK, debería bastar con actualizar este paquete y recompilarlo para empezar a usar la nueva versión.
 - `Microsoft.Azure.Search.Data`: use este paquete si va a desarrollar una aplicación de .NET mediante Azure Search y solo necesita consultar o actualizar los documentos en los índices. Si también tiene que crear o actualizar los índices, las asignaciones de sinónimos u otros recursos de nivel de servicio, use el paquete `Microsoft.Azure.Search`.
 - `Microsoft.Azure.Search.Service`: use este paquete si está desarrollando la automatización en .NET para administrar los índices, las asignaciones de sinónimos, los indexadores, los orígenes de datos u otros recursos de nivel de servicio de Azure Search. Si solo necesita consultar o actualizar los documentos de los índices, use el paquete `Microsoft.Azure.Search.Data` en su lugar. Si necesita toda la funcionalidad de Azure Search, use el paquete `Microsoft.Azure.Search`.
 - `Microsoft.Azure.Search.Common`: tipos comunes que necesitan las bibliotecas de .NET de Azure Search. No es necesario usar este paquete directamente en la aplicación; solo debe usarse como una dependencia.
 
Este cambio es técnicamente importante porque se han movido muchos tipos de un ensamblado a otro. Por este motivo, es necesario recompilar la aplicación para poder actualizar a la versión 5 del SDK.

Hay unos pocos cambios importantes más en la versión 5 que pueden requerir cambios de código además de la recompilación de la aplicación.

### <a name="change-to-suggesters"></a>Cambio a los proveedores de sugerencias 

El constructor `Suggester` ya no tiene un parámetro `enum` para `SuggesterSearchMode`. Esta enumeración solo tenía un valor y, por tanto, era redundante. Si como resultado se observan errores de compilación, basta con eliminar las referencias al parámetro `SuggesterSearchMode`.

### <a name="removed-obsolete-members"></a>Miembros obsoletos quitados

Es posible que vea errores de compilación relacionados con métodos o propiedades que se marcaron como obsoletos en versiones anteriores y que posteriormente se eliminaron en la versión 5. Si se encuentra con tales errores, aquí le decimos cómo resolverlos:

- Si usaba el método `IndexingParametersExtensions.IndexStorageMetadataOnly`, use `SetBlobExtractionMode(BlobExtractionMode.StorageMetadata)`.
- Si usaba el método `IndexingParametersExtensions.SkipContent`, use `SetBlobExtractionMode(BlobExtractionMode.AllMetadata)`.

### <a name="removed-preview-features"></a>Características quitadas de la versión preliminar

Si va a actualizar de la versión 4.0-preview a la versión 5, debe saber que se ha eliminado la compatibilidad con el análisis de la matriz JSON y CSV en los indexadores de blobs, dado que estas características aún se encuentran en versión preliminar. En concreto, se han quitado los siguientes métodos de la clase `IndexingParametersExtensions`:

- `ParseJsonArrays`
- `ParseDelimitedTextFiles`

Si la aplicación tiene una dependencia fuerte de estas características, no podrá actualizar a la versión 5 del SDK de Azure Search para .NET. Podrá seguir usando la versión preliminar 4.0. Sin embargo, tenga en cuenta que **no se recomienda usar SDK de versión preliminar en aplicaciones de producción**. Las características de versión preliminar son solo para evaluación y pueden cambiar.

## <a name="conclusion"></a>Conclusión
Si necesita más detalles sobre el uso del SDK de .NET para Azure Search, consulte la [guía de .NET](search-howto-dotnet-sdk.md).

Agradecemos sus comentarios sobre el SDK. Si tiene algún problema, no dude en pedirnos ayuda en [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-search). Si encuentra un error, puede enviarlo al [repositorio de GitHub del SDK de .NET para Azure](https://github.com/Azure/azure-sdk-for-net/issues). Asegúrese de agregar "[Azure Search]" como prefijo en el título del problema.

Gracias por usar Azure Search.