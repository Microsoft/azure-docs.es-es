---
title: Actualización a la versión 10 del SDK de .NET
titleSuffix: Azure Cognitive Search
description: Migración del código a la versión 10 del SDK de .NET para Azure Cognitive Search a partir de versiones anteriores. Obtenga información sobre las novedades y los cambios de código necesarios.
manager: nitinme
author: arv100kri
ms.author: arjagann
ms.service: cognitive-search
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 11/04/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: bfe24ff38446fa0d0ccea96799e6f42b561713bf
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89002817"
---
# <a name="upgrade-to-azure-cognitive-search-net-sdk-version-10"></a>Actualización a la versión 10 del SDK de .NET para Azure Cognitive Search

Si usa la versión 9.0 o anterior del [SDK de .NET](/dotnet/api/overview/azure/search), este artículo le ayudará a actualizar la aplicación para que use la versión 10.

Se ha cambiado el nombre a Azure Search por el de Azure Cognitive Search en la versión 10, pero los espacios de nombres y los nombres de paquetes no se han modificado. Las versiones anteriores del SDK (9.0 y versiones anteriores) siguen usando el nombre anterior. Para más información sobre el SDK (ejemplos incluidos), consulte [Cómo usar Azure Cognitive Search desde una aplicación .NET](search-howto-dotnet-sdk.md).

