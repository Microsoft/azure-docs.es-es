---
title: Lectura de la fuente de cambios de Azure Cosmos DB
description: En este artículo se describen las diferentes opciones disponibles para leer y obtener acceso a la fuente de cambios en Azure Cosmos DB.
author: timsander1
ms.author: tisande
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 05/20/2020
ms.reviewer: sngun
ms.openlocfilehash: 23f99dc5c648948ce07f1b40106667d24906328a
ms.sourcegitcommit: 3bf69c5a5be48c2c7a979373895b4fae3f746757
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88236801"
---
# <a name="reading-azure-cosmos-db-change-feed"></a>Lectura de la fuente de cambios de Azure Cosmos DB

Puede trabajar con la fuente de cambios de Azure Cosmos DB mediante un modelo de inserción o un modelo de extracción. Con un modelo de inserción, un servidor (la fuente de cambios) envía el trabajo a un cliente que tiene la lógica de negocios para procesar este trabajo. Sin embargo, el servidor se encarga de la complejidad de comprobar el trabajo y almacenar el estado del último trabajo procesado.

Con un modelo de extracción, el cliente tiene que extraer el trabajo del servidor. En este caso, el cliente no solo tiene lógica de negocios para procesar el trabajo, sino que también almacena el estado del último trabajo procesado. De este modo, se encarga del equilibrio de carga entre varios clientes que procesan el trabajo en paralelo, así como de los errores.

Para leer la fuente de cambios de Azure Cosmos DB, normalmente se recomienda usar un modelo de inserción, ya que no es necesario preocuparse por lo siguiente:

