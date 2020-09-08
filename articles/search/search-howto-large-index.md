---
title: Indexación de un conjunto de datos grande mediante indizadores integrados
titleSuffix: Azure Cognitive Search
description: Estrategias de indexación de datos de gran tamaño o indexación de cálculo intensivo mediante modo por lotes, recursos y técnicas para indexación programada, paralela y distribuida.
manager: liamca
author: dereklegenzoff
ms.author: delegenz
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 05/05/2020
ms.openlocfilehash: 80307c97464e61d7b7d338703de90d1199adc819
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88927024"
---
# <a name="how-to-index-large-data-sets-in-azure-cognitive-search"></a>Indexación de grandes conjuntos de datos en Azure Cognitive Search

Azure Cognitive Search admite [dos enfoques básicos](search-what-is-data-import.md) para importar datos a un índice de búsqueda: *insertar* los datos en el índice mediante programación, o bien señalar un [indexador de Azure Cognitive Search](search-indexer-overview.md) en un origen de datos admitido para *extraer* los datos.

A medida que los volúmenes de datos aumentan o las necesidades de procesamiento cambian, es posible que las estrategias de indexación simples o predeterminadas ya no sean prácticas. Para Azure Cognitive Search, existen varios enfoques para adaptar los grandes conjuntos de datos, que van desde cómo estructurar una solicitud de carga de datos hasta usar un indizador específico del origen para las cargas de trabajo programadas y distribuidas.

