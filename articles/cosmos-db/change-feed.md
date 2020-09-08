---
title: Compatibilidad con la fuente de cambios en Azure Cosmos DB
description: Use la compatibilidad con la fuente de cambios de Azure Cosmos DB para controlar los cambios en documentos y realizar el procesamiento basado en eventos tales como desencadenadores y mantener actualizados las cachés y los sistemas de análisis.
author: TheovanKraay
ms.author: thvankra
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 04/08/2020
ms.reviewer: sngun
ms.custom: seodec18
ms.openlocfilehash: 4cd0ad1553f04a781349a4664fbb408108015632
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88510290"
---
# <a name="change-feed-in-azure-cosmos-db"></a>Fuente de cambios en Azure Cosmos DB

Para proporcionar compatibilidad con la fuente de cambios en Azure Cosmos DB, se observa si hay algún cambio en un contenedor de Azure Cosmos. A continuación, muestra la lista ordenada de los documentos que han cambiado en el orden en el que se modificaron. Los cambios se conservan y se pueden procesar de manera asincrónica. El resultado puede distribuirse entre uno o varios clientes para procesarse en paralelo.

Más información sobre los [modelos de diseño de la fuente de cambios](change-feed-design-patterns.md).

## <a name="supported-apis-and-client-sdks"></a>SDK de cliente y API compatibles

La característica actualmente es compatible con los siguientes SDK de cliente y API de Cosmos DB.

| **Controladores cliente** | **SQL API** | **API de Azure Cosmos DB para Cassandra** | **API de Azure Cosmos DB para MongoDB** | **Gremlin API**|**Table API** |
| --- | --- | --- | --- | --- | --- | --- |
| .NET | Sí | Sí | Sí | Sí | No |
|Java|Sí|Sí|Sí|Sí|No|
|Python|Sí|Sí|Sí|Sí|No|
|Node/JS|Sí|Sí|Sí|Sí|No|

## <a name="change-feed-and-different-operations"></a>Fuente de cambios y operaciones diferentes

Actualmente, verá todas las inserciones y actualizaciones en la fuente de cambios. No se puede filtrar la fuente de cambios para un tipo de operación específico. Una posible alternativa consiste en agregar un "marcador temporal" en el elemento para las actualizaciones y filtrar los elementos basándose en dicho marcador durante el procesamiento en la fuente de cambios.

Actualmente, la fuente de cambios no registra las eliminaciones. De forma similar al ejemplo anterior, puede agregar un marcador flexible en los elementos que se van a eliminar. Por ejemplo, puede agregar un atributo en el elemento denominado "deleted" y establecerlo en "true" y establecer un TTL en el elemento, de modo que pueda eliminarlo automáticamente. Puede leer la fuente de cambios para los elementos históricos (el cambio más reciente que corresponde al elemento no incluye los cambios intermedios), por ejemplo, los que se agregaron hace cinco años. Puede leer la fuente de cambios desde el origen del contenedor, pero si se elimina un elemento, se quitará de la fuente de cambios.

### <a name="sort-order-of-items-in-change-feed"></a>Criterio de ordenación de los elementos en la fuente de cambios

Los elementos de la fuente de cambios están ordenados según su hora de modificación. Este criterio de ordenación solo se garantiza por clave de partición lógica.

### <a name="consistency-level"></a>Nivel de coherencia

Al consumir la fuente de cambios en un nivel de posible coherencia, podría haber eventos duplicados entre las operaciones de lectura de la fuente de cambios posteriores (el último evento de una operación de lectura aparece como el primero de la siguiente).

### <a name="change-feed-in-multi-region-azure-cosmos-accounts"></a>Fuente de cambios en las cuentas de Azure Cosmos de varias regiones

En una cuenta de Azure Cosmos de varias regiones, si se produce un error en una región de escritura, la fuente de cambios funcionará durante la operación de conmutación por error manual y será contigua.

### <a name="change-feed-and-time-to-live-ttl"></a>Fuente de cambios y el período de vida (TTL)

Si la propiedad TTL (período de vida) se establece en -1 en un elemento, la fuente de cambios se conservará para siempre. Si los datos no se eliminan, se mantendrán en la fuente de cambios.  

### <a name="change-feed-and-_etag-_lsn-or-_ts"></a>Fuente de cambios y _etag, _lsn o _ts

El formato _etag es interno y no debe depender de él, porque puede cambiar en cualquier momento. _ts es una modificación o una marca de tiempo de creación. Puede usar _ts para la comparación cronológica. _lsn es un identificador de lote que se agrega solo para la fuente de cambios y representa el identificador de transacción. Muchos elementos pueden tener el mismo _lsn. ETag en FeedResponse es diferente del formato _etag que se ve en un elemento. _etag es un identificador interno que se usa para el control de simultaneidad. La propiedad _etag informa de la versión del elemento, mientras que la propiedad ETag se usa para secuenciar la fuente.

