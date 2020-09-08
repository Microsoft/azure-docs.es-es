---
title: Análisis y validación de modelos
titleSuffix: Azure Digital Twins
description: Obtenga información sobre cómo usar la biblioteca del analizador para analizar los modelos de DTDL.
author: baanders
ms.author: baanders
ms.date: 4/10/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 69b52be3a3eca2ab48ed09f6401780ea033f223c
ms.sourcegitcommit: 5b6acff3d1d0603904929cc529ecbcfcde90d88b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88723986"
---
# <a name="parse-and-validate-models-with-the-dtdl-parser-library"></a>Análisis y validación de modelos con la biblioteca del analizador de DTDL

Los [modelos](concepts-models.md) de Azure Digital Twins se definen con el lenguaje de definición de gemelos digitales (DTDL) basado en JSON-LD. **Se recomienda validar los modelos sin conexión antes de cargarlos en la instancia de Azure Digital Twins**.

Para facilitar esta tarea, se proporciona una biblioteca de análisis de DTDL del lado cliente de .NET en NuGet: [**Microsoft.Azure.DigitalTwins.Parser**](https://nuget.org/packages/Microsoft.Azure.DigitalTwins.Parser/). 

Puede usar la biblioteca del analizador directamente en el código de C#, o bien usar el proyecto de ejemplo de código independiente del lenguaje que se basa en la biblioteca del analizador: [**Ejemplo de validador de DTDL**](https://docs.microsoft.com/samples/azure-samples/dtdl-validator/dtdl-validator).

## <a name="use-the-dtdl-validator-sample"></a>Uso del ejemplo de validador de DTDL

El [**validador de DTDL**](https://docs.microsoft.com/samples/azure-samples/dtdl-validator/dtdl-validator) es un proyecto de ejemplo que puede validar los documentos del modelo para garantizar que el DTDL sea válido. Se basa en la biblioteca del analizador de .NET y es independiente del lenguaje. Puede obtenerlo con el botón *Descargar archivo ZIP* en el vínculo de ejemplo.

El código fuente muestra ejemplos de cómo usar la biblioteca del analizador. Puede usar el ejemplo de validador como una utilidad de línea de comandos para validar un árbol de directorios de archivos DTDL. También proporciona un modo interactivo.

En la carpeta del ejemplo de validador de DTDL, vea el archivo *readme.md* para obtener instrucciones sobre cómo empaquetar el ejemplo en un ejecutable independiente.

Una vez que haya creado un paquete independiente y agregado el archivo ejecutable a la ruta de acceso, puede ejecutar el validador con este comando en una consola del equipo:

```cmd/sh
DTDLValidator
```

Con las opciones predeterminadas, el ejemplo buscará los archivos `*.json` en el directorio actual y en todos los subdirectorios. También puede agregar la opción siguiente para que el ejemplo busque, en el directorio indicado y en todos los subdirectorios, los archivos con la extensión *.dtdl*:

```cmd/sh
DTDLValidator -d C:\Work\DTDL -e dtdl 
```

Puede agregar la opción `-i` para que el ejemplo entre en el modo interactivo:

```cmd/sh
DTDLValidator -i
```

Para obtener más información sobre este ejemplo, vea el código fuente o ejecute `DTDLValidator --help`.

## <a name="use-the-net-parser-library"></a>Uso de la biblioteca del analizador de .NET 

La biblioteca [**Microsoft.Azure.DigitalTwins.Parser**](https://nuget.org/packages/Microsoft.Azure.DigitalTwins.Parser/) proporciona al modelo el acceso a las definiciones en DTDL, actuando esencialmente como el equivalente de reflexión en C# para DTDL. Esta biblioteca se puede usar independientemente de cualquier [SDK de Azure Digital Twins](how-to-use-apis-sdks.md), en especial para la validación de DTDL en un editor de texto o visual. Es útil para asegurarse de que los archivos de definición del modelo sean válidos antes de intentar cargarlos en el servicio.

Para usar la biblioteca del analizador, se le proporciona un conjunto de documentos DTDL. Normalmente, estos documentos de modelo se recuperan del servicio, pero también pueden estar disponibles localmente, si el cliente fue responsable de cargarlos en el servicio en primer lugar. 

Este es el flujo de trabajo general para usar el analizador:
1. Recupere algunos o todos los documentos DTDL del servicio.
2. Pase los documentos DTDL en memoria devueltos al analizador.
3. El analizador validará el conjunto de documentos que se le pasa y devolverá información detallada sobre el error. Esta capacidad es útil en escenarios de editor.
4. Use las API del analizador para seguir analizando los modelos incluidos en el conjunto de documentos. 

Entre las capacidades del analizador se incluyen:
* Obtener todas las interfaces del modelo implementadas (el contenido de la sección `extends` de la interfaz).
* Obtener todas las propiedades, la telemetría, los comandos, los componentes y las relaciones declaradas en el modelo. Este comando también obtiene todos los metadatos incluidos en estas definiciones y tiene en cuenta la herencia (secciones `extends`).
* Obtener todas las definiciones de modelos complejos.
* Determinar si un modelo es asignable desde otro modelo.

> [!NOTE]
> Los dispositivos[IoT Plug and Play (PnP)](../iot-pnp/overview-iot-plug-and-play.md) usan una pequeña variante de la sintaxis para describir su funcionalidad. Esta variante de sintaxis es un subconjunto compatible semánticamente del DTDL que se usa en Azure Digital Twins. Cuando se usa la biblioteca del analizador, no es necesario saber qué variante de sintaxis se ha usado para crear el DTDL para el gemelo digital. El analizador siempre devolverá, de forma predeterminada, el mismo modelo para la sintaxis de PnP y de Azure Digital Twins.

### <a name="code-with-the-parser-library"></a>Código con la biblioteca del analizador

Puede usar la biblioteca del analizador directamente para, por ejemplo, la validación de modelos en su propia aplicación o la generación de interfaces de usuario, paneles e informes dinámicos controlados por modelos.

Para admitir el ejemplo de código del analizador siguiente, examine varios modelos definidos en una instancia de Azure Digital Twins:

> [!TIP] 
> El modelo `dtmi:com:contoso:coffeeMaker` usa la sintaxis del *modelo de capacidad*, lo que implica que se ha instalado en el servicio mediante la conexión de un dispositivo PnP que expone ese modelo.

```json
{
  "@id": " dtmi:com:contoso:coffeeMaker",
  "@type": "CapabilityModel",
  "implements": [
        { "name": "coffeeMaker", "schema": " dtmi:com:contoso:coffeeMakerInterface" }
  ]    
}
{
  "@id": " dtmi:com:contoso:coffeeMakerInterface",
  "@type": "Interface",
  "contents": [
      { "@type": "Property", "name": "waterTemp", "schema": "double" }  
  ]
}
{
  "@id": " dtmi:com:contoso:coffeeBar",
  "@type": "Interface",
  "contents": [
        { "@type": "relationship", "contains": " dtmi:com:contoso:coffeeMaker" },
        { "@type": "property", "name": "capacity", "schema": "integer" }
  ]    
}
```

En el código siguiente se muestra un ejemplo de cómo usar la biblioteca del analizador para reflejar estas definiciones en C# :

```csharp
async void ParseDemo(DigitalTwinsClient client)
{
    try
    {
        AsyncPageable<ModelData> mdata = client.GetModelsAsync(null, true);
        List<string> models = new List<string>();
        await foreach (ModelData md in mdata)
            models.Add(md.Model);
        ModelParser parser = new ModelParser();
        IReadOnlyDictionary<Dtmi, DTEntityInfo> dtdlOM = await parser.ParseAsync(models);

        List<DTInterfaceInfo> interfaces = new List<DTInterfaceInfo>();
        IEnumerable<DTInterfaceInfo> ifenum = 
            from entity in dtdlOM.Values
            where entity.EntityKind == DTEntityKind.Interface
            select entity as DTInterfaceInfo;
        interfaces.AddRange(ifenum);
        foreach (DTInterfaceInfo dtif in interfaces)
        {
            PrintInterfaceContent(dtif, dtdlOM);
        }

    } catch (RequestFailedException rex)
    {

    }
}

void PrintInterfaceContent(DTInterfaceInfo dtif, IReadOnlyDictionary<Dtmi, DTEntityInfo> dtdlOM, int indent=0)
{
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < indent; i++) sb.Append("  ");
    Console.WriteLine($"{sb}Interface: {dtif.Id} | {dtif.DisplayName}");
    SortedDictionary<string, DTContentInfo> contents = dtif.Contents;
    foreach (DTContentInfo item in contents.Values)
    {
        switch (item.EntityKind)
        {
            case DTEntityKind.Property:
                DTPropertyInfo pi = item as DTPropertyInfo;
                Console.WriteLine($"{sb}--Property: {pi.Name} with schema {pi.Schema}");
                break;
            case DTEntityKind.Relationship:
                DTRelationshipInfo ri = item as DTRelationshipInfo;
                Console.WriteLine($"{sb}--Relationship: {ri.Name} with target {ri.Target}");
                break;
            case DTEntityKind.Telemetry:
                DTTelemetryInfo ti = item as DTTelemetryInfo;
                Console.WriteLine($"{sb}--Telemetry: {ti.Name} with schema {ti.Schema}");
                break;
            case DTEntityKind.Component:
                DTComponentInfo ci = item as DTComponentInfo;
                Console.WriteLine($"{sb}--Component: {ci.Id} | {ci.Name}");
                dtdlOM.TryGetValue(ci.Id, out DTEntityInfo value);
                DTInterfaceInfo component = value as DTInterfaceInfo;
                PrintInterfaceContent(component, dtdlOM, indent + 1);
                break;
            default:
                break;
        }
    }
}
```

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya terminado de escribir los modelos, consulte cómo cargarlos (y realizar otras operaciones de administración) con las API de DigitalTwinsModels:
* [*Procedimiento: Administración de modelos personalizados*](how-to-manage-model.md)