La versión 10 agrega varias características y correcciones de errores, lo que la sitúa en el mismo nivel funcional que la versión de la API de REST `2019-05-06`. En los casos en los que un cambio interrumpa el código existente, le guiaremos a lo largo de los [pasos necesarios para resolver el problema.](#UpgradeSteps)

> [!NOTE]
> Si usa la versión 8.0-preview o anterior, debe actualizar primero a la versión 9 y luego a la versión 10. Consulte [Actualización a la versión 9 del SDK de .NET para Azure Search](search-dotnet-sdk-migration-version-9.md) para obtener instrucciones.
>
> La instancia del servicio Search es compatible con varias versiones de API REST, incluida la más reciente. Puede seguir usando una versión aunque no sea la más reciente, pero es recomendable que migre el código para usar la versión más actualizada. Cuando se usa la API de REST, debe especificar la versión de API en cada solicitud realizada a través del parámetro api-version. Al usar el SDK de .NET, la versión del SDK que usa determina la versión correspondiente de la API de REST. Si usa un SDK anterior, puede seguir ejecutando ese código sin realizar ningún cambio, incluso si el servicio se ha actualizado para admitir una versión más reciente de la API.

<a name="WhatsNew"></a>

## <a name="whats-new-in-version-10"></a>Novedades de la versión 10
La versión 10 del SDK de .NET para Azure Cognitive Search tiene como destino la API de REST `2019-05-06` con estas actualizaciones:

* Introducción de dos nuevas aptitudes: [aptitud Condicional](cognitive-search-skill-conditional.md) y [aptitud Traducción de texto](cognitive-search-skill-text-translation.md).
* Las entradas de la [aptitud Conformador](cognitive-search-skill-shaper.md) se han reestructurado para dar cabida a la consolidación de contextos anidados. Para más información, consulte esta [definición de JSON de ejemplo](./cognitive-search-skill-shaper.md#scenario-3-input-consolidation-from-nested-contexts).
* Incorporación de dos nuevas [funciones de asignación de campos](search-indexer-field-mappings.md):
    - [urlEncode](./search-indexer-field-mappings.md#urlencode-function)
    - [urlDecode](./search-indexer-field-mappings.md#urldecode-function)
* En algunas ocasiones, los errores y las advertencias que se muestran en el [estado de ejecución del indexador](/rest/api/searchservice/get-indexer-status) pueden tener detalles adicionales que ayudan a la depuración. `IndexerExecutionResult` se ha actualizado para reflejar este comportamiento.
* Las aptitudes individuales definidas dentro de un [conjunto de aptitudes](cognitive-search-defining-skillset.md) se pueden identificar opcionalmente especificando una propiedad `name`.
* `ServiceLimits` muestra los límites de los [tipos complejos](./search-howto-complex-data-types.md) y `IndexerExecutionInfo` muestra las cuotas o los límites del indexador pertinentes.

<a name="UpgradeSteps"></a>

## <a name="steps-to-upgrade"></a>Pasos para actualizar

1. Actualice la referencia de NuGet para `Microsoft.Azure.Search` mediante la Consola del Administrador de paquetes NuGet, o bien haga clic con el botón derecho en las referencias del proyecto y seleccione "Administrar paquetes NuGet" en Visual Studio.

2. Una vez que NuGet ha descargado los nuevos paquetes y sus dependencias, recompile el proyecto. 

3. Si se produce un error en la compilación, deberá corregir cada error de compilación. Consulte [Cambios importantes en la versión 10](#ListOfChanges) para más información sobre cómo resolver cada posible error de compilación.

4. Una vez que haya solucionado los errores o las advertencias de compilación, puede efectuar cambios en la aplicación para aprovechar las ventajas de la nueva funcionalidad, si lo desea. Las nuevas características en el SDK se detallan en [Novedades de la versión 10](#WhatsNew).

<a name="ListOfChanges"></a>

## <a name="breaking-changes-in-version-10"></a>Cambios importantes en la versión 10

Hay varios cambios importantes en la versión 10 que pueden requerir cambios de código además de la recompilación de la aplicación.

> [!NOTE]
> La lista de cambios siguiente no es exhaustiva. Algunos cambios probablemente no producirán errores de compilación, pero técnicamente son importantes, ya que anulan la compatibilidad binaria con los ensamblados que dependen de las versiones anteriores de los ensamblados del SDK de .NET de Azure Cognitive Search. Los cambios importantes que se forman parte de esta categoría se enumeran también con recomendaciones. Vuelva a generar la aplicación al actualizar a la versión 10 para evitar problemas de compatibilidad binaria.

### <a name="custom-web-api-skill-definition"></a>Definición de la aptitud API web personalizada

La definición de la [aptitud API web personalizada](cognitive-search-custom-skill-web-api.md) estaba especificada de forma incorrecta en la versión 9 y anteriores. 

El modelo para la aptitud `WebApiSkill` especificaba `HttpHeaders` como una propiedad de objeto que _contenía_ un diccionario. La creación de un conjunto de aptitudes con una aptitud `WebApiSkill` construida de esta manera daría como resultado una excepción porque la API REST consideraría la solicitud mal formada. Este problema se ha corregido al hacer de `HttpHeaders`**una propiedad de diccionario de nivel superior** en el propio modelo `WebApiSkill`, que se considera una solicitud válida de la API REST.

Por ejemplo, si ha intentado crear anteriormente una instancia de `WebApiSkill` del modo siguiente:

```csharp

var webApiSkill = new WebApiSkill(
            inputs, 
            outputs,
            uri: "https://contoso.example.org")
{
    HttpHeaders = new WebApiHttpHeaders()
    {
        Headers = new Dictionary<string, string>()
        {
            ["header"] = "value"
        }
    }
};

```

cámbielo por lo siguiente para evitar el error de validación de la API REST:

```csharp

var webApiSkill = new WebApiSkill(
            inputs, 
            outputs,
            uri: "https://contoso.example.org")
{
    HttpHeaders = new Dictionary<string, string>()
    {
        ["header"] = "value"
    }
};

```

## <a name="shaper-skill-allows-nested-context-consolidation"></a>La aptitud Conformador permite la consolidación de contextos anidados

La aptitud Conformador puede permitir ahora la consolidación de entradas de contextos anidados. Para habilitar este cambio, se ha modificado `InputFieldMappingEntry` para que se pueda crear una instancia suya mediante la especificación de una propiedad `Source` o de las dos propiedades `SourceContext` y `Inputs`.

Lo más probable es que no tenga que realizar ningún cambio en el código; no obstante, tenga en cuenta que solo se permite una de estas dos combinaciones. Esto significa lo siguiente:

- La creación de `InputFieldMappingEntry` donde solo se inicializa `Source` es válida.
- La creación de `InputFieldMappingEntry` donde solo se inicializan `SourceContext` y `Inputs` es válida.
- Todas las demás combinaciones que impliquen estas tres propiedades no son válidas.

Si decide utilizar esta nueva funcionalidad, asegúrese de que todos los clientes estén actualizados para usar primero la versión 10 antes de implementar ese cambio. De lo contrario, existe la posibilidad de que una actualización de un cliente (con una versión anterior del SDK) para la aptitud Conformador pueda producir errores de validación.

> [!NOTE]
> Aunque el modelo `InputFieldMappingEntry` subyacente se ha modificado para permitir la consolidación de contextos anidados, su uso solo es válido dentro de la definición de una aptitud Conformador. El uso de esta funcionalidad en otras aptitudes, aunque sea válido en tiempo de compilación, producirá un error de validación en tiempo de ejecución.

## <a name="skills-can-be-identified-by-a-name"></a>Identificación de las aptitudes mediante un nombre

Cada aptitud dentro de un conjunto de aptitudes tiene ahora una nueva propiedad `Name`, que se puede inicializar en su código para ayudar a identificar la aptitud. Es opcional: cuando no se especifica (que es el valor predeterminado si no se ha realizado ningún cambio de código explícito), se le asigna un nombre predeterminado mediante el índice basado en 1 de la aptitud en el conjunto de aptitudes, precedido del carácter "#". Por ejemplo, en la siguiente definición del conjunto de aptitudes (la mayoría de las inicializaciones se omiten por motivos de brevedad):

```csharp
var skillset = new Skillset()
{
    Skills = new List<Skill>()
    {
        new SentimentSkill(),
        new WebApiSkill(),
        new ShaperSkill(),
        ...
    }
}
```

A `SentimentSkill` se le asigna un nombre `#1`, a `WebApiSkill` se le asigna `#2`, a `ShaperSkill` se le asigna `#3`, y así sucesivamente.

Si decide identificar las aptitudes mediante un nombre personalizado, asegúrese de actualizar primero todas las instancias de los clientes a la versión 10 del SDK. Si no, existe la posibilidad de que un cliente que use una versión anterior del SDK pueda anular mediante `null` la propiedad `Name` de una aptitud, lo que hará que el cliente se revierta al esquema de nomenclatura predeterminado.

## <a name="details-about-errors-and-warnings"></a>Detalles acerca de errores y advertencias

Los modelos `ItemError` y `ItemWarning` que encapsulan los detalles de los errores y advertencias (respectivamente) que se producen durante la ejecución de un indexador se han modificado para incluir tres nuevas propiedades con el objetivo de ayudar a la depuración del indexador. Estas propiedades son:

- `Name`: nombre del origen en el que se originó el error. Por ejemplo, podría hacer referencia a una aptitud determinada en el conjunto de aptitudes adjunto.
- `Details`: información adicional detallada sobre el error o la advertencia.
- `DocumentationLink`: vínculo a una guía de solución de problemas para el error o la advertencia específico.

> [!NOTE]
> Se ha comenzado a estructurar los errores y las advertencias para incluir estos detalles útiles siempre que sea posible. Se está trabajando para garantizar que estos detalles estén presentes para todos los errores y las advertencias, pero es un trabajo que está aún en curso y es posible que no siempre aparezcan estos detalles adicionales.

## <a name="next-steps"></a>Pasos siguientes

- Los cambios en la aptitud Conformador tienen el mayor impacto posible en el código nuevo o existente. Como paso siguiente, asegúrese de volver a ver este ejemplo en el que se muestra la estructura de entrada: [Ejemplo de definición JSON de la aptitud Conformador](cognitive-search-skill-shaper.md)
- Consulte [Introducción al enriquecimiento de inteligencia artificial](cognitive-search-concept-intro.md).
- Agradecemos sus comentarios sobre el SDK. Si tiene algún problema, no dude en pedirnos ayuda en [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-search). Si encuentra un error, puede enviarlo al [repositorio de GitHub del SDK de .NET para Azure](https://github.com/Azure/azure-sdk-for-net/issues). Asegúrese de agregar "[Azure Cognitive Search]" como prefijo en el título del problema.