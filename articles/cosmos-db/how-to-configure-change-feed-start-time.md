---
title: 'Procedimiento para configurar la hora de inicio del procesador de fuente de cambios: Azure Cosmos DB'
description: Aprenda a configurar el procesador de fuente de cambios para empezar a leer desde una fecha y hora específicas.
author: ealsur
ms.service: cosmos-db
ms.topic: how-to
ms.date: 08/13/2019
ms.author: maquaran
ms.custom: devx-track-csharp
ms.openlocfilehash: dcd92421a7ef4e2314d27a7724be01336c26c9eb
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89018202"
---
# <a name="how-to-configure-the-change-feed-processor-start-time"></a>Procedimiento para configurar la hora de inicio del procesador de fuente de cambios

En este artículo se le explica cómo puede configurar el [procesador de fuente de cambios](./change-feed-processor.md) para empezar a leer desde una fecha y hora específicas.

## <a name="default-behavior"></a>Comportamiento predeterminado

Cuando se inicie un procesador de fuente de cambios por primera vez, inicializará el contenedor de concesiones e iniciará su [ciclo de vida de procesamiento](./change-feed-processor.md#processing-life-cycle). No se detectará ningún cambio que se haya producido en el contenedor antes de que se inicializara por primera vez el procesador de fuente de cambios.

## <a name="reading-from-a-previous-date-and-time"></a>Lectura desde una fecha y hora anteriores

Es posible inicializar el procesador de fuente de cambios para leer los cambios a partir de una **fecha y hora específicas**; para ello, pase una instancia de `DateTime` a la extensión del generador `WithStartTime`:

[!code-csharp[Main](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed/Program.cs?name=TimeInitialization)]

El procesador de fuente de cambios se inicializará para esa fecha y hora específicas y comenzará a leer los cambios que se produzcan después.

## <a name="reading-from-the-beginning"></a>Lectura desde el principio

En otros escenarios, como las migraciones de datos o el análisis de todo el historial de un contenedor, es necesario leer la fuente de cambios desde **el principio de la vigencia de ese contenedor**. Para ello, podemos usar `WithStartTime` en la extensión del generador, pero pasando `DateTime.MinValue.ToUniversalTime()`, de forma que se genere la representación UTC del valor `DateTime` mínimo, de la siguiente manera:

[!code-csharp[Main](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed/Program.cs?name=StartFromBeginningInitialization)]

El procesador de fuente de cambios se inicializará y comenzará a leer los cambios desde el principio de la vigencia del contenedor.

> [!NOTE]
> Estas opciones de personalización solo funcionan para configurar el momento dado inicial del procesador de fuente de cambios. Una vez que el contenedor de concesiones se inicialice por primera vez, no tendrá ningún efecto cambiar las opciones.

> [!NOTE]
> Al especificar un momento dado, solo se leerán los cambios de los elementos que existan actualmente en el contenedor. Si se ha eliminado un elemento, también se quita su historial de la fuente de cambios.

## <a name="additional-resources"></a>Recursos adicionales

* [SDK de Azure Cosmos DB](sql-api-sdk-dotnet.md)
* [Ejemplos de uso en GitHub](https://github.com/Azure/azure-cosmos-dotnet-v3/tree/master/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed)
* [Ejemplos adicionales en GitHub](https://github.com/Azure-Samples/cosmos-dotnet-change-feed-processor)

## <a name="next-steps"></a>Pasos siguientes

Puede obtener más información sobre el procesador de la fuente de cambios en los siguientes artículos:

* [Introducción a la fuente de cambios](change-feed-processor.md)
* [Uso del calculador de la fuente de cambios](how-to-use-change-feed-estimator.md)