Las mismas técnicas también se aplican a procesos de ejecución prolongada. En concreto, los pasos que se describen en [indización paralela](#parallel-indexing) son útiles para la indización de cálculo intensivo, como el análisis de imágenes o el procesamiento de lenguaje natural en la [canalización de enriquecimiento de inteligencia artificial](cognitive-search-concept-intro.md).

En las secciones siguientes se exploran técnicas para indexar grandes cantidades de datos mediante la API de inserción y los índices.

## <a name="push-api"></a>API de inserción

Al insertar datos en un índice, hay varias consideraciones importantes que afectan a la velocidad de indexación de la API de inserción. Estos factores se describen en la siguiente sección. 

Además de la información de este artículo, también puede aprovechar los ejemplos de código del [tutorial para optimizar las velocidades de indexación](tutorial-optimize-indexing-push-api.md) para obtener más información.

### <a name="service-tier-and-number-of-partitionsreplicas"></a>Nivel de servicio y número de particiones o réplicas

La adición de particiones o el aumento del nivel del servicio de búsqueda aumentarán las velocidades de indexación.

Agregar réplicas adicionales también puede aumentar las velocidades de indexación, pero no está garantizado. Por otro lado, las réplicas adicionales aumentarán el volumen de consultas que el servicio de búsqueda puede controlar. Las réplicas también son un componente clave para obtener un [SLA](https://azure.microsoft.com/support/legal/sla/search/v1_0/).

Antes de agregar particiones o réplicas o actualizar a un nivel superior, tenga en cuenta el costo monetario y el tiempo de asignación. La adición de particiones puede aumentar significativamente la velocidad de indexación, pero las operaciones de adición o eliminación pueden tardar entre 15 minutos y varias horas. Para obtener más información, consulte la documentación sobre cómo [ajustar la capacidad](search-capacity-planning.md).

### <a name="index-schema"></a>Esquema de índices

El esquema del índice desempeña un papel importante en la indexación de los datos. La incorporación de campos y propiedades adicionales a esos campos (por ejemplo, *que permitan buscar*, *clasificar* o *filtrar*) reducen la velocidad de indexación.

En general, recomendamos agregar propiedades adicionales a los campos solo si piensa usarlos.

> [!NOTE]
> Para mantener un tamaño de documento reducido, evite agregar datos no consultables a un índice. Las imágenes y otros datos binarios no se pueden buscar directamente y no se deben almacenar en el índice. Para integrar los datos no consultables en los resultados de búsqueda, debe definir un campo que no admita búsquedas que almacene una referencia de dirección URL al recurso.

### <a name="batch-size"></a>Tamaño de lote

Uno de los mecanismos más sencillos para la indización de un conjunto de datos más grande es enviar varios documentos o registros en una sola solicitud. Siempre y cuando la carga completa sea menor a 16 MB, una solicitud puede administrar hasta 1000 documentos en una operación de carga masiva. Estos límites se aplican tanto si usa la [API REST para agregar documentos](/rest/api/searchservice/addupdate-or-delete-documents) como el [método Index](/dotnet/api/microsoft.azure.search.documentsoperationsextensions.index?view=azure-dotnet) del SDK de .NET. Con cualquier API, se empaquetarían 1000 documentos en el cuerpo de cada solicitud.

El uso de lotes para indexar los documentos mejora significativamente el rendimiento de la indexación. Determinar el tamaño de lote óptimo para los datos es un aspecto fundamental de la optimización de las velocidades de indexación. Los dos principales aspectos que influyen en el tamaño de lote óptimo son:
+ El esquema del índice
+ El tamaño de los datos

Dado que el tamaño de lote óptimo depende del índice y los datos, la mejor estrategia es probar distintos tamaños de lote para determinar cuál ofrece mayores velocidades de indexación en su caso. En este [tutorial](tutorial-optimize-indexing-push-api.md) se proporciona código de ejemplo para probar los tamaños de lote mediante el SDK de .NET. 

### <a name="number-of-threadsworkers"></a>Número de subprocesos/trabajos

Para aprovechar al máximo las velocidades de indexación de Azure Cognitive Search, es posible que tenga que usar varios subprocesos para enviar solicitudes de indexación por lotes de forma simultánea al servicio.  

El número óptimo de subprocesos viene determinado por lo siguiente:

+ El nivel del servicio de búsqueda
+ El número de particiones
+ El tamaño de los lotes
+ El esquema del índice

Puede modificar este ejemplo y probar con diferentes números de subprocesos para determinar cuál es el número óptimo de subprocesos en su caso. Sin embargo, siempre que tenga varios subprocesos que se ejecuten simultáneamente, debería poder aprovechar al máximo la mayoría de los aumentos de eficiencia. 

> [!NOTE]
> A medida que aumente el nivel del servicio de búsqueda o el número de particiones, también debe aumentar el número de subprocesos simultáneos.

A medida que aumenta las solicitudes que alcanzan el servicio de búsqueda, puede ver [códigos de estado HTTP](/rest/api/searchservice/http-status-codes) que indican que la solicitud no se completó correctamente. Durante la indexación, dos de los códigos de estado HTTP comunes son:

+ **Servicio no disponible 503**: este error significa que el sistema está sobrecargado y la solicitud no se puede procesar en ese momento.
+ **207 Varios estados**: este error significa que algunos documentos se han procesado correctamente, pero al menos uno generó error.

### <a name="retry-strategy"></a>Estrategia de reintento 

Si se produce un error, las solicitudes deberán reintentarse con una [estrategia de reintento de retroceso exponencial](/dotnet/architecture/microservices/implement-resilient-applications/implement-retries-exponential-backoff).

El SDK de Azure Cognitive Search para .NET reintenta automáticamente las solicitudes 503 y otras con error, pero es preciso que implemente su propia lógica para reintentar las solicitudes 207. También se pueden usar herramientas de código abierto como [Polly](https://github.com/App-vNext/Polly) para implementar una estrategia de reintento.

### <a name="network-data-transfer-speeds"></a>Velocidades de transferencia de datos entre redes

Las velocidades de transferencia de datos de red pueden ser un factor de limitación durante la indexación de datos. La indexación de datos desde su entorno de Azure es una manera fácil de acelerar la indexación.

## <a name="indexers"></a>Indexadores

Los [indexadores](search-indexer-overview.md) sirven para rastrear en los orígenes de datos de Azure admitidos contenido que permite búsquedas. Aunque no se diseñaron específicamente para el indexado a gran escala, varias funcionalidades de los indizadores son especialmente útiles para adaptar grandes conjuntos de datos:

+ Los programadores le permiten empaquetar la indexación en intervalos regulares para que se pueda distribuir con el tiempo.
+ Puede reanudar la indexación programada en el último punto de detención conocido. Si un origen de datos no se rastrea completamente en un plazo de 24 horas, el indizador reanudará la indexación en el segundo día donde se quedó.
+ Particionar los datos en orígenes de datos individuales más pequeños permite realizar procesamientos en paralelo. Puede dividir los datos de origen en componentes más pequeños, por ejemplo, en varios contenedores en Azure Blob Storage y, luego, crear [varios orígenes de datos](/rest/api/searchservice/create-data-source) correspondientes en Azure Cognitive Search que se puedan indexar en paralelo.

> [!NOTE]
> Los indizadores son específicos del origen de datos, por lo que un enfoque con indizadores solo es posible para algunos orígenes de datos en Azure: [SQL Database](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md), [Blob Storage](search-howto-indexing-azure-blob-storage.md), [Table Storage](search-howto-indexing-azure-tables.md) y [Cosmos DB](search-howto-index-cosmosdb.md).

### <a name="batch-size"></a>Tamaño de lote

Al igual que con la API de inserciones, los indizadores permiten configurar el número de elementos por lote. Para los indizadores basados en la [API REST para crear indizadores](/rest/api/searchservice/Create-Indexer), puede establecer el argumento `batchSize` para personalizar esta configuración de modo que se adapte mejor a las características de los datos. 

Los tamaños de lote predeterminados son específicos para cada origen de datos. Azure SQL Database y Azure Cosmos DB tienen un tamaño de lote predeterminado de 1000. Por el contrario, la indexación de Azure Blob establece el tamaño de lote en 10 documentos en atención al tamaño máximo medio de los documentos. 

### <a name="scheduled-indexing"></a>Indexación programada

La programación de indizadores es un mecanismo importante para procesar conjuntos de datos de gran tamaño y realizar procesos de ejecución lenta, como el análisis de imágenes en una canalización de Cognitive Search. El procesamiento de los indexadores funciona dentro de una ventana de 24 horas. Si el procesamiento no se completa en menos de 24 horas, los comportamientos de la programación de los indexadores pueden resultar beneficiosos. 

De manera predeterminada, la indexación programada se inicia en intervalos específicos, habitualmente con un trabajo que se completa antes de reanudar en el intervalo programado siguiente. Sin embargo, si el procesamiento no se completa dentro del intervalo, el indexador se detiene (porque se agotó el tiempo). En el intervalo siguiente, el procesamiento se reanuda donde quedó, porque el sistema lleva un registro de dónde sucedió eso. 

En la práctica, para cargas de índice que abarcan varios días, puede poner el indexador en una programación de 24 horas. Cuando la indexación se reanuda para el próximo ciclo de 24 horas, se reinicia en el último documento correcto conocido. De este modo, un indexador puede abrirse camino a través del trabajo pendiente de un documento en una serie de días hasta procesar todos los documentos no procesados. Para obtener más información sobre este enfoque, consulte la sección sobre la [indexación de conjuntos de datos de gran tamaño en Azure Blob Storage](search-howto-indexing-azure-blob-storage.md#indexing-large-datasets). Para más información sobre cómo configurar las programaciones en general, consulte la [API REST para crear indizadores ](/rest/api/searchservice/Create-Indexer) o [Programación de indizadores para Azure Cognitive Search](search-howto-schedule-indexers.md).

<a name="parallel-indexing"></a>

### <a name="parallel-indexing"></a>Indexación en paralelo

La estrategia de indexación en paralelo se basa en la indexación de varios orígenes de datos al unísono, donde cada definición de origen de datos especifica un subconjunto de los datos. 

Para los requisitos de indexación que no son de rutina ni de cálculo intensivo (tales como el OCR en documentos digitalizados en una canalización de Cognitive Search, análisis de imagen o procesamiento de lenguaje natural), la estrategia de indexación en paralelo a menudo es el enfoque correcto para completar un proceso de larga ejecución en el tiempo más corto. Si puede eliminar o reducir las solicitudes de consulta, la indexación en paralelo en un servicio que no administra soluciones de consulta de manera simultánea es la mejor estrategia para trabajar en un gran volumen de contenido de procesamiento lento. 

El procesamiento paralelo tiene los siguientes elementos:

+ Subdivida los datos de origen entre varios contenedores o varias carpetas virtuales dentro del mismo contenedor. 
+ Asigne cada pequeño conjunto de datos a su propio [origen de datos](/rest/api/searchservice/create-data-source), emparejado con su propio [indexador](/rest/api/searchservice/create-indexer).
+ En el caso de Cognitive Search, haga referencia al mismo [conjunto de datos](/rest/api/searchservice/create-skillset) en cada definición de indexador.
+ Escriba en el mismo índice de búsqueda de destino. 
+ Programe todos los indexadores para que se ejecuten al mismo tiempo.

> [!NOTE]
> En Azure Cognitive Search, no se pueden asignar réplicas o particiones individuales al procesamiento de indexación o consulta. El sistema determina cómo se usan los recursos. Para comprender el impacto en el rendimiento de las consultas, puede probar la indexación paralela en un entorno de prueba antes de implementarlo en producción.  

### <a name="how-to-configure-parallel-indexing"></a>Cómo configurar la indexación en paralelo

En el caso de los indexadores, la capacidad de procesamiento se basa parcialmente en un subsistema de indexador para cada unidad de servicio que el servicio de búsqueda usa. Es posible tener varios indizadores simultáneos en el servicio de Azure Cognitive Search aprovisionado en los niveles Básico o Estándar con dos réplicas al menos. 

1. En [Azure Portal](https://portal.azure.com), en la página **Información general** del panel del servicio de búsqueda, compruebe el **plan de tarifa** para comprobar que puede permitir la indexación en paralelo. Los niveles Básico y Estándar ofrecen varias réplicas.

2. Puede ejecutar tantos indexadores en paralelo como el número de unidades de búsqueda del servicio. En **Configuración** > **Escala**, [aumente las réplicas](search-capacity-planning.md) o las particiones para el procesamiento paralelo: una réplica o partición adicional para cada carga de trabajo de indexador. Deje un número suficiente para el volumen de consultas existente. No se recomienda sacrificar las cargas de trabajo por la indexación.

3. Distribuya los datos en varios contenedores en un nivel que puedan alcanzar los indizadores de Azure Cognitive Search. Esto podría tratarse de varias tablas en Azure SQL Database, varios contenedores en Azure Blob Storage o varias colecciones. Defina un objeto de origen de datos para cada tabla o contenedor.

4. Cree y programe varios indexadores para que se ejecuten en paralelo:

   + Considere un servicio con seis réplicas. Configure seis indexadores, cada uno asignado a un origen de datos con un sexto del conjunto de datos para dividir en seis todo el conjunto de datos. 

   + Dirija cada indexador al mismo índice. En el caso de las cargas de trabajo de Cognitive Search, dirija cada indexador al mismo conjunto de aptitudes.

   + Dentro de cada definición de indexador, programe el mismo patrón de ejecución del tiempo de ejecución. Por ejemplo, `"schedule" : { "interval" : "PT8H", "startTime" : "2018-05-15T00:00:00Z" }` crea una programación para el 15-05-2018 en todos los indexadores, que se ejecutará a intervalos de ocho horas.

A la hora programada, todos los indexadores empezarán a ejecutarse, cargarán los datos, aplicarán los enriquecimientos (si configuró una canalización de Cognitive Search) y escribirán en el índice. Azure Cognitive Search no bloquea las actualizaciones del índice. Las escrituras simultáneas se administran y se realiza un reintento si una escritura determinada no se realizó correctamente la primera vez.

> [!Note]
> Cuando aumente las réplicas, considere aumentar el número de particiones si se espera que el tamaño del índice aumente de manera considerable. Las particiones almacenan segmentos del contenido indexado: cuantas más particiones tenga, más pequeño será el segmento que cada una debe almacenar.

## <a name="see-also"></a>Consulte también

+ [Información general del indexador](search-indexer-overview.md)
+ [Indexing in the portal](search-import-data-portal.md) (Indexación en el portal)
+ [Azure SQL Database indexer](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md) (Indexador de Azure SQL Database)
+ [Indexador de Azure Cosmos DB](search-howto-index-cosmosdb.md)
+ [Indexador de Azure Blob Storage](search-howto-indexing-azure-blob-storage.md)
+ [Indexador de Azure Table Storage](search-howto-indexing-azure-tables.md)
+ [Seguridad en Azure Cognitive Search](search-security-overview.md)