- Sondeo de la fuente de cambios para los cambios futuros.
- Almacenamiento del estado del último cambio procesado. Al leer la fuente de cambios, este se almacena automáticamente en un [contenedor de concesión](change-feed-processor.md#components-of-the-change-feed-processor).
- Equilibrio de la carga entre varios clientes que consumen cambios. Por ejemplo, si un cliente no puede hacer frente al procesamiento de los cambios y otro tiene capacidad disponible.
- [Control de errores](change-feed-processor.md#error-handling). Por ejemplo, los cambios que no se procesaron correctamente se vuelven a intentar automáticamente después de una excepción no controlada en el código o un problema de red transitorio.

La mayoría de los escenarios que usan la fuente de cambios de Azure Cosmos DB usarán una de las opciones del modelo de inserción. Sin embargo, hay algunas situaciones en las que es preferible el control de bajo nivel adicional del modelo de extracción. Entre ellas se incluyen las siguientes:

- Leer cambios de una clave de partición determinada.
- Controlar el ritmo con el que el cliente recibe los cambios para procesarlos.
- Realizar una lectura única de los datos existentes en la fuente de cambios (por ejemplo, para realizar una migración de datos).

## <a name="reading-change-feed-with-a-push-model"></a>Lectura de la fuente de cambios con un modelo de inserción

Usar un modelo de inserción es la forma más fácil de leer la fuente de cambios. Hay dos maneras de leer la fuente de cambios con un modelo de inserción: Los [desencadenadores de Azure Functions para Cosmos DB](change-feed-functions.md) y la [biblioteca de procesadores de fuente de cambios](change-feed-processor.md). Azure Functions usa el procesador de fuente de cambios en segundo plano, por lo que ambas son formas muy similares de leer la fuente de cambios. Simplemente, piense en Azure Functions como una plataforma de hospedaje para el procesador de fuente de cambios, no como una forma totalmente diferente de leer la fuente de cambios.

### <a name="azure-functions"></a>Azure Functions

Azure Functions es la opción más sencilla si está empezando a usar la fuente de cambios. Debido a su simplicidad, también es la opción recomendada para la mayoría de los casos de uso de la fuente de cambios. Cuando cree un desencadenador de Azure Functions para Azure Cosmos DB, seleccione el contenedor para conectarse, y la función se desencadenará siempre que se realice un cambio en el contenedor. Dado que Azure Functions usa el procesador de fuente de cambios en segundo plano, ejecuta en paralelo de forma automática el procesamiento de los cambios en las [particiones](partition-data.md) del contenedor.

El desarrollo con Azure Functions es una experiencia sencilla y puede resultar más rápida que implementar el procesador de fuente de cambios por su cuenta. Los desencadenadores pueden crearse con el portal de Azure Functions o mediante programación con los SDK. Visual Studio y VS Code proporcionan compatibilidad para escribir funciones de Azure Functions e incluso puede usar la CLI de Azure Functions para el desarrollo multiplataforma. Puede escribir y depurar el código en el escritorio y, luego, implementar la función con un solo clic. Consulte los artículos [Informática de base de datos sin servidor con Azure Functions](serverless-computing-database.md) y [Uso de la fuente de cambios con Azure Functions](change-feed-functions.md) para obtener más información.

### <a name="change-feed-processor-library"></a>Biblioteca de procesadores de fuente de cambios

El procesador de fuente de cambios proporciona un mayor control sobre la fuente de cambios y oculta también la mayor parte de la complejidad. La biblioteca de procesadores de fuente de cambios sigue el patrón de observador, en el que la biblioteca llama a la función de procesamiento. La biblioteca comprueba automáticamente si hay cambios y, si se encuentran cambios, los envía al cliente. Si tiene una fuente de cambios de alto rendimiento, puede crear instancias de varios clientes para leer la fuente de cambios. La biblioteca de procesadores de fuente de cambios dividirá automáticamente la carga entre distintos clientes. No tendrá que implementar ninguna lógica para el equilibrio de cargas entre varios clientes, ni ninguna lógica para mantener el estado de la concesión.

La biblioteca de procesadores de fuente de cambios garantiza una entrega de "al menos una vez" de todos los cambios. Es decir, si usa la biblioteca de procesadores de fuente de cambios, la función de procesamiento se llamará correctamente por cada elemento de la fuente de cambios. Si hay una excepción no controlada en la lógica de negocios de la función de procesamiento, los cambios con errores se volverán a intentar hasta que se procesen correctamente. Para evitar que el procesador de fuente de cambios se "atasque" al reintentar continuamente los mismos cambios, agregue lógica en la función de procesamiento para escribir documentos, en caso de excepción, en una cola de mensajes fallidos. Más información sobre el [control de errores](change-feed-processor.md#error-handling).

En Azure Functions, la recomendación para controlar los errores es la misma. Aún debe agregar lógica en el código de delegado para escribir documentos, en caso de excepción, en una cola de mensajes fallidos. Sin embargo, si hay una excepción no controlada en la función de Azure Functions, el cambio que ocasionó la excepción no se volverá a intentar automáticamente. Si hay una excepción no controlada en la lógica de negocios, la función de Azure Functions procesará el siguiente cambio. La función de Azure Functions no volverá a intentar el mismo cambio con errores.

Al igual que con Azure Functions, el desarrollo resulta fácil con la biblioteca de procesadores de fuente de cambios. Sin embargo, tiene la responsabilidad de implementar uno o más hosts para el procesador de fuente de cambios. Un host es una instancia de aplicación que usa el procesador de fuente de cambios para escuchar los cambios. Aunque Azure Functions tiene funcionalidades para escalado automático, el usuario debe escalar los hosts. Para obtener más información, consulte [Uso del procesador de fuente de cambios](change-feed-processor.md#dynamic-scaling). La biblioteca de procesadores de fuente de cambios es parte del [SDK V3 de Azure Cosmos DB](https://github.com/Azure/azure-cosmos-dotnet-v3).

## <a name="reading-change-feed-with-a-pull-model"></a>Lectura de la fuente de cambios con un modelo de extracción

El [modelo de extracción de fuente de cambios](change-feed-pull-model.md) permite consumir la fuente de cambios a su propio ritmo. El cliente debe solicitar los cambios y no hay ningún sondeo automático de cambios. Si quiere "marcar" de forma permanente el último cambio procesado (de manera similar al contenedor de concesión del modelo de inserción), debe [guardar un token de continuación](change-feed-pull-model.md#saving-continuation-tokens).

Con el modelo de extracción de fuente de cambios, obtiene un control de nivel más bajo sobre la fuente de cambios. Cuando lee la fuente de cambios con el modelo de extracción, tiene tres opciones:

- Leer los cambios de un contenedor completo.
- Leer los cambios de un [FeedRange](change-feed-pull-model.md#using-feedrange-for-parallelization) específico.
- Leer los cambios de un valor de clave de partición específico.

Puede ejecutar en paralelo el procesamiento de los cambios en varios clientes, del mismo modo que con el procesador de fuente de cambios. Sin embargo, el modelo de extracción no se encarga automáticamente del equilibrio de carga entre clientes. Cuando se usa el modelo de extracción para el procesamiento paralelo de la fuente de cambios, primero se obtiene una lista de FeedRanges. Un FeedRange contiene un intervalo de valores de clave de partición. Deberá tener un proceso de orquestador que obtenga los FeedRanges y los distribuya entre sus máquinas. Después, puede usar estos FeedRanges para que varias máquinas lean la fuente de cambios en paralelo.

No hay ninguna garantía de entrega "al menos una vez" incluida en el modelo de extracción. Este le ofrece un control de bajo nivel para decidir cómo quiere administrar los errores.

> [!NOTE]
> El modelo de extracción de la fuente de cambios actualmente se encuentra en [versión preliminar solo para el SDK de .NET para Azure Cosmos DB](https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.9.0-preview). La versión preliminar todavía no está disponible para otras versiones del SDK.

## <a name="change-feed-in-apis-for-cassandra-and-mongodb"></a>Fuente de cambios en las API de Cassandra y MongoDB

La funcionalidad de fuente de cambios aparece como flujos de cambios en la API de MongoDB y como consulta con predicado en Cassandra API. Para más información sobre los detalles de implementación de la API de MongoDB, consulte [Flujos de cambios en la API de Azure Cosmos DB para MongoDB](mongodb-change-streams.md).

Apache Cassandra nativo proporciona captura de datos modificados (CDC), un mecanismo para marcar tablas concretas para el archivado, así como para rechazar escrituras en esas tablas una vez que se alcanza un tamaño en disco que se puede configurar para el registro de CDC. La característica de fuente de cambios de la API de Azure Cosmos DB para Cassandra mejora la capacidad de consultar los cambios con el predicado a través de CQL. Para más información sobre los detalles de implementación, consulte [Fuente de cambios de la API de Azure Cosmos DB para Cassandra](cassandra-change-feed.md).

## <a name="next-steps"></a>Pasos siguientes

Ahora, puede seguir aprendiendo acerca de las fuentes de cambios en los siguientes artículos:

* [Introducción a la fuente de cambios](change-feed.md)
* [Using change feed with Azure Functions](change-feed-functions.md) (Usar la fuente de cambios con Azure Functions)
* [Uso de la biblioteca de procesadores de fuente de cambios](change-feed-processor.md)