## <a name="working-with-change-feed"></a>Trabajo con la fuente de cambios

Puede trabajar con la fuente de cambios mediante las siguientes opciones:

* [Using change feed with Azure Functions](change-feed-functions.md) (Usar la fuente de cambios con Azure Functions)
* [Usar la fuente de cambios con el procesador de fuente de cambios](change-feed-processor.md) 

La fuente de cambios está disponible para cada clave de partición lógica dentro del contenedor y, por tanto, se puede distribuir entre uno o varios consumidores para el procesamiento en paralelo, tal y como se muestra en la imagen a continuación.

:::image type="content" source="./media/change-feed/changefeedvisual.png" alt-text="Procesamiento distribuido de la fuente de cambios de Azure Cosmos DB" border="false":::

## <a name="features-of-change-feed"></a>Características de la fuente de cambios

* La fuente de cambios está habilitada de forma predeterminada para todas las cuentas de Azure Cosmos.

* Puede usar el [procesamiento aprovisionado](request-units.md) para leer la fuente de cambios, igual que con cualquier otra operación de Azure Cosmos DB, en cualquiera de las regiones asociadas con la base de datos de Azure Cosmos.

* La fuente de cambios incluye inserciones y operaciones de actualización realizadas en los elementos dentro del contenedor. Puede capturar las eliminaciones si establece un marcador de "eliminación temporal" dentro de los elementos (tales como documentos) en lugar de eliminaciones. También pude establecer un período finito de expiración para los elementos con la [funcionalidad de período de vida](time-to-live.md). Por ejemplo, puede elegir 24 horas y usar el valor de esa propiedad para capturar las eliminaciones. Con esta solución, tiene que procesar los cambios dentro de un intervalo de tiempo más corto que el período de expiración de TTL.

* Cada cambio realizado en un elemento aparece exactamente una vez en la fuente de cambios y los clientes deben administrar la lógica de puntos de comprobación. Si quiere evitarse la complejidad de administrar los puntos de comprobación, el procesador de fuentes de cambio proporciona puntos de comprobación automáticos y semántica de "al menos una vez". Consulte [Usar la fuente de cambios con el procesador de fuente de cambios](change-feed-processor.md).

* Solo el cambio más reciente en un elemento determinado se incluye en el registro de cambios. Puede que los cambios intermedios no estén disponibles.

* La fuente de cambios está organizada según el orden de modificación dentro de cada valor de clave de partición lógica. No hay ningún orden garantizado entre valores de clave de partición.

* Los cambios se pueden sincronizar desde cualquier punto en el tiempo, es decir, no hay ningún período fijo de retención de datos durante el que los cambios estén disponibles.

* Los cambios están disponibles en paralelo para todas las claves de partición lógica de un contenedor de Azure Cosmos. Esta funcionalidad permite que los cambios de contenedores grandes se procesen en paralelo por medio de varios consumidores.

* Las aplicaciones pueden solicitar varias fuentes de cambios en el mismo contenedor de forma simultánea. ChangeFeedOptions.StartTime se puede usar para proporcionar un punto inicial. Por ejemplo, para buscar el token de continuación correspondiente a una hora especificada. ContinuationToken, si se especifica, tiene prioridad sobre los valores StartTime y StartFromBeginning. La precisión de ChangeFeedOptions.StartTime es aproximadamente de 5 segundos.

## <a name="change-feed-in-apis-for-cassandra-and-mongodb"></a>Fuente de cambios en las API de Cassandra y MongoDB

La funcionalidad de fuente de cambios aparece como flujo de cambios en la API de MongoDB y como consulta con predicado en Cassandra API. Para más información sobre los detalles de implementación de la API de MongoDB, consulte [Flujos de cambios en la API de Azure Cosmos DB para MongoDB](mongodb-change-streams.md).

Apache Cassandra nativo proporciona captura de datos modificados (CDC), un mecanismo para marcar tablas concretas para el archivado, así como para rechazar escrituras en esas tablas una vez que se alcanza un tamaño en disco que se puede configurar para el registro de CDC. La característica de fuente de cambios de la API de Azure Cosmos DB para Cassandra mejora la capacidad de consultar los cambios con el predicado a través de CQL. Para más información sobre los detalles de implementación, consulte [Fuente de cambios de la API de Azure Cosmos DB para Cassandra](cassandra-change-feed.md).

## <a name="next-steps"></a>Pasos siguientes

Ahora, puede obtener más información acerca de las fuentes de cambios en los siguientes artículos:

* [Options to read change feed](read-change-feed.md) (Opciones para leer la fuente de cambios)
* [Using change feed with Azure Functions](change-feed-functions.md) (Usar la fuente de cambios con Azure Functions)
* [Usar el procesador de fuente de cambios](change-feed-processor.md)
