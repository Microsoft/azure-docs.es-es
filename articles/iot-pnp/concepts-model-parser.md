---
title: Uso del analizador de modelos de Digital Twins | Microsoft Docs
description: La finalidad de este artículo es que los desarrolladores aprendan a usar el analizador de DTDL para validar los modelos.
author: rido-min
ms.author: rmpablos
ms.date: 04/29/2020
ms.topic: conceptual
ms.custom: mvc
ms.service: iot-pnp
services: iot-pnp
manager: peterpr
ms.openlocfilehash: 20c4452a32c791f33e08c883d8cec89a345ab188
ms.sourcegitcommit: 46f8457ccb224eb000799ec81ed5b3ea93a6f06f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87352050"
---
# <a name="understand-the-digital-twins-model-parser"></a>Uso del analizador de modelos de Digital Twins

El lenguaje de definición de Digital Twins (DTDL) se describe en la [especificación de DTDL](https://github.com/Azure/opendigitaltwins-dtdl). Los usuarios pueden usar el paquete de NuGet _Digital Twins Model Parser_ para validar y consultar los modelos definidos en varios archivos.

## <a name="install-the-dtdl-model-parser"></a>Instalación del analizador de modelos de DTDL

El analizador está disponible en NuGet.org con el identificador: [Microsoft.Azure.DigitalTwins.Parser](https://www.nuget.org/packages/Microsoft.Azure.DigitalTwins.Parser). Para instalar el analizador, use cualquier administrador de paquetes de NuGet compatible, como el de Visual Studio o la CLI de `dotnet`.

```bash
dotnet add package Microsoft.Azure.DigitalTwins.Parser
```

## <a name="use-the-parser-to-validate-a-model"></a>Uso del analizador para validar un modelo

El modelo que quiere validar puede estar formado por una o varias interfaces descritas en archivos JSON. Puede usar el analizador para cargar todos los archivos de una carpeta determinada y validar todos los archivos en su totalidad, incluidas las referencias entre ellos:

1. Cree un elemento `IEnumerable<string>` con una lista de todo el contenido del modelo:

    ```csharp
    using System.IO;

    string folder = @"c:\myModels\";
    string filespec = "*.json";

    List<string> modelJson = new List<string>();
    foreach (string filename in Directory.GetFiles(folder, filespec))
    {
        using StreamReader modelReader = new StreamReader(filename);
        modelJson.Add(modelReader.ReadToEnd());
    }
    ```

1. Cree una instancia de `ModelParser` y llame a `ParseAsync`:

    ```csharp
    using Microsoft.Azure.DigitalTwins.Parser;

    ModelParser modelParser = new ModelParser();
    IReadOnlyDictionary<Dtmi, DTEntityInfo> parseResult = await modelParser.ParseAsync(modelJson);
    ```

1. Compruebe si hay errores de validación. Si el analizador encuentra algún error, inicia una excepción `AggregateException` con una lista de mensajes de error detallados:

    ```csharp
    try
    {
        IReadOnlyDictionary<Dtmi, DTEntityInfo> parseResult = await modelParser.ParseAsync(modelJson);
    }
    catch (AggregateException ae)
    {
        foreach (var e in ae.InnerExceptions)
        {
            Console.WriteLine(e.Message);
        }
    }
    ```

1. Inspeccione el elemento `Model`. Si la validación se realiza correctamente, puede usar la API del analizador de modelos para inspeccionar el modelo. En el fragmento de código siguiente se muestra cómo recorrer en iteración todos los modelos analizados y se muestran las propiedades existentes:

    ```csharp
    foreach (var m in parseResult)
    {
        Console.WriteLine(m.Key);
        foreach (var item in m.Value.AsEnumerable<DTEntityInfo>())
        {
            var p = item as DTInterfaceInfo;
            if (p!=null)
            {
                Console.WriteLine($"\t{p.Id}");
                Console.WriteLine($"\t{p.Description.FirstOrDefault()}");
            }
            Console.WriteLine("--------------");
        }
    }
    ```

## <a name="next-steps"></a>Pasos siguientes

La API del analizador de modelos revisada en este artículo permite muchos escenarios para automatizar o validar tareas que dependen de modelos de DTDL. Por ejemplo, podría crear de forma dinámica una interfaz de usuario a partir de la información del modelo